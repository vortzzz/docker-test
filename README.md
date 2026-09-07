# Getting Started with Create React App

This project was bootstrapped with [Create React App](https://github.com/facebook/create-react-app).
hi

## Available Scripts

In the project directory, you can run:

### `npm start`

Runs the app in the development mode.\
Open [http://localhost:3000](http://localhost:3000) to view it in your browser.

The page will reload when you make changes.\
You may also see any lint errors in the console.

### `npm test`

Launches the test runner in the interactive watch mode.\
See the section about [running tests](https://facebook.github.io/create-react-app/docs/running-tests) for more information.

### `npm run build`

Builds the app for production to the `build` folder.\
It correctly bundles React in production mode and optimizes the build for the best performance.

The build is minified and the filenames include the hashes.\
Your app is ready to be deployed!

See the section about [deployment](https://facebook.github.io/create-react-app/docs/deployment) for more information.

### `npm run eject`

**Note: this is a one-way operation. Once you `eject`, you can't go back!**

If you aren't satisfied with the build tool and configuration choices, you can `eject` at any time. This command will remove the single build dependency from your project.

Instead, it will copy all the configuration files and the transitive dependencies (webpack, Babel, ESLint, etc) right into your project so you have full control over them. All of the commands except `eject` will still work, but they will point to the copied scripts so you can tweak them. At this point you're on your own.

You don't have to ever use `eject`. The curated feature set is suitable for small and middle deployments, and you shouldn't feel obligated to use this feature. However we understand that this tool wouldn't be useful if you couldn't customize it when you are ready for it.

## Learn More

You can learn more in the [Create React App documentation](https://facebook.github.io/create-react-app/docs/getting-started).

To learn React, check out the [React documentation](https://reactjs.org/).

### Code Splitting

This section has moved here: [https://facebook.github.io/create-react-app/docs/code-splitting](https://facebook.github.io/create-react-app/docs/code-splitting)

### Analyzing the Bundle Size

This section has moved here: [https://facebook.github.io/create-react-app/docs/analyzing-the-bundle-size](https://facebook.github.io/create-react-app/docs/analyzing-the-bundle-size)

### Making a Progressive Web App

This section has moved here: [https://facebook.github.io/create-react-app/docs/making-a-progressive-web-app](https://facebook.github.io/create-react-app/docs/making-a-progressive-web-app)

### Advanced Configuration

This section has moved here: [https://facebook.github.io/create-react-app/docs/advanced-configuration](https://facebook.github.io/create-react-app/docs/advanced-configuration)

### Deployment

This section has moved here: [https://facebook.github.io/create-react-app/docs/deployment](https://facebook.github.io/create-react-app/docs/deployment)



---

## Crear el Dockerfile


En la raíz del proyecto (junto a `package.json`) se creó el archivo `Dockerfile` con un build en dos etapas:

1. **Build** con Node.js: instala dependencias y genera la carpeta `build`.
2. **Serve** con nginx: sirve los archivos estáticos en el puerto 80.

### Contenido del `Dockerfile`

```dockerfile
# Etapa 1: construir la app React
FROM node:18-alpine AS build
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

# Etapa 2: servir los archivos con nginx
FROM nginx:alpine
COPY --from=build /app/build /usr/share/nginx/html
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

---

##  Crear el .dockerignore

### Qué se hizo

Se creó `.dockerignore` para no copiar archivos innecesarios a la imagen (más rápido y limpio).

### Contenido

```text
node_modules
build
.git
*.md
```


## Configurar secretos en GitHub

### Qué se hizo

En el repositorio de GitHub:

1. **Settings** → **Secrets and variables** → **Actions**
2. **New repository secret**
3. Se crearon estos dos secretos:

| Nombre del secreto     | Valor                                      |
|------------------------|--------------------------------------------|
| `DOCKERHUB_USERNAME`   | Usuario de Docker Hub (no el email)        |
| `DOCKERHUB_TOKEN`      | Access Token creado en el paso anterior    |

![](https://cdn.phototourl.com/free/2026-09-07-ffb1e7d3-3372-44b9-8e3a-2f6d61b5c8c0.png)


---

## Crear la GitHub Action

### Qué se hizo

Se creó el archivo:

```text
.github/workflows/docker-publish.yml
```

### Contenido del workflow

```yaml
name: Build and Push Docker Image

on:
  push:
    branches: [main]
  workflow_dispatch:

jobs:
  docker:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3

      - name: Login to Docker Hub
        uses: docker/login-action@v3
        with:
          username: ${{ secrets.DOCKERHUB_USERNAME }}
          password: ${{ secrets.DOCKERHUB_TOKEN }}

      - name: Build and push
        uses: docker/build-push-action@v5
        with:
          context: .
          push: true
          tags: ${{ secrets.DOCKERHUB_USERNAME }}/rick-morty:latest
```

### Pasos del workflow 

| Paso | Acción | Qué hace |
|------|--------|----------|
| 1 | Checkout | Descarga el código del repositorio |
| 2 | Set up Docker Buildx | Prepara Docker para construir imágenes |
| 3 | Login to Docker Hub | Inicia sesión con los secretos de GitHub |
| 4 | Build and push | Construye la imagen y la sube a Docker Hub |

### Nota sobre indentación YAML

`name`, `on` y `jobs` deben ir al **mismo nivel** (sin espacios extra al inicio). Si `on` o `jobs` quedan indentados debajo de `name`, GitHub marca error de sintaxis en la línea 2.

---

##  Verificar el workflow en Actions

### Qué se hizo

1. Entrar al repositorio en GitHub
2. Pestaña **Actions**
3. Abrir el workflow **Build and Push Docker Image**
4. Confirmar que todos los pasos quedaron en verde

![](https://cdn.phototourl.com/free/2026-09-07-54d3827e-3563-4b24-9f09-a1f67405542d.png)

![](https://cdn.phototourl.com/free/2026-09-07-e04cc5fd-7344-46f3-9728-b3d63ccf2d8a.png)
---

##  Verificar la imagen en Docker Hub

### Qué se hizo

1. Entrar a [https://hub.docker.com](https://hub.docker.com)
2. Ir a **Repositories**
3. Confirmar el repositorio `rick-morty` con el tag `latest`

Imagen publicada (ejemplo):

```text
vikyria/rick-morty:latest
```

![](https://cdn.phototourl.com/free/2026-09-07-f1aa2998-b3af-4e78-bda0-af025699739b.png)





## Resumen de archivos creados

| Archivo | Propósito |
|---------|-----------|
| `Dockerfile` | Define cómo construir la imagen |
| `.dockerignore` | Excluye archivos del contexto de build |
| `.github/workflows/docker-publish.yml` | Automatiza build + push a Docker Hub |

## Checklist final

- Dockerfile creado
- .dockerignore creado
- Access Token en Docker Hub
- Secretos `DOCKERHUB_USERNAME` y `DOCKERHUB_TOKEN` en GitHub
- Workflow `docker-publish.yml` creado
- Push a `main` realizado
- Actions en verde
- Imagen visible en Docker Hub
- Contenedor corriendo y app visible en el navegador

---


### `npm run build` fails to minify

This section has moved here: [https://facebook.github.io/create-react-app/docs/troubleshooting#npm-run-build-fails-to-minify](https://facebook.github.io/create-react-app/docs/troubleshooting#npm-run-build-fails-to-minify)
