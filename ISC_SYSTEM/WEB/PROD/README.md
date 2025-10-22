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
- `NODEJS_TOOL_NAME` (string): nombre del Tool NodeJS (por defecto `NodeJS_18`).
- `NPM_TOKEN_CREDENTIALS_ID` (string): ID de la credencial de NPM.
- `DOCKER_CREDENTIALS_ID` (string): ID credencial del registry Docker.
- `REGISTRY_URL` (string): URL del registry (p. ej. `ghcr.io`).
- `IMAGE_NAME` (string): nombre de la imagen sin tag (p. ej. `org/isc-system-web`).
- `IMAGE_TAG` (string): tag explícito; si va vacío, se genera `version-commit`.
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

## Notas de operación
- Validar primero en un entorno de prueba (ej. Vercel u otro) antes de activar `DEPLOY_ENABLED`.
- Ajustar Dockerfile y context según el proyecto.
- Si no usas Slack, no definas `SLACK_CHANNEL` y el pipeline omitirá esa notificación.
