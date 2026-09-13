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


### `npm run build` fails to minify

This section has moved here: [https://facebook.github.io/create-react-app/docs/troubleshooting#npm-run-build-fails-to-minify](https://facebook.github.io/create-react-app/docs/troubleshooting#npm-run-build-fails-to-minify)

---

## Create the Dockerfile


In the project, the `Dockerfile` file was created with a two-stage build:

1. **Build** with Node.js: installs dependencies and generates the `build` folder.
2. **Serve** with nginx: serves the static files on port 80.

### Contents of `Dockerfile`

```dockerfile
# Stage 1: build the React app
FROM node:18-alpine AS build
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

# Stage 2: serve the files with nginx
FROM nginx:alpine
COPY --from=build /app/build /usr/share/nginx/html
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

---

##  Create the .dockerignore

### What was done

`.dockerignore` was created to avoid copying unnecessary files into the image (faster and cleaner).

### Contents

```text
node_modules
build
.git
*.md
```


## Configure secrets in GitHub

### What was done

In the GitHub repository:

1. **Settings** → **Secrets and variables** → **Actions**
2. **New repository secret**
3. These two secrets were created:

| Secret name     | Value                                      |
|------------------------|--------------------------------------------|
| `DOCKERHUB_USERNAME`   | Docker Hub username (not the email)        |
| `DOCKERHUB_TOKEN`      | Access Token created  |

![](https://cdn.phototourl.com/free/2026-09-07-ffb1e7d3-3372-44b9-8e3a-2f6d61b5c8c0.png)


---

## Create the GitHub Action

### What was done

The following file was created:

```text
.github/workflows/docker-publish.yml
```

### Workflow contents

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

### Workflow steps

| Step | Action | What it does |
|------|--------|----------|
| 1 | Checkout | Downloads the repository code |
| 2 | Set up Docker Buildx | Prepares Docker to build images |
| 3 | Login to Docker Hub | Logs in using the GitHub secrets |
| 4 | Build and push | Builds the image and pushes it to Docker Hub |

### Note on YAML indentation

`name`, `on`, and `jobs` must be at the **same level** (no extra leading spaces). If `on` or `jobs` end up indented under `name`, GitHub flags a syntax error on line 2.

---

##  Verify the workflow in Actions

### What was done

1. Go to the repository on GitHub
2. **Actions** tab
3. Open the **Build and Push Docker Image** workflow
4. Confirm that all steps show green

![](https://cdn.phototourl.com/free/2026-09-07-54d3827e-3563-4b24-9f09-a1f67405542d.png)

![](https://cdn.phototourl.com/free/2026-09-07-e04cc5fd-7344-46f3-9728-b3d63ccf2d8a.png)
---

##  Verify the image on Docker Hub

### What was done

1. Go to [https://hub.docker.com](https://hub.docker.com)
2. Go to **Repositories**
3. Confirm the `rick-morty` repository with the `latest` tag

Published image (example):

```text
vikyria/rick-morty:latest
```

![](https://cdn.phototourl.com/free/2026-09-07-f1aa2998-b3af-4e78-bda0-af025699739b.png)





## Summary of files created

| File | Purpose |
|---------|-----------|
| `Dockerfile` | Defines how to build the image |
| `.dockerignore` | Excludes files from the build context |
| `.github/workflows/docker-publish.yml` | Automates build + push to Docker Hub |

## Final checklist

- Dockerfile created
- .dockerignore created
- Access Token on Docker Hub
- `DOCKERHUB_USERNAME` and `DOCKERHUB_TOKEN` secrets in GitHub
- `docker-publish.yml` workflow created
- Push to `main` done
- Actions green
- Image visible on Docker Hub
- Container running and app visible in the browser

---
