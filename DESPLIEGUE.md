Documentación técnica del proceso de despliegue de la aplicación PokéDex Angular en Azure Static Web Apps, con pipeline CI/CD mediante GitHub Actions.

📋 Tabla de Contenidos

Pre-requisitos
Preparación del Repositorio
Creación del Recurso en Azure
Configuración del Workflow
Configuración del Token de Despliegue
Configuración de Seguridad HTTP
Verificación del Despliegue
Errores Encontrados y Soluciones
Checklist Final


1. Pre-requisitos
Antes de iniciar, verificar que se cuenta con:
HerramientaVersión mínimaVerificaciónGit2.xgit --versionNode.js16.xnode --versionAngular CLI14.xng versionCuenta Azure—portal.azure.comCuenta GitHub—github.com

2. Preparación del Repositorio
2.1 — Obtener el código fuente
cmdgit clone https://github.com/rcuello/ac4dem1a.git
cd ac4dem1a\sistemas-distribuidos\poke-dex-lab
2.2 — Crear el repositorio propio en GitHub

Ir a github.com → clic en "New"
Configurar:

Repository name: Pokedex
Visibility: Public


Clic en "Create repository"

2.3 — Inicializar el repositorio local
cmdmkdir Pokedex
cd Pokedex
git init
git remote add origin https://github.com/TU-USUARIO/Pokedex.git
2.4 — Copiar los archivos del proyecto
En Windows CMD usar robocopy en lugar de cp:
cmdrobocopy "..\poke-dex-lab" "." /E /XD "..\poke-dex-lab\Pokedex"

⚠️ Importante: No usar cp en CMD de Windows — solo funciona en Linux/Mac.

2.5 — Verificar la estructura del proyecto
cmddir
La estructura esperada es:
Pokedex/
├── source/
│   └── Pokedex/
│       └── pokedex-angular/
│           ├── src/
│           ├── angular.json
│           └── package.json
└── staticwebapp.config.json
2.6 — Primer commit
cmdgit add .
git commit -m "feat: initial deploy - PokeDex app"
git push -u origin master

3. Creación del Recurso en Azure
3.1 — Acceder al Portal de Azure

Ingresar a portal.azure.com
En la barra de búsqueda escribir "Static Web Apps"
Clic en "+ Crear"

3.2 — Configurar el recurso
Pestaña Basics:
CampoValorSuscripciónAzure for StudentsGrupo de recursosrg-pokedex (crear nuevo)Nombrepokedex-pueblopaletaPlan de hospedajeFreeRegiónEast US 2OrigenGitHub
Conexión con GitHub:

Clic en "Iniciar sesión con GitHub"
Autorizar a Azure
Seleccionar:

Organization: tu usuario
Repository: Pokedex
Branch: master



Build Details:
CampoValorBuild PresetsCustomApp locationsource/Pokedex/pokedex-angularApi location(vacío)Output locationdist/pokedex-angular

⚠️ Importante: El campo Output location debe ser un string simple, sin corchetes ni comillas adicionales.

3.3 — Crear el recurso
Clic en "Revisar y crear" → "Crear"
Azure creará automáticamente el archivo de workflow en el repositorio.

4. Configuración del Workflow
Azure genera un archivo en .github/workflows/. Verificar y ajustar su contenido:
cmddir .github\workflows\
type .github\workflows\azure-static-web-apps-*.yml
El archivo debe contener:
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
    if: github.event_name == 'push' || (github.event_name == 'pull_request' && github.event.action != 'closed')
    runs-on: ubuntu-latest
    name: Build and Deploy Job
    steps:
      - uses: actions/checkout@v3
        with:
          submodules: true
          lfs: false

      - name: Build And Deploy
        uses: Azure/static-web-apps-deploy@v1
        with:
          azure_static_web_apps_api_token: ${{ secrets.AZURE_STATIC_WEB_APPS_API_TOKEN_CALM_GLACIER_0B51F6A10 }}
          repo_token: ${{ secrets.GITHUB_TOKEN }}
          action: "upload"
          app_location: "source/Pokedex/pokedex-angular"
          api_location: ""
          output_location: "dist/pokedex-angular"

  close_pull_request_job:
    if: github.event_name == 'pull_request' && github.event.action == 'closed'
    runs-on: ubuntu-latest
    name: Close Pull Request Job
    steps:
      - name: Close Pull Request
        uses: Azure/static-web-apps-deploy@v1
        with:
          azure_static_web_apps_api_token: ${{ secrets.AZURE_STATIC_WEB_APPS_API_TOKEN_CALM_GLACIER_0B51F6A10 }}
          action: "close"
Si se requieren cambios, subir con:
cmdgit add .github\workflows\
git commit -m "fix: update workflow app_location"
git push origin master

5. Configuración del Token de Despliegue
5.1 — Obtener el token desde Azure

Ir al recurso en Azure Portal
Clic en "Administrar token de implementación"
Copiar el token generado

5.2 — Agregar el secret en GitHub

Ir al repositorio en GitHub
Settings → Secrets and variables → Actions
Clic en "New repository secret"
Configurar:

Name: AZURE_STATIC_WEB_APPS_API_TOKEN_CALM_GLACIER_0B51F6A10
Value: pegar el token copiado


Clic en "Add secret"


⚠️ El nombre del secret debe coincidir exactamente con el nombre referenciado en el workflow. Mayúsculas, guiones y números incluidos.

5.3 — Disparar el workflow manualmente
cmdgit commit --allow-empty -m "Trigger workflow after token setup"
git push origin master

6. Configuración de Seguridad HTTP
El archivo staticwebapp.config.json en la raíz del proyecto configura los headers HTTP de seguridad:
json{
  "globalHeaders": {
    "Content-Security-Policy": "default-src 'self'; script-src 'self' 'unsafe-inline'; style-src 'self' 'unsafe-inline' https://fonts.googleapis.com; style-src-elem 'self' 'unsafe-inline' https://fonts.googleapis.com; font-src 'self' https://fonts.gstatic.com data:; img-src 'self' data: https://raw.githubusercontent.com https://pokeapi.co https://assets.pokemon.com https://beta.pokeapi.co; connect-src 'self' https://pokeapi.co https://beta.pokeapi.co https://beta.pokeapi.co/graphql/v1beta; object-src 'none'; base-uri 'self'; form-action 'self'; frame-ancestors 'none'; upgrade-insecure-requests",
    "Strict-Transport-Security": "max-age=31536000; includeSubDomains; preload",
    "X-Content-Type-Options": "nosniff",
    "X-Frame-Options": "DENY",
    "Referrer-Policy": "strict-origin-when-cross-origin",
    "Permissions-Policy": "camera=(), microphone=(), geolocation=(), payment=()",
    "X-XSS-Protection": "1; mode=block"
  },
  "navigationFallback": {
    "rewrite": "/index.html",
    "exclude": ["/images/*.{png,jpg,gif,svg}", "/css/*", "/js/*"]
  }
}
Descripción de cada header:
HeaderPropósitoContent-Security-PolicyPreviene ataques XSS bloqueando recursos no autorizadosStrict-Transport-SecurityFuerza HTTPS por 1 añoX-Content-Type-OptionsBloquea detección automática de MIME typesX-Frame-OptionsImpide que la app sea embebida en iframes (clickjacking)Referrer-PolicyControla información enviada en cabeceras RefererPermissions-PolicyRestringe acceso a APIs del navegadorX-XSS-ProtectionActiva filtro XSS en navegadores legacy

7. Verificación del Despliegue
7.1 — Verificar el pipeline en GitHub Actions

Ir a https://github.com/TU-USUARIO/Pokedex/actions
Confirmar que el workflow más reciente muestra ✅ verde
Si hay errores, revisar los logs expandiendo cada paso

7.2 — Verificar la aplicación en el navegador

Abrir la URL de Azure: https://proud-desert-02d7af910.7.azurestaticapps.net
Confirmar que la aplicación carga correctamente
Abrir DevTools (F12) → consola → verificar que no hay errores

7.3 — Verificar los headers de seguridad

DevTools (F12) → pestaña Network
Recargar la página
Clic en la primera petición → pestaña Headers
En Response Headers deben aparecer:

content-security-policy: default-src 'self'; ...
strict-transport-security: max-age=31536000; ...
x-content-type-options: nosniff
x-frame-options: DENY
referrer-policy: strict-origin-when-cross-origin
permissions-policy: camera=(), ...
7.4 — Auditoría externa con securityheaders.com

Ir a securityheaders.com
Ingresar la URL de la aplicación
Clic en "Scan"
Resultado esperado: calificación A o superior


8. Errores Encontrados y Soluciones
❌ Error 1: cp no reconocido en Windows CMD
"cp" no se reconoce como un comando interno o externo
Causa: cp es un comando exclusivo de Linux/Mac.
Solución:
cmdrobocopy "..\origen" "." /E /XD "..\origen\Pokedex"

❌ Error 2: InvalidTemplate — appArtifactLocation es Array
json"Expected a value of type 'String, Uri', but received a value of type 'Array'"
Causa: El campo Output location en Azure Portal fue ingresado con formato incorrecto.
Solución: Ingresar el valor como string simple:
✅ dist/pokedex-angular
❌ ["dist/pokedex-angular"]

❌ Error 3: Could not detect any platform
Could not detect the language from repo.
Failed to find a default file in the app artifacts folder (/).
Causa: El app_location en el workflow apuntaba a / en lugar de donde está el package.json.
Solución: Actualizar el workflow:
yamlapp_location: "source/Pokedex/pokedex-angular"
output_location: "dist/pokedex-angular"

❌ Error 4: Repositorio Git embebido
warning: adding embedded git repository: source/Pokedex
fatal: No url found for submodule path 'source/Pokedex' in .gitmodules
Causa: La carpeta source/Pokedex tenía su propio .git, creando un submodule no intencionado.
Solución:
cmdrmdir /s /q source\Pokedex\.git
git rm --cached source/Pokedex
git add source/Pokedex
git commit -m "Fix: remove embedded git repository"
git push origin master --force

❌ Error 5: deployment_token was not provided
The deployment_token is required for deploying content.
Causa: El nombre del secret en GitHub no coincidía exactamente con el nombre en el workflow.
Causa frecuente: Se creó el secret como AZURE_STATIC_WEB_APPS_API_TOKEN pero el workflow referenciaba AZURE_STATIC_WEB_APPS_API_TOKEN_CALM_GLACIER_0B51F6A10.
Solución: Crear el secret con el nombre exacto del workflow:
AZURE_STATIC_WEB_APPS_API_TOKEN_CALM_GLACIER_0B51F6A10

❌ Error 6: Push rechazado — fetch first
Updates were rejected because the remote contains work that you do not have locally
Causa: El repositorio remoto tenía commits (como el workflow generado por Azure) que no estaban en local.
Solución:
cmdgit pull origin master --allow-unrelated-histories
git push origin master

❌ Error 7: Workflow no se dispara tras un push
Causa: El push fue rechazado y luego sincronizado sin generar un nuevo commit. GitHub solo dispara el workflow ante nuevos commits.
Solución: Crear un commit vacío para forzar el trigger:
cmdgit commit --allow-empty -m "Trigger workflow"
git push origin master

9. Checklist Final
Antes de entregar, verificar:

 La aplicación carga en la URL pública sin errores
 No hay errores en la consola del navegador (F12)
 HTTPS activo (candado 🔒 en la barra de direcciones)
 Los headers de seguridad son visibles en DevTools → Network → Headers
 El escaneo en securityheaders.com muestra calificación A o superior
 README.md y DESPLIEGUE.md están en el repositorio
 staticwebapp.config.json está en la raíz del repositorio
 El workflow de GitHub Actions muestra ✅ en la última ejecución


🔗 Recursos de Referencia

Documentación Azure Static Web Apps
Configuración de headers en Azure SWA
securityheaders.com
Angular CLI — Documentación oficial
GitHub Actions — Documentación


Desarrollado como parte del curso de Sistemas Distribuidos.
