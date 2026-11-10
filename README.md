# Plantilla DevOps — Electiva 1 (ITLA 2026-C-3)

Aplicación web mínima (lista de tareas) con todo lo necesario para practicar un pipeline **CI/CD** sin cuentas pagas ni Azure:

| Pieza | Archivo | Para qué sirve |
|---|---|---|
| App Node.js/Express | `src/app.js`, `src/server.js` | API de tareas + `/health` y `/metrics` |
| Frontend estático | `public/index.html` | Se publica en GitHub Pages |
| Pruebas automatizadas | `tests/app.test.js` | Jest + Supertest con cobertura |
| Contenedor | `Dockerfile` | Imagen de producción con *healthcheck* |
| Pipeline | `.github/workflows/ci.yml` | Lint → pruebas → build Docker → deploy a Pages |

## Requisitos (gratis)
- Cuenta de **GitHub** (usa un repositorio **público**: GitHub Actions es gratuito).
- **Git** y **Node.js 20+**. Opcional para probar el contenedor en tu PC: **Docker Desktop**.

## Ejecutar en tu computadora
```bash
npm install
npm test          # pruebas + cobertura
npm start         # http://localhost:3000
```
Endpoints: `GET /health`, `GET /metrics`, `GET/POST /api/tareas`, `PATCH /api/tareas/:id` y `DELETE /api/tareas/:id`. Los títulos son obligatorios y pueden tener hasta 100 caracteres.

Con Docker:
```bash
docker build -t tareas-devops .
docker run -p 3000:3000 tareas-devops
```

## Subirlo a GitHub y activar el pipeline
1. Crea un repositorio **público** vacío en GitHub (sin README).
2. En la carpeta de la plantilla:
   ```bash
   git init
   git add .
   git commit -m "Plantilla inicial"
   git branch -M main
   git remote add origin https://github.com/TU_USUARIO/TU_REPO.git
   git push -u origin main
   ```
3. En GitHub: **Settings → Pages → Source: GitHub Actions**.
4. Abre la pestaña **Actions**: verás el workflow *CI/CD* ejecutándose. Cuando termine, la URL de tu página aparece en el job *Despliegue a GitHub Pages*.

## Práctica 3 — qué debes agregar
- Al menos **2 pruebas nuevas** (por ejemplo: borrar tareas, validar títulos largos).
- Una funcionalidad nueva pequeña con su prueba (`DELETE /api/tareas/:id`).
- Provocar a propósito una prueba fallida, mostrar que el pipeline se pone en **rojo**, corregirla y mostrarlo en **verde**.
- Completar este README con la explicación de cada etapa del pipeline.

### Funcionalidad y pruebas agregadas

Se agregó `DELETE /api/tareas/:id`. Cuando el identificador existe, elimina la tarea y responde con un mensaje y los datos de la tarea eliminada. Si no existe, responde con código HTTP `404`. También se validó que el título de una tarea no supere los 100 caracteres; los títulos vacíos o compuestos solo por espacios tampoco son aceptados.

Las pruebas nuevas comprueban el rechazo de títulos demasiado largos, la eliminación correcta de una tarea y la respuesta `404` al intentar eliminar una tarea inexistente. Jest ejecuta estas pruebas junto con las originales y genera el reporte de cobertura.


### Funcionamiento del frontend en GitHub Pages

El frontend cuenta con dos modos de funcionamiento. Cuando se ejecuta la aplicación localmente y está disponible el servidor Express, utiliza la API para administrar las tareas.

Cuando se publica en GitHub Pages, utiliza `localStorage` para guardar las tareas directamente en el navegador, sin necesidad de tener un servidor Node.js funcionando.

En este segundo modo se pueden agregar tareas, marcarlas como completadas y eliminar todas las tareas completadas. La información permanece guardada en el navegador aunque se cierre la página.

Es importante aclarar que las tareas almacenadas en GitHub Pages son independientes de las que se guardan mediante la API local.


### ¿Cómo funciona el pipeline CI/CD?

El workflow está definido en `.github/workflows/ci.yml` y se activa automáticamente cuando se realiza un `push` a la rama `main` o se abre/actualiza un `pull request` dirigido a `main`.

1. **Checkout:** GitHub Actions descarga el contenido del repositorio para trabajar con el código.
2. **Preparación de Node.js:** configura Node.js 20 en el entorno de ejecución.
3. **Instalación:** `npm install` instala Express y las dependencias de desarrollo necesarias.
4. **Revisión estática (lint):** `npm run lint` utiliza `node --check` para comprobar que los archivos principales de la aplicación no tengan errores de sintaxis.
5. **Pruebas automatizadas:** `npm test` ejecuta Jest y Supertest, y genera el reporte de cobertura. Si una prueba falla, el trabajo se detiene y el pipeline queda marcado como fallido.
6. **Construcción Docker:** `docker build` comprueba que la aplicación pueda empaquetarse en una imagen.
7. **Despliegue:** únicamente después de que el trabajo de build y pruebas termine correctamente, y cuando el evento sea un `push` a `main`, se publica la carpeta `public` en GitHub Pages.

El despliegue a Pages publica solamente el frontend estático. GitHub Pages no ejecuta el servidor Node.js/Express ni aloja la API. Para probar todos los endpoints se debe ejecutar la aplicación localmente con `npm start` o mediante Docker.

### Cómo comprobar el pipeline

1. Sube los cambios a la rama `main` o crea un pull request hacia esa rama.
2. En GitHub, abre la pestaña **Actions** y selecciona la ejecución llamada **CI/CD**.
3. Comprueba que las etapas de revisión, pruebas y construcción Docker finalicen correctamente.
4. Para demostrar una ejecución fallida, introduce temporalmente una expectativa incorrecta en una prueba, por ejemplo espera un código `201` en una prueba que debe responder `400`. Sube ese cambio y verifica que el paso de pruebas falle. No dejes ese error en la versión final.
5. Corrige la expectativa, vuelve a subir los cambios y comprueba que el workflow finalice en verde.
6. En una ejecución exitosa por `push` a `main`, revisa el job **Despliegue a GitHub Pages** para abrir la página publicada.

### Guía breve de contribución

- Trabaja en una rama para cada cambio y utiliza nombres descriptivos.
- Escribe mensajes de commit claros, por ejemplo: `Agregar endpoint para eliminar tareas`.
- Antes de subir cambios, ejecuta `npm test` y `npm run lint`.
- Abre un pull request hacia `main` y revisa que GitHub Actions termine correctamente antes de integrar los cambios.

### Instrucciones de despliegue

1. En el repositorio de GitHub, entra en **Settings → Pages**.
2. En **Build and deployment**, selecciona **GitHub Actions** como fuente.
3. Envía los cambios a `main`.
4. Espera a que finalice el workflow **CI/CD**.
5. Abre **Settings → Pages** o el job de despliegue en Actions para consultar la URL publicada.

Recuerda: el sitio de GitHub Pages sirve los archivos de `public`; la API de Express requiere un servidor Node.js activo y no queda desplegada en Pages.

## Proyecto Final — cómo extenderla
- Persistencia con **SQLite** (o MongoDB) en lugar del arreglo en memoria.
- Publicar la imagen en **GitHub Container Registry** (`ghcr.io`) desde el pipeline.
- Análisis estático real (ESLint) y umbral de cobertura mínima.
- Monitoreo: usar `/metrics` y los logs JSON; definir al menos una **alerta** (ej. errores > 0).
- Documentar: guía de instalación, manual de operaciones y lecciones aprendidas.
