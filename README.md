# GitHub Actions Workflows para Node.js / Fastify

Este repositorio contiene un conjunto de **Workflows Reutilizables de GitHub Actions** diseñados para estandarizar el proceso de Integración y Entrega Continua (CI/CD) de los proyectos Node.js, en especial aquellos desarrollados con el framework Fastify.

La idea es mantener centralizada la lógica de CI/CD para que múltiples proyectos puedan aprovecharla y mantenerse actualizados de forma sencilla.

## 🚀 Workflows Disponibles

El proyecto expone los siguientes flujos que pueden ser importados en cualquier repositorio:

1. **`ci-node.yml` (Orquestador principal)**: Llama de manera secuencial a los pasos de Linter, Tests y por último, si todo fue exitoso, empaqueta la aplicación en Docker y la sube al registry.
2. **`node-lint.yml`**: Ejecuta `npm run lint`.
3. **`node-test.yml`**: Ejecuta `npm test`.
4. **`docker-publish.yml`**: Construye el archivo `Dockerfile` y empuja (push) la imagen a GitHub Container Registry (`ghcr.io`).

## 📦 Cómo usar el Workflow Principal (`ci-node.yml`)

Para usar este workflow en tu proyecto de Fastify (o cualquier proyecto de Node.js), solo debes crear un archivo en tu repositorio destino (por ejemplo `.github/workflows/ci.yml`) con el siguiente contenido:

```yaml
name: CI/CD Pipeline

on:
  push:
    branches: [ "main" ]
  pull_request:
    branches: [ "main" ]

jobs:
  build-and-deploy:
    # ⚠️ IMPORTANTE: Otorga permisos para subir la imagen a GitHub Packages (GHCR)
    permissions:
      contents: read
      packages: write

    # Recuerda usar la rama adecuada (ej. @main) y colocar tu nombre de usuario de GitHub
    uses: TU_USUARIO_U_ORGANIZACION/github-actions-workflows/.github/workflows/ci-node.yml@main
    with:
      # Opcional: Versión de Node.js (por defecto es 22.x)
      node-version: '22.x'
      
      # Opcional: Ruta hacia tu Dockerfile (por defecto es ./Dockerfile)
      dockerfile-path: './Dockerfile'
    
    # Opcional: Si tus tests necesitan acceder a secretos del repositorio destino, 
    # puedes pasarlos todos con 'inherit'
    # secrets: inherit
```

> **Nota sobre el nombre de la imagen Docker**: El workflow tomará automáticamente el nombre de tu repositorio destino (en minúsculas) y generará la imagen con él. Ejemplo: Si tu proyecto se llama `Mi-Org/API-Usuarios`, la imagen Docker se llamará `ghcr.io/mi-org/api-usuarios`.

## ⚙️ Configuración Importante de Permisos

Dado que el paso final de este workflow empuja una imagen hacia **GitHub Container Registry (GHCR)** utilizando el `GITHUB_TOKEN` integrado, el repositorio que invoca este workflow **debe tener permisos de escritura sobre los paquetes**.

Si el pipeline te arroja un error como `The nested job 'docker-build-push' is requesting 'packages: write', but is only allowed 'packages: read'`, tienes dos formas de solucionarlo:

### Opción 1: En tu archivo `ci.yml` (Recomendado)
Agrega explícitamente los permisos al job que llama al workflow reutilizable (como se muestra en el ejemplo de arriba):

```yaml
    permissions:
      contents: read
      packages: write
```

### Opción 2: A nivel del repositorio
Si prefieres configurarlo para todos los workflows del repositorio:

1. Ve a la pestaña **Settings** del repositorio destino.
2. En el menú izquierdo, ve a **Actions** > **General**.
3. Baja hasta la sección que dice **Workflow permissions**.
4. Selecciona la opción **Read and write permissions**.
5. Guarda los cambios.

## 🧩 Usar los Jobs de forma independiente

Si en algún repositorio no necesitas todo el flujo (por ejemplo, es una librería donde no necesitas Docker, solo correr tests), puedes invocar los flujos por separado:

```yaml
jobs:
  test-only:
    uses: TU_USUARIO_U_ORGANIZACION/github-actions-workflows/.github/workflows/node-test.yml@main
    with:
      node-version: '22.x'
```
