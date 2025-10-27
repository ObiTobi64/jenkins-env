# ISC System Web - Jenkinsfile (PROD)

Pipeline para el entorno PRODUCCIÓN, genérico y reutilizable. **Etapas:** checkout, install, test, build, package y publish. El despliegue/publicación a PROD solo se habilita en la rama `main` y bajo aprobación manual.

## Requisitos del agente
- Label con Node.js y Docker disponibles, o configurar el Tool de Node.js:
  - Jenkins > Global Tool Configuration > NodeJS: crear `NodeJS_18` (o el nombre que prefieras).
- Docker CLI instalado para la etapa Publish.
- Git disponible.

## Credenciales
- `npm-token` (Secret Text): NPM_TOKEN para `npm ci` y acceso a registries privados.
- `docker-registry` (Username with password): credenciales de tu registry 
- (Opcional) Variable global `SLACK_CHANNEL` si usas `slackSend`.

**No almacenar secretos en el repositorio.**

## Parámetros del pipeline
- `DEPLOY_ENABLED` (bool): habilita la etapa Publish. Requiere estar en `main` y aprobación manual.
- `RUN_TESTS` (bool): habilita tests.
- `NPM_TOKEN_CREDENTIALS_ID` (string): ID de la credencial de NPM (por defecto `npm-token`).
- `DOCKER_CREDENTIALS_ID` (string): ID credencial del registry Docker (por defecto `docker-registry`).
- `REGISTRY_URL` (string): URL del registry (p. ej. `ghcr.io`).
- `IMAGE_NAME` (string): nombre de la imagen sin tag (p. ej. `org/isc-system-web`).
- `IMAGE_TAG` (string): tag explícito; si va vacío, se genera a partir de `package.json` version + commit o `GIT_SHORT_COMMIT`.
- `PACKAGE_FORMAT` (choice): `zip` (por defecto) o `tgz` (usa `npm pack`).

## Flujo
1. Checkout: limpia workspace y descarga código.
2. Prepare: configura Node.js, calcula `GIT_SHORT_COMMIT` y `IMAGE_TAG`.
3. Install: `npm ci` usando `NPM_TOKEN` si se proporciona.
4. Test: ejecuta `npm test` y publica JUnit si existen reportes.
5. Build: `npm run build` (si existe).
6. Package:
   - `zip`: empaqueta `build/` o `dist/` (si no existen, empaqueta el proyecto).
   - `tgz`: `npm pack`.
   - Publica artefacto en Jenkins (fingerprint).
7. Publish:
   - Solo `main` + `DEPLOY_ENABLED=true` + aprobación manual.
   - Login en registry, build y push de la imagen Docker: `${REGISTRY_URL}/${IMAGE_NAME}:${IMAGE_TAG}`.

   ## Cómo ejecutar una validación (pasos detallados)

   Se recomienda ejecutar una validación completa en un entorno controlado antes de habilitar despliegues a producción. A continuación hay 3 opciones.

   ### Opción A — Jenkins en Docker (UI)

   1) Levantar Jenkins LTS en Docker (PowerShell):

   ```powershell
   docker run --name jenkins-local -p 8080:8080 -p 50000:50000 -v jenkins_home:C:\var\jenkins_home -d jenkins/jenkins:lts
   ```

   2) Abre http://localhost:8080, completa el setup e instala plugins: Pipeline, Git, Credentials Binding, Docker Pipeline, JUnit, Workspace Cleanup y Slack (si lo usarás).

   3) Crear Credentials (Manage Jenkins -> Credentials -> System -> Global credentials):
      - Secret text: ID `npm-token` -> valor: tu NPM_TOKEN
      - Username with password: ID `docker-registry` -> username/password del registry

   4) Crear un job Pipeline (Pipeline script from SCM) apuntando al repo y ejecutar.

   Notas: si Jenkins corre en un contenedor y necesitas ejecutar `docker build`, habilita acceso al daemon del host (p. ej. montar `\var\run\docker.sock`) o provisiona un agente con Docker instalado.

   ### Opción B — Jenkinsfile Runner (rapidez, sin UI)

   Ejecuta el Jenkinsfile localmente con Jenkinsfile Runner (Docker). Desde la raíz del repo (donde está `Jenkinsfile`):

   ```powershell
   # Ejecutar Jenkinsfile Runner en Docker (PowerShell)
   docker run --rm -v ${PWD}.Path:/workspace -w /workspace jenkins/jenkinsfile-runner
   ```

   Limitaciones: la imagen del runner debe incluir los plugins que usa tu `Jenkinsfile` (credentials-binding, docker-workflow, junit, slack, etc.). `input` (aprobaciones) puede bloquear la ejecución; para pruebas automatizadas ejecuta con `DEPLOY_ENABLED=false`.

   ### Opción C — Validación parcial en Vercel (build & deploy)

   Vercel ejecuta el `build` definido en `package.json` y hace deploys automáticos desde Git. No ejecuta Jenkinsfiles ni `docker push`.

   Pasos rápidos para Vercel:
   1) Crear Project en Vercel apuntando al repo o a la subcarpeta.
   2) Settings -> Environment Variables -> agregar `NPM_TOKEN` si hace falta.
   3) Build Command: `npm run build` — Output Directory: `build` o `dist` según tu proyecto.
   4) Hacer push a branch de validación (p. ej. `staging`) y revisar el Preview Deployment en Vercel.

   ## Checklist de verificación (aceptación)
   Antes de habilitar `DEPLOY_ENABLED` para producción, valida lo siguiente en tu entorno de prueba:

   - [ ] El job (o runner) ejecuta y muestra las etapas nombradas: `Checkout`, `Prepare`, `Install`, `Test`, `Build`, `Package`, `Publish` (si aplicable).
   - [ ] La ejecución finaliza con estado `SUCCESS`.
   - [ ] Despliegues a producción están restringidos: solo ejecutables desde `main` y requieren `DEPLOY_ENABLED=true` y aprobación manual.
   - [ ] Las notificaciones de fallo funcionan: forzar fallo en `Test` y verificar `post { failure { ... } }` (si `SLACK_CHANNEL` configurado se intentará enviar mensaje).
   - [ ] Todas las credenciales están en Jenkins Credentials y no en el repo.

   ## Sugerencias adicionales
   - Para PRs y validaciones automáticas (tests/builds), considera replicar `Install`/`Test`/`Build` en GitHub Actions y usar la integración de Vercel para deploys automáticos.
   - Si necesitas que Jenkinsfile Runner ejecute `Publish`, crea una imagen del runner con los plugins necesarios y variables/credentials configuradas en el entorno.

   ---
   Actualizado para añadir instrucciones de validación y checklist de aceptación.

## Notas de operación
- Validar primero en un entorno de prueba (ej. Vercel u otro) antes de activar `DEPLOY_ENABLED`.
- Ajustar Dockerfile y context según el proyecto.
- Si no usas Slack, no definas `SLACK_CHANNEL` y el pipeline omitirá esa notificación.
