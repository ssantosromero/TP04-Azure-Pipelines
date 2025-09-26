# Decisiones de Desarrollo - TP04 Azure DevOps Pipelines

## 1. Preparación del Entorno

1) Creé Pool y Self-Hosted Agent:
```bash
# En Azure DevOps > Project Settings > Agent pools
# Pool name: Default
# Agent name: Santos-MacBook
```

2) Configuré el agent en mi MacBook:
```bash
cd ~/myagent
./config.sh
# Server URL: https://dev.azure.com/tp03-ingsoft-RomeroReyna
# Authentication: PAT (Personal Access Token)
# Agent pool: Default
# Agent name: Santos-MacBook
./run.sh
```

**Problemas encontrados:** macOS bloqueó inicialmente Agent.Listener y libhostfxr.dylib por seguridad. Solucioné permitiendo desde Configuración del Sistema > Privacidad y Seguridad.

## 2. Estructura del Repo y Aplicación

3) Creé estructura mono-repo:
```bash
mkdir ~/TP04-Pipelines && cd ~/TP04-Pipelines
mkdir front back
```

4) Frontend (HTML + JavaScript):
```bash
# front/index.html - Interfaz web con estilos CSS
# front/package.json - Scripts de build
```

**Justificación:** Frontend simple pero funcional que demuestra conexión con backend y maneja errores adecuadamente.

5) Backend (Node.js + Express):
```bash
cd back
npm init -y
npm install express cors
# server.js - API REST con endpoints / y /health
```

**Stack elegido:** Node.js + Express
**Por qué:** Rápido de configurar, ideal para CI/CD, compatible con npm scripts que Azure Pipelines maneja nativamente.

## 3. Pipeline YAML Multi-Stage

6) Creé azure-pipelines.yml en la raíz:
```yaml
trigger:
  branches:
    include:
      - main

pool:
  name: Default  # Self-hosted agent

stages:
- stage: CI
  displayName: 'Continuous Integration'
  jobs:
  - job: BuildFrontend
    displayName: 'Build Frontend'
    steps:
    - script: |
        cd front
        npm run build
      displayName: 'Build Frontend'
    
    - task: PublishBuildArtifacts@1
      inputs:
        PathtoPublish: 'front/dist'
        ArtifactName: 'frontend-dist'

  - job: BuildBackend
    displayName: 'Build Backend'  
    steps:
    - script: |
        cd back
        npm install
        npm test
        npm run build
      displayName: 'Build and Test Backend'
    
    - task: PublishBuildArtifacts@1
      inputs:
        PathtoPublish: 'back'
        ArtifactName: 'backend-dist'
```

**Diseño del Pipeline:**
- **Trigger en main:** Automático en cada push a rama principal
- **Multi-stage:** Un stage CI con dos jobs paralelos
- **Self-hosted:** Corre en mi MacBook (Santos-MacBook)
- **Artefactos:** Frontend dist/ y Backend completo publicados

## 4. Ejecuciones y Resultados

7) Pipeline ejecutado exitosamente:
```bash
Pipeline #20250925.6: ✅ Succeeded (1m 32s)
Jobs ejecutados en self-hosted agent:
- Build Frontend: ✅ Succeeded  
- Build Backend: ✅ Succeeded
- Artefactos publicados: ✅ frontend-dist, backend-dist
```

**Evidencia de funcionamiento:**
- ✅ Self-hosted agent corriendo en terminal
- ✅ Jobs tomados y ejecutados por Santos-MacBook
- ✅ Logs visibles en Azure DevOps
- ✅ Artefactos generados y publicados
- ✅ Tiempo de ejecución: 1m 32s

## 5. Problemas Encontrados y Soluciones

### Problema 1: Bloqueo de seguridad macOS
**Issue:** macOS bloqueó Agent.Listener y libhostfxr.dylib
**Solución:** 
```bash
# Permitir desde Configuración del Sistema
# Privacidad y Seguridad > Permitir de todas formas
xattr -d com.apple.quarantine ~/myagent
```

### Problema 2: Jobs en cola inicial
**Issue:** Pipeline quedaba en "Queued"
**Solución:** Verificar que agent pool en YAML coincida con pool real del agent (ambos "Default")

### Problema 3: npm scripts faltantes
**Issue:** Frontend no tenía script de build
**Solución:** 
```bash
npm pkg set scripts.build="mkdir -p dist && cp index.html dist/"
npm pkg set scripts.test="echo 'Frontend tests OK'"
```

## 6. Reflexión: CI/CD en Equipos Reales

### Lo que aplicaría del TP:
- **Pipeline como código** YAML versionado en el repo para reproducibilidad
- **Self-hosted agents** para control total del entorno (SDKs específicos, bases de datos locales)
- **Multi-stage pipelines** separando build/test/deploy claramente
- **Artefactos publicados** para trazabilidad entre stages

### Herramientas que investigaría:
- **Azure Artifacts** para gestión de paquetes NuGet/npm privados
- **Deployment slots** en Azure App Service para blue-green deployments
- **Infrastructure as Code** con Azure Resource Manager templates
- **Security scanning** integrado con WhiteSource/SonarQube

### Para proyectos grandes:
- **Conditional triggers** por carpetas modificadas (trigger solo si cambió /back o /front)
- **Matrix builds** para múltiples versiones Node.js simultáneas
- **Deployment gates** con aprobaciones manuales para producción
- **Rollback strategies** automáticos si health checks fallan

**Tiempo total invertido:** ~1.5 horas
**Valor del ejercicio:** Dominar CI/CD end-to-end desde código hasta artefactos, usando self-hosted infrastructure que simula entornos empresariales reales.
