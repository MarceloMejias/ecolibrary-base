# Configuración de VS Code para EcoLibrary

Esta carpeta contiene la configuración de VS Code para el proyecto EcoLibrary, que incluye dos instancias de Django (Backend API y Frontend Web) ejecutándose en Docker.

## Archivos de configuración

### 📋 launch.json
Configuraciones de depuración:
- **Docker: API (Backend)** - Depura el contenedor API en el puerto 5678
- **Docker: Web (Frontend)** - Depura el contenedor Web en el puerto 5679
- **Python: Core API** - Ejecuta el servidor Django API localmente
- **Python: Web Frontend** - Ejecuta el servidor Django Web localmente
- **Docker: API + Web** - Depura ambos contenedores simultáneamente

### ⚙️ tasks.json
Tareas comunes:
- **Docker: Build All** - Construye todas las imágenes
- **Docker: Up API + Web** - Levanta solo API, Web y DB (tarea por defecto)
- **Docker: Up All Services** - Levanta todos los servicios (incluye Jenkins, SonarQube)
- **Docker: Down All** - Detiene todos los contenedores
- **Docker: Logs API/Web** - Muestra logs en tiempo real
- **Docker: Restart API/Web** - Reinicia servicios específicos
- **Django: Migrate/Makemigrations** - Gestión de migraciones
- **Django: Shell API/Web** - Abre shell de Django
- **Django: Test API/Web** - Ejecuta tests

### 🔧 settings.json
Configuración del workspace:
- Rutas de Python para análisis de código
- Configuración de linting (flake8)
- Formateo automático al guardar
- Asociaciones de archivos Django
- Exclusión de archivos generados

### 📦 extensions.json
Extensiones recomendadas:
- Python + Pylance
- Docker
- Django
- YAML
- Prettier

## Uso rápido

### Iniciar desarrollo
1. Presiona `Cmd+Shift+P` (macOS) o `Ctrl+Shift+P` (Windows/Linux)
2. Escribe "Tasks: Run Task"
3. Selecciona "Docker: Up API + Web"

### Ejecutar tarea por defecto
- Presiona `Cmd+Shift+B` (macOS) o `Ctrl+Shift+B` (Windows/Linux)
- Esto ejecutará "Docker: Up API + Web"

### Depurar con Docker
1. Inicia los servicios con la tarea "Docker: Up API + Web"
2. Ve a la pestaña de Depuración (Run and Debug)
3. Selecciona "Docker: API + Web" del dropdown
4. Presiona F5 o haz clic en el botón verde de play

**Nota:** Para usar la depuración con Docker, necesitas agregar `debugpy` a tus contenedores y modificar el comando de inicio.

## Puertos

- **API (Backend):** http://localhost:8001
- **Web (Frontend):** http://localhost:8002
- **PostgreSQL:** localhost:5432
- **Jenkins:** http://localhost:8080
- **SonarQube:** http://localhost:9010

## Estructura del proyecto

```
ecolibrary/
├── core_api/          # Backend API Django REST
├── web/               # Frontend Django
├── docker-compose.yaml
└── .vscode/           # Configuración VS Code
```
