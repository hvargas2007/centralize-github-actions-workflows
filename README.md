
# 🤖 Reusable GitHub Actions Workflows - CloudHesive

Este repositorio centraliza workflows reutilizables de GitHub Actions para ser utilizados en múltiples proyectos dentro de la organización **cloudhesive**. Están diseñados para estandarizar, simplificar y acelerar pipelines de CI/CD, especialmente para entornos basados en Terraform y AWS.

---

## 📁 Estructura del repositorio

```
.github
└── actions
    └── terraform
        ├── apply
        │   └── action.yml
        ├── destroy
        │   └── action.yml
        └── plan
            └── action.yml
```

Cada subcarpeta contiene un `action.yml` que define un job reutilizable (`plan`, `apply` o `destroy`), orientado a Terraform en AWS.

---

## 🔐 Permisos necesarios

### En este repositorio (centralizado):
- **Access**: `Accessible from repositories in the 'cloudhesive' organization`
- **Actions Permissions**: `Allow cloudhesive actions and reusable workflows`

Estos ajustes permiten que los demás repositorios privados dentro de la organización accedan y ejecuten los workflows definidos aquí.

---

## 🚀 ¿Cómo usar un workflow reutilizable?

Desde otro repositorio dentro de la organización, puedes referenciar un workflow como este:

```yaml
jobs:
  terraform_plan:
    uses: cloudhesive/centralize-github-actions-workflows/.github/actions/terraform/plan@main
    with:
      ACCESS_TOKEN: ${{ secrets.ACCESS_TOKEN }}
      AWS_ROLE: arn:aws:iam::123456789012:role/TerraformRole
      AWS_REGION: us-east-1
      AWS_BACKEND: s3
      PROJECT_NAME: example-project
      USERNAME_GITHUB: ${{ github.actor }}
      RULES_TO_SKIP_CHECKOV: CKV_AWS_20,CKV_AWS_21
```

> 🔁 Puedes cambiar `plan` por `apply` o `destroy` según el objetivo.

---

## ⚙️ Inputs disponibles

Todos los workflows siguen un set estándar de entradas:

| Nombre                 | Requerido | Descripción                                                                 |
|------------------------|-----------|-----------------------------------------------------------------------------|
| `ACCESS_TOKEN`         | ✅        | Token de GitHub para operaciones autenticadas                              |
| `USERNAME_GITHUB`      | ✅        | Usuario que ejecuta la acción                                               |
| `AWS_ROLE`             | ✅        | ARN del rol de AWS a asumir                                                 |
| `AWS_REGION`           | ✅        | Región de AWS (ej. `us-east-1`)                                             |
| `AWS_BACKEND`          | ✅        | Tipo de backend de Terraform (ej. `s3`)                                     |
| `PROJECT_NAME`         | ✅        | Nombre del proyecto Terraform                                               |
| `RULES_TO_SKIP_CHECKOV`| ❌        | IDs de reglas Checkov a omitir (ej. `CKV_AWS_20,CKV_AWS_21`)                |

---

## 📌 Buenas prácticas

- Usa `@main` para desarrollo o `@v1`, `@v2`, etc. con tags para producción.
- Versiona los cambios importantes en workflows y documenta en cada PR.
- Mantén el formato de entradas (`inputs`) claro y bien tipado.
- Asegúrate de que el workflow que consume tenga configurado `permissions` correctos y secrets necesarios.

---

## 👤 Autor

📧 hermesvargas200720@gmail.com

---

## ⚠️ Requisitos en los repositorios consumidores

Los repositorios que usen estas actions deben incluir:

1. **state-checker.py**: Script que crea el bucket S3 para el backend de Terraform
2. **provider.tf**: Archivo con placeholders `BUCKET_NAME` y `PROJECT_NAME` que serán reemplazados
3. **env/**: Directorio con archivos de variables por ambiente (`dev.auto.tfvars`, `qa.auto.tfvars`, `main.auto.tfvars`)

---
