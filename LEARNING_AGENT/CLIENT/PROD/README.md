# Learning Agent Client — Jenkinsfile (PROD)

Pipeline para Learning Agent Client (PRODUCCIÓN), reusable entre proyectos. Etapas: checkout, install, test, build, package y publish. El despliegue a PROD solo se habilita en la rama `main` y con aprobación manual.

## Requisitos del agente
- JDK 17 (Tool en Jenkins: `JDK_17` o el que definas).
- Android SDK instalado (build-tools y plataformas requeridas). Exportar `ANDROID_SDK_ROOT`/`ANDROID_HOME` o usar el parámetro `ANDROID_SDK_PATH`.
- Gradle Wrapper en el repo (`gradlew` / `gradlew.bat` ejecutable).
- Git instalado.
- Plugins Jenkins:
  - Pipeline, Git, Credentials Binding
  - Pipeline Utility Steps (findFiles, archiveArtifacts)
  - Workspace Cleanup, ANSI Color, JUnit
  - (Opcional) Slack Notification

## Credenciales (Jenkins Credentials)
- Keystore (File): `android-keystore`
- Keystore password (Secret Text): `android-keystore-pass`
- Key alias (Secret Text): `android-key-alias`
- Key password (Secret Text): `android-key-pass`
- Google Play service account (File): `play-service-account` (usada por Gradle Play Publisher)
- Nunca almacenar secretos en el repo.

## Parámetros principales
- `DEPLOY_ENABLED` (bool): habilita Publish (requiere `main` y aprobación manual).
- `RUN_TESTS` (bool): ejecuta tests unitarios.
- `JAVA_TOOL_NAME` (string): Tool JDK (por defecto `JDK_17`).
- `ANDROID_SDK_PATH` (string): ruta del SDK si no está en PATH.
- `MODULE` (string): módulo a compilar (por defecto `app`).
- `BUILD_VARIANT` (choice): `Release` o `Debug`.
- `PACKAGE_FORMAT` (choice): `aab` o `apk`.
- IDs de credenciales para keystore y Play.

## Flujo
1. Checkout: limpia y descarga código.
2. Prepare: configura JDK/SDK y calcula `GIT_SHORT_COMMIT`.
3. Install: valida Gradle wrapper.
4. Test: `test{Variant}UnitTest`, publica JUnit si existen reportes.
5. Build: `bundle{Variant}`  con signing desde variables de entorno.
6. Package: detecta y archiva el artefacto (`*.aab`/`*.apk`) y `mapping.txt` si existe.
7. Publish:
   - Solo `main` + `DEPLOY_ENABLED=true` + aprobación manual.
   - Ejecuta `publish{Variant}Bundle` o `publish{Variant}Apk` (Gradle Play Publisher).

## Integración de signing y Play
En `build.gradle`/`gradle.properties`, referenciar las variables de entorno:
```gradle
// ...existing code...
android {
  signingConfigs {
    release {
      storeFile file(System.getenv("ANDROID_KEYSTORE"))
      storePassword System.getenv("ANDROID_KEYSTORE_PASSWORD")
      keyAlias System.getenv("ANDROID_KEY_ALIAS")
      keyPassword System.getenv("ANDROID_KEY_PASSWORD")
    }
  }
  buildTypes {
    release { signingConfig signingConfigs.release }
  }
}
// Play Publisher (ejemplo)
play {
  serviceAccountCredentials = file(System.getenv("PLAY_SERVICE_ACCOUNT_JSON"))
  defaultToAppBundles = true
}
// ...existing code...
```

## Validación previa
- Ejecutar el pipeline con `BUILD_VARIANT=Debug`, `DEPLOY_ENABLED=false` y verificar artefactos en Jenkins.
- Corregir observaciones antes de habilitar producción (`main` + `DEPLOY_ENABLED=true`).
