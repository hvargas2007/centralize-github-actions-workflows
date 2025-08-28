# Workflows Centralizados de GitHub Actions para Terraform

[![Licencia: MIT](https://img.shields.io/badge/Licencia-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Terraform](https://img.shields.io/badge/terraform-%235835CC.svg?style=flat&logo=terraform&logoColor=white)](https://www.terraform.io/)
[![GitHub Actions](https://img.shields.io/badge/github%20actions-%232671E5.svg?style=flat&logo=githubactions&logoColor=white)](https://github.com/features/actions)
[![AWS](https://img.shields.io/badge/AWS-%23FF9900.svg?style=flat&logo=amazon-aws&logoColor=white)](https://aws.amazon.com/)

Una colección de GitHub Actions reutilizables y listos para producción para la automatización de infraestructura Terraform. Optimiza tus pipelines CI/CD con workflows probados en batalla para planificar, aplicar y destruir infraestructura en múltiples ambientes.

## 🎯 Visión General

Este repositorio proporciona Actions de GitHub centralizados y reutilizables que permiten a los equipos:
- Estandarizar los workflows de Terraform en múltiples repositorios
- Implementar automáticamente las mejores prácticas de seguridad
- Reducir la duplicación de código en pipelines CI/CD
- Asegurar procesos consistentes de despliegue de infraestructura

## 📋 Tabla de Contenidos

- [Características](#características)
- [Actions Disponibles](#actions-disponibles)
- [Requisitos Previos](#requisitos-previos)
- [Inicio Rápido](#inicio-rápido)
- [Ejemplos de Uso](#ejemplos-de-uso)
- [Configuración](#configuración)
- [Seguridad](#seguridad)
- [Solución de Problemas](#solución-de-problemas)
- [Contribuir](#contribuir)
- [Licencia](#licencia)

## ✨ Características

### Capacidades Principales
- **🔄 Workflows Reutilizables**: Fuente única de verdad para todas las operaciones de Terraform
- **🛡️ Escaneo de Seguridad**: Integración incorporada con Checkov, tflint y TFSec
- **💰 Gestión de Costos**: Estimación automática de costos con Infracost
- **📊 Integración con Pull Request**: Comentarios detallados del plan en PRs
- **🔐 Autenticación OIDC**: Autenticación segura de AWS sin credenciales de larga duración
- **🌍 Soporte Multi-Región**: Despliega en cualquier región de AWS
- **📦 Soporte de Módulos**: Maneja módulos de Terraform públicos y privados
- **🎨 Personalizable**: Configuración flexible para diferentes casos de uso

### Características de Seguridad
- Escaneo automatizado de seguridad con múltiples herramientas
- Prevención de escaneo de secretos
- Generación de reportes SARIF para la pestaña de Seguridad de GitHub
- Aplicación de políticas de seguridad configurables
- Verificación de cumplimiento para estándares de la industria

## 📦 Actions Disponibles

### 1. Terraform Plan (`terraform/plan`)
Valida y planifica cambios de infraestructura con análisis completo de seguridad y costos.

**Características:**
- Escaneo de seguridad (Checkov, tflint, TFSec)
- Estimación y comparación de costos
- Comentarios detallados en PR con la salida del plan
- Detección de deriva
- Validación de políticas

### 2. Terraform Apply (`terraform/apply`)
Aplica de forma segura los cambios de infraestructura con salvaguardas integradas.

**Características:**
- Verificación automática del plan
- Confirmación de apply
- Bloqueo de estado
- Captura de salida y almacenamiento de artefactos
- Capacidades de rollback

### 3. Terraform Destroy (`terraform/destroy`)
Destrucción controlada de infraestructura con verificaciones de seguridad.

**Características:**
- Vista previa del plan de destrucción
- Requisito de aprobación manual
- Manejo de dependencias de recursos
- Respaldo de estado antes de la destrucción

## 🔧 Requisitos Previos

### Requisitos del Repositorio
- Repositorio de GitHub con Actions habilitadas
- Código Terraform (>= 1.0.0)
- Configuración de backend (S3, Azure Storage o GCS)

### Requisitos AWS
- Cuenta AWS con permisos apropiados
- Proveedor OIDC configurado para GitHub Actions
- Rol IAM con relación de confianza a GitHub

### Secrets Requeridos
Configura estos en la configuración de tu repositorio:

```yaml
secrets:
  ACCESS_TOKEN          # Token de Acceso Personal de GitHub (para módulos privados)
  USERNAME_GITHUB       # Nombre de usuario de GitHub (para módulos privados)

variables:
  AWS_ROLE             # ARN del rol IAM a asumir
  AWS_REGION           # Región AWS objetivo
  AWS_BACKEND          # String de configuración del backend
  PROJECT_NAME         # Identificador del proyecto para etiquetado
```

## 🚀 Inicio Rápido

### Paso 1: Configurar tu Workflow

Crea `.github/workflows/terraform.yml` en tu repositorio:

```yaml
name: Terraform CI/CD

on:
  pull_request:
    branches: [main, develop]
  push:
    branches: [main, develop]

jobs:
  plan:
    if: github.event_name == 'pull_request'
    uses: hvargas2007/centralize-github-actions-workflows/.github/actions/terraform/plan@main
    with:
      ACCESS_TOKEN: ${{ secrets.ACCESS_TOKEN }}
      USERNAME_GITHUB: ${{ secrets.USERNAME_GITHUB }}
      AWS_ROLE: ${{ vars.AWS_ROLE }}
      AWS_REGION: ${{ vars.AWS_REGION }}
      AWS_BACKEND: ${{ vars.AWS_BACKEND }}
      PROJECT_NAME: ${{ vars.PROJECT_NAME }}
      RULES_TO_SKIP_CHECKOV: "CKV_TF_1"
    secrets: inherit

  apply:
    if: github.event_name == 'push'
    uses: hvargas2007/centralize-github-actions-workflows/.github/actions/terraform/apply@main
    with:
      ACCESS_TOKEN: ${{ secrets.ACCESS_TOKEN }}
      USERNAME_GITHUB: ${{ secrets.USERNAME_GITHUB }}
      AWS_ROLE: ${{ vars.AWS_ROLE }}
      AWS_REGION: ${{ vars.AWS_REGION }}
      AWS_BACKEND: ${{ vars.AWS_BACKEND }}
      PROJECT_NAME: ${{ vars.PROJECT_NAME }}
    secrets: inherit
```

### Paso 2: Configurar AWS OIDC

1. Crea un proveedor de identidad OIDC en AWS:
```bash
aws iam create-open-id-connect-provider \
  --url https://token.actions.githubusercontent.com \
  --client-id-list sts.amazonaws.com \
  --thumbprint-list 6938fd4d98bab03faadb97b34396831e3780aea1
```

2. Crea un rol IAM con política de confianza:
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Federated": "arn:aws:iam::ACCOUNT_ID:oidc-provider/token.actions.githubusercontent.com"
      },
      "Action": "sts:AssumeRoleWithWebIdentity",
      "Condition": {
        "StringEquals": {
          "token.actions.githubusercontent.com:aud": "sts.amazonaws.com",
          "token.actions.githubusercontent.com:sub": "repo:TU_ORG/TU_REPO:ref:refs/heads/main"
        }
      }
    }
  ]
}
```

### Paso 3: Configurar el Repositorio

Configura los secrets y variables de tu repositorio en Configuración → Secrets y variables → Actions.

## 📚 Ejemplos de Uso

### Despliegue Multi-Ambiente

```yaml
name: Terraform Multi-Ambiente

on:
  push:
    branches: [dev, staging, production]

jobs:
  deploy:
    uses: hvargas2007/centralize-github-actions-workflows/.github/actions/terraform/apply@main
    with:
      ENVIRONMENT: ${{ github.ref_name }}
      AWS_ROLE: ${{ vars[format('AWS_ROLE_{0}', github.ref_name)] }}
      AWS_REGION: ${{ vars.AWS_REGION }}
      AWS_BACKEND: ${{ vars[format('AWS_BACKEND_{0}', github.ref_name)] }}
      PROJECT_NAME: ${{ vars.PROJECT_NAME }}
    secrets: inherit
```

### Con Reglas de Seguridad Personalizadas

```yaml
jobs:
  secure-plan:
    uses: hvargas2007/centralize-github-actions-workflows/.github/actions/terraform/plan@main
    with:
      AWS_ROLE: ${{ vars.AWS_ROLE }}
      AWS_REGION: ${{ vars.AWS_REGION }}
      RULES_TO_SKIP_CHECKOV: "CKV_AWS_1,CKV_AWS_2"
      ENABLE_TFSEC: true
      TFSEC_SEVERITY: "CRITICAL,HIGH"
      CUSTOM_CHECKOV_POLICIES: "./policies"
    secrets: inherit
```

### Detección de Deriva Programada

```yaml
name: Detección de Deriva

on:
  schedule:
    - cron: '0 8 * * MON'  # Todos los lunes a las 8 AM

jobs:
  detect-drift:
    uses: hvargas2007/centralize-github-actions-workflows/.github/actions/terraform/plan@main
    with:
      AWS_ROLE: ${{ vars.AWS_ROLE }}
      AWS_REGION: ${{ vars.AWS_REGION }}
      DRIFT_DETECTION: true
      SEND_SLACK_NOTIFICATION: true
      SLACK_WEBHOOK: ${{ secrets.SLACK_WEBHOOK }}
    secrets: inherit
```

## ⚙️ Configuración

### Parámetros de Entrada

| Parámetro | Requerido | Por Defecto | Descripción |
|-----------|-----------|-------------|-------------|
| `ACCESS_TOKEN` | No | - | PAT de GitHub para módulos privados |
| `USERNAME_GITHUB` | No | - | Usuario de GitHub para autenticación |
| `AWS_ROLE` | Sí | - | ARN del rol IAM a asumir |
| `AWS_REGION` | Sí | `us-east-1` | Región AWS para recursos |
| `AWS_BACKEND` | Sí | - | Configuración backend de Terraform |
| `PROJECT_NAME` | Sí | - | Identificador del proyecto para etiquetado |
| `TERRAFORM_VERSION` | No | `latest` | Versión de Terraform a usar |
| `WORKING_DIRECTORY` | No | `.` | Directorio con código Terraform |
| `RULES_TO_SKIP_CHECKOV` | No | - | Reglas Checkov a omitir (separadas por comas) |
| `ENABLE_INFRACOST` | No | `true` | Habilitar estimación de costos |
| `VAR_FILE` | No | - | Ruta al archivo terraform.tfvars |
| `ADDITIONAL_ARGS` | No | - | Argumentos adicionales de Terraform |

### Ejemplos de Configuración del Backend

#### Backend S3
```hcl
terraform {
  backend "s3" {
    bucket         = "mi-terraform-state"
    key            = "infrastructure/terraform.tfstate"
    region         = "us-east-1"
    dynamodb_table = "terraform-locks"
    encrypt        = true
  }
}
```

#### Backend Azure
```hcl
terraform {
  backend "azurerm" {
    resource_group_name  = "terraform-state-rg"
    storage_account_name = "tfstate12345"
    container_name      = "tfstate"
    key                 = "terraform.tfstate"
  }
}
```

## 🔐 Seguridad

### Herramientas de Escaneo de Seguridad

1. **Checkov**: Escaneo completo de seguridad de infraestructura
2. **tflint**: Reglas de linting y seguridad específicas de Terraform
3. **TFSec**: Escáner de seguridad para plantillas Terraform
4. **Terrascan**: Política como código para seguridad de infraestructura

### Mejores Prácticas

- Usa OIDC para autenticación (sin credenciales de larga duración)
- Habilita todos los escáneres de seguridad en workflows de producción
- Actualiza regularmente la versión del action para últimas correcciones de seguridad
- Implementa políticas IAM de menor privilegio
- Usa cuentas AWS separadas para diferentes ambientes
- Habilita el cifrado del archivo de estado
- Implementa workflows de aprobación para producción

### Cumplimiento

Los workflows soportan escaneo de cumplimiento para:
- Benchmarks CIS
- PCI DSS
- HIPAA
- SOC2
- Políticas organizacionales personalizadas

## 🔍 Solución de Problemas

### Problemas Comunes

#### 1. Falla la Autenticación OIDC
```
Error: Could not assume role with OIDC
```
**Solución:** Verifica la política de confianza y el thumbprint de OIDC de GitHub

#### 2. Falla la Descarga del Módulo
```
Error: Failed to download module
```
**Solución:** Verifica ACCESS_TOKEN y permisos del repositorio del módulo

#### 3. Error de Inicialización del Backend
```
Error: Backend initialization required
```
**Solución:** Asegúrate de que la variable AWS_BACKEND esté configurada correctamente

#### 4. Fallos en el Escaneo de Seguridad
```
Error: Security scan found critical issues
```
**Solución:** Revisa los hallazgos y corrígelos o agrégalos a la lista de omisión

### Modo Debug

Habilita la salida debug configurando:
```yaml
env:
  ACTIONS_STEP_DEBUG: true
  ACTIONS_RUNNER_DEBUG: true
```

## 📊 Estado del Proyecto

![Versión](https://img.shields.io/github/v/release/hvargas2007/centralize-github-actions-workflows)
![Estado de Build](https://img.shields.io/github/workflow/status/hvargas2007/centralize-github-actions-workflows/CI)
![Licencia](https://img.shields.io/github/license/hvargas2007/centralize-github-actions-workflows)
![Contribuidores](https://img.shields.io/github/contributors/hvargas2007/centralize-github-actions-workflows)

## 👨‍💻 Autor

**Hermes Vargas**  
📧 Email: hermesvargas200720@gmail.com  
🔗 GitHub: [@hvargas2007](https://github.com/hvargas2007)

---

Hecho con ❤️ para toda la comunidad