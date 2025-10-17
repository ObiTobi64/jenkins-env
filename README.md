# Jenkinsfiles Repository

## Introduccion
Este repositorio está diseñado para almacenar archivos Jenkinsfiles para diversos proyectos y entornos. Incluye una configuración de Docker Compose para configurar fácilmente un servicio Jenkins para la integración y entrega continuas.

## Estructura
La estructura de folder es la siguiente:

```
proyecto/
├── ISC_SYSTEM/
│   ├── CORE/
│   │   ├── QA/
│   │   │   └── Jenkinsfile
│   │   └── PROD/
│   │       └── Jenkinsfile
│   └── WEB/
│       ├── QA/
│       │   └── Jenkinsfile
│       └── PROD/
│           └── Jenkinsfile
└── LEARNING_AGENT/
    ├── BACKEND/
    │   ├── QA/
    │   │   └── Jenkinsfile
    │   └── PROD/
    │       └── Jenkinsfile
    └── CLIENT/
        ├── QA/
        │   └── Jenkinsfile
        └── PROD/
            └── Jenkinsfile
```

- Cada proyecto tiene su propio directorio y, dentro de cada proyecto, hay subdirectorios para diferentes entornos (QA y PROD).
- Cada entorno contiene un archivo `Jenkinsfile` que define el proceso para ese entorno específico.

## Levantar Jenkins

### Comando para levantar
```bash

docker-compose up -d

```
### Comando para obtener la contraseña
```bash
docker-compose logs jenkins
```
#### Ejemplo 
```bash
76de7062fe8d4574b906dc8c97635621
```

### Prerequisites

- Asegúrate de tener Docker y Docker Compose instalados en tu máquina.

### Steps to Start Jenkins

1. **Clonar el repositorio**
   ```bash
   git clone <https://github.com/ISC-UPB/jenkins-env.git>
   ```

2. **Iniciar Jenkins con Docker Compose**
   ```bash
   docker-compose up -d
   ```

3. **Acceder a Jenkins**
   - Abra su navegador web y vaya a `http://localhost:8080`.
   - Siga las instrucciones que aparecen en pantalla para desbloquear Jenkins utilizando la contraseña de administrador inicial que se encuentra en los registros o en la ubicación especificada.

4. **Verificar que Jenkins se está ejecutando**
   - Una vez que Jenkins esté en funcionamiento, puede comprobar el estado en el panel de control.
   - También puede crear un nuevo trabajo de canalización y dirigirlo a uno de los marcadores de posición `Jenkinsfile` para probar la configuración.

### Conclusion
Este repositorio proporciona un enfoque estructurado para gestionar archivos Jenkinsfiles para diferentes proyectos y entornos. Siga los pasos anteriores para configurar Jenkins y empezar a utilizarlo para sus necesidades de CI/CD.