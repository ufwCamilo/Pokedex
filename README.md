Aplicación web desarrollada en Angular que consume la PokéAPI para mostrar información detallada de pokémons. Desplegada en Azure Static Web Apps mediante un pipeline de CI/CD con GitHub Actions.

📋 Tabla de Contenidos

Descripción del Proyecto
Requisitos Previos
Instalación Local
Despliegue en Azure Static Web Apps
Problemas Encontrados y Soluciones


🎯 Descripción del Proyecto
PokéDex es una aplicación SPA (Single Page Application) construida con Angular que permite:

Explorar el listado completo de pokémons
Ver detalles de cada pokémon (estadísticas, tipos, evoluciones)
Filtrar y buscar pokémons por nombre o tipo
Visualizar sprites animados de diferentes generaciones

La aplicación está desplegada en Azure Static Web Apps con cabeceras de seguridad configuradas para proteger contra ataques XSS, clickjacking y otros vectores de amenaza.
URL de producción: https://proud-desert-02d7af910.7.azurestaticapps.net

⚙️ Requisitos Previos
Antes de instalar el proyecto asegúrate de tener:

Node.js v16 o superior
npm v8 o superior
Angular CLI v14 o superior
Git


💻 Instalación Local
1. Clona el repositorio
bashgit clone https://github.com/ufwCamilo/Pokedex.git
cd Pokedex
2. Navega al directorio del proyecto Angular
bashcd source/Pokedex/pokedex-angular
3. Instala las dependencias
bashnpm install
4. Ejecuta el servidor de desarrollo
bashng serve
La aplicación estará disponible en http://localhost:4200.
5. Compila para producción
bashng build --configuration production
Los archivos compilados se generarán en dist/pokedex-angular.

☁️ Despliegue en Azure Static Web Apps
Requisitos

Cuenta de Microsoft Azure
Repositorio en GitHub
Azure CLI o acceso al portal de Azure

Paso 1 — Crear el recurso en Azure

Ingresa al Portal de Azure
Busca "Static Web Apps" y clic en "Crear"
Completa los campos:

Suscripción: Azure subscription 1
Grupo de recursos: rg-pokedex
Nombre: pokedex-pueblopaleta
Plan de hospedaje: Free
Origen: GitHub → selecciona el repositorio Pokedex


En "Build Details" configura:
CampoValorBuild PresetsAngularApp locationsource/Pokedex/pokedex-angularApi location(vacío)Output locationdist/pokedex-angular

Clic en "Revisar y crear"

Paso 2 — Configurar el token de despliegue

En el recurso creado, clic en "Administrar token de implementación"
Copia el token generado
Ve a tu repositorio en GitHub → Settings → Secrets and variables → Actions
Clic en "New repository secret":

Name: AZURE_STATIC_WEB_APPS_API_TOKEN_CALM_GLACIER_0B51F6A10
Value: pega el token copiado



Paso 3 — Configurar el workflow de GitHub Actions
El archivo .github/workflows/azure-static-web-apps-calm-glacier-0b51f6a10.yml debe contener:
yamlname: Azure Static Web Apps CI/CD

on:
  push:
    branches:
      - master
  pull_request:
    types: [opened, synchronize, reopened, closed]
    branches:
      - master

jobs:
  build_and_deploy_job:
    runs-on: ubuntu-latest
    name: Build and Deploy Job
    steps:
      - uses: actions/checkout@v3
        with:
          submodules: true

      - name: Build And Deploy
        uses: Azure/static-web-apps-deploy@v1
        with:
          azure_static_web_apps_api_token: ${{ secrets.AZURE_STATIC_WEB_APPS_API_TOKEN_CALM_GLACIER_0B51F6A10 }}
          repo_token: ${{ secrets.GITHUB_TOKEN }}
          action: "upload"
          app_location: "source/Pokedex/pokedex-angular"
          api_location: ""
          output_location: "dist/pokedex-angular"
Paso 4 — Verificar el despliegue

Ve a GitHub → Actions y verifica que el workflow haya pasado ✅
Visita la URL de Azure para confirmar que la app está en línea


🐛 Problemas Encontrados y Soluciones
❌ Problema 1: Comando cp no reconocido en Windows
Error:
"cp" no se reconoce como un comando interno o externo
Causa: cp es un comando de Linux/Mac, no existe en CMD de Windows.
Solución: Usar robocopy en su lugar:
cmdrobocopy ".." "." /E /XD "..\pokedex"

❌ Problema 2: Error InvalidTemplate en Azure
Error:
json"Expected a value of type 'String, Uri', but received a value of type 'Array'"
Causa: El campo Output location en Azure Portal fue ingresado como array en vez de string.
Solución: Ingresar el valor sin corchetes: dist/pokedex-angular en lugar de ["dist/pokedex-angular"].

❌ Problema 3: Could not detect any platform in the source directory
Causa: El app_location en el workflow apuntaba a / (raíz) en lugar de donde está el package.json de Angular.
Solución: Actualizar el workflow:
yamlapp_location: "source/Pokedex/pokedex-angular"

❌ Problema 4: Repositorio Git embebido (embedded git repository)
Error:
warning: adding embedded git repository: source/Pokedex
Causa: La carpeta source/Pokedex tenía su propia carpeta .git, creando un submodule no intencionado.
Solución:
cmdrmdir /s /q source\Pokedex\.git
git rm --cached source/Pokedex
git add source/Pokedex
git commit -m "Fix: remove embedded git repository"
git push origin master --force

❌ Problema 5: deployment_token was not provided
Causa: El nombre del secret en GitHub no coincidía con el nombre referenciado en el workflow.
Solución: Crear el secret en GitHub con el nombre exacto que aparece en el workflow:
AZURE_STATIC_WEB_APPS_API_TOKEN_CALM_GLACIER_0B51F6A10

❌ Problema 6: Push rechazado (fetch first)
Error:
Updates were rejected because the remote contains work that you do not have locally
Causa: El repositorio remoto tenía commits que no estaban en local.
Solución:
cmdgit pull origin master --allow-unrelated-histories
git push origin master

📁 Estructura del Proyecto
Pokedex/
├── .github/
│   └── workflows/
│       └── azure-static-web-apps-calm-glacier-0b51f6a10.yml
├── source/
│   └── Pokedex/
│       ├── pokedex-angular/        # Proyecto Angular
│       │   ├── src/
│       │   ├── angular.json
│       │   └── package.json
│       └── staticwebapp.config.json
└── README.md

🔒 Configuración de Seguridad
El archivo staticwebapp.config.json incluye cabeceras de seguridad HTTP:
HeaderValorStrict-Transport-Securitymax-age=31536000X-Content-Type-OptionsnosniffX-Frame-OptionsDENYReferrer-Policyno-referrerPermissions-Policycamera=(), microphone=()

Desarrollado como parte del curso de Sistemas Distribuidos.
