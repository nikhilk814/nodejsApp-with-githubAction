**CI/CD Pipeline with GitHub Actions and Docker**
Absolutely! I can help you format your detailed CI/CD workflow explanation into a **clean, GitHub README-ready format**. The idea is to make it visually structured, easy to follow, and include placeholders for screenshots. Here’s how it can look:

---

# CI/CD Pipeline with GitHub Actions and Docker

This repository demonstrates a **complete CI/CD workflow** using a self-hosted GitHub Actions runner. It builds, tests, pushes a Docker image to Docker Hub, and deploys the container on the same machine.

---

## Table of Contents

1. [Workflow Overview](#workflow-overview)
2. [Prerequisites](#prerequisites)
3. [Folder Structure](#folder-structure)
4. [Workflow YAML](#workflow-yaml)
5. [Step-by-Step Setup](#step-by-step-setup)
6. [Verification](#verification)
7. [Notes & Tips](#notes--tips)

---

## Workflow Overview

The workflow performs the following tasks:

* Triggered on push, pull request, or manual run (`workflow_dispatch`).
* Uses a **self-hosted runner** for all jobs.
* Installs Node.js dependencies and runs tests.
* Builds and pushes a Docker image to Docker Hub.
* Stops any running container and deploys the new image on port `8000`.

---

## Prerequisites

* A **self-hosted runner** registered and running on the deployment machine.
* Docker installed and the runner user added to the `docker` group.
* GitHub repository secrets:

  * `DOCKER_USERNAME`
  * `DOCKER_PASSWORD`
* A Dockerfile in the project root that exposes port `8000` and runs the app.

---

## Folder Structure

```
.github/
 └── workflows/
     └── manual.yml   # GitHub Actions workflow YAML
Dockerfile             # Docker image definition
package.json           # Node.js project dependencies
app.js                 # Node.js application entry
```

---

## Workflow YAML

Create a file `.github/workflows/manual.yml` with the following content:

```yaml
name: CI/CD Pipeline

on:
  push:
    branches:
      - main
  pull_request:
    branches:
      - main
  workflow_dispatch:

jobs:
  build:
    runs-on: self-hosted
    steps:
      - name: Checkout code
        uses: actions/checkout@v3
      - name: Set up Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '14'
      - name: Install dependencies
        run: npm install
      - name: Run tests
        run: npm test

  docker:
    runs-on: self-hosted
    needs: build
    steps:
      - name: Checkout code
        uses: actions/checkout@v3
      - name: Log in to Docker Hub
        uses: docker/login-action@v3
        with:
          username: ${{ secrets.DOCKER_USERNAME }}
          password: ${{ secrets.DOCKER_PASSWORD }}
      - name: Build Docker image
        run: docker build -t ${{ secrets.DOCKER_USERNAME }}/todo-webapp:latest .
      - name: Push Docker image
        run: docker push ${{ secrets.DOCKER_USERNAME }}/todo-webapp:latest

  deploy:
    runs-on: self-hosted
    needs: docker
    steps:
      - name: Deploy Docker container
        run: |
          docker pull ${{ secrets.DOCKER_USERNAME }}/todo-webapp:latest
          docker stop todo-webapp || true
          docker rm todo-webapp || true
          docker run -d --restart=unless-stopped --name todo-webapp -p 8000:8000 ${{ secrets.DOCKER_USERNAME }}/todo-webapp:latest
```

---

## Step-by-Step Setup

### 1. Set up Self-Hosted Runner

* Install Docker on your Linux VM and add the runner user to the `docker` group.
* Register and start the GitHub Actions runner:

  ```bash
  ./run.sh
  ```
* **Screenshot placeholder:** Runner connected and running jobs.

---

### 2. Add Dockerfile

```dockerfile
FROM node:14-alpine
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
EXPOSE 8000
CMD ["node","app.js"]
```

* **Screenshot placeholder:** Dockerfile in GitHub UI.
<img width="1892" height="816" alt="Image" src="https://github.com/user-attachments/assets/2c3e1d87-3350-4a4a-a4dc-bb737d79a300" />
---

### 3. Create Workflow

* Save the YAML to `.github/workflows/manual.yml` on the `main` branch.
* **Screenshot placeholder:** Actions run summary with all jobs green.

---

### 4. Configure Secrets

* In repository **Settings → Secrets and variables → Actions**, add:

  * `DOCKER_USERNAME`
  * `DOCKER_PASSWORD`
* **Screenshot placeholder:** Docker login success in workflow log.

---

### 5. Trigger the Pipeline

* Push to `main`, create a PR to `main`, or trigger manually via the **Actions tab**.
* Jobs sequence: **build → docker → deploy**.
<img width="1920" height="1020" alt="Image" src="https://github.com/user-attachments/assets/bd265bfa-487e-409e-979a-abac90a2b325" />
---

### 6. Verify Deployment

* On the server, check:

```bash
docker images
docker ps
```

* Container should run as `todo-webapp` on port `8000`.
* Open `http://SERVER_IP:8000/todo` in a browser.
* **Screenshot placeholders:**
<img width="1889" height="416" alt="Image" src="https://github.com/user-attachments/assets/b41f22bd-46ef-44c3-8520-790df29e0494" />

  * `docker images` & `docker ps` output
  * Todo app page loaded

---

<img width="1916" height="951" alt="Image" src="https://github.com/user-attachments/assets/4510548d-0129-4f95-a873-e84daba170f2" />

---
