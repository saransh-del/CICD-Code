
# Node.js CI/CD Pipeline with Docker and Kubernetes

This project demonstrates a CI/CD pipeline for a Node.js application using **GitHub Actions**, **Docker**, and **Kubernetes**.

## **Pipeline Overview**
1. Automatically runs tests on pull requests.
2. Builds a Docker image of the Node.js application.
3. Pushes the Docker image to Docker Hub.
4. Deploys the Docker image to a Kubernetes cluster.
5. Sends notifications for deployment success or failure.

---

## **Prerequisites**
1. A Node.js project initialized with a `package.json` file.
2. A GitHub repository.
3. Docker installed and a Docker Hub account.
4. Access to a Kubernetes cluster.
5. Configured GitHub Secrets for sensitive credentials.

---

## **Setting Up Your CI/CD Pipeline**

### **1. Docker Hub Credentials**
You need to store your Docker Hub username and password as GitHub Secrets:
- `DOCKER_USERNAME`: Your Docker Hub username.
- `DOCKER_PASSWORD`: Your Docker Hub password.

### **2. Kubernetes Configuration**
You need to upload your Kubernetes configuration file (`~/.kube/config`) as a GitHub Secret:
1. Base64-encode your Kubernetes configuration file:
   ```bash
   cat ~/.kube/config | base64
   ```
2. Copy the output and create a GitHub Secret:
   - Name: `KUBECONFIG`
   - Value: The base64-encoded string of your Kubernetes configuration.

### **3. Update Your Workflow File**
Create a file at `.github/workflows/cicd.yml` in your repository with the following pipeline configuration:

```yaml
name: CI/CD Pipeline

on:
  push:
    branches:
      - main
  pull_request:

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v3

      - name: Set up Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '16'

      - name: Install dependencies
        run: npm install

      - name: Run tests
        run: npm test

      - name: Build Docker image
        run: |
          docker build -t ${{ secrets.DOCKER_USERNAME }}/nodejs-app:latest .
          echo "${{ secrets.DOCKER_PASSWORD }}" | docker login -u "${{ secrets.DOCKER_USERNAME }}" --password-stdin
          docker push ${{ secrets.DOCKER_USERNAME }}/nodejs-app:latest

      - name: Set up Kubernetes config
        run: |
          echo "${{ secrets.KUBECONFIG }}" | base64 -d > kubeconfig
          export KUBECONFIG=kubeconfig

      - name: Deploy to Kubernetes
        run: kubectl apply -f k8s/deployment.yml
```

---

## **File Descriptions**

### **1. Dockerfile**
Ensure your project contains a `Dockerfile` for building the image. Example:

```Dockerfile
FROM node:16

WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
EXPOSE 3000
CMD ["npm", "start"]
```

### **2. Kubernetes Deployment File**
Add a Kubernetes deployment configuration at `k8s/deployment.yml`:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nodejs-app
spec:
  replicas: 2
  selector:
    matchLabels:
      app: nodejs-app
  template:
    metadata:
      labels:
        app: nodejs-app
    spec:
      containers:
      - name: nodejs-app
        image: <your-docker-username>/nodejs-app:latest
        ports:
        - containerPort: 3000
```

Replace `<your-docker-username>` with your Docker Hub username.

---

## **Configuring GitHub Secrets**

1. Go to your GitHub repository → **Settings** → **Secrets and variables** → **Actions**.
2. Add the following secrets:
   - `DOCKER_USERNAME`: Your Docker Hub username.
   - `DOCKER_PASSWORD`: Your Docker Hub password.
   - `KUBECONFIG`: Base64-encoded Kubernetes configuration.

---

## **How to Run the Pipeline**
1. Push your code to the `main` branch or open a pull request.
2. GitHub Actions will automatically:
   - Run `npm install` and `npm test`.
   - Build and push the Docker image to Docker Hub.
   - Deploy the image to the Kubernetes cluster.

---

## **Troubleshooting**
- If the pipeline fails, check the **GitHub Actions logs** for detailed error messages.
- Verify that:
  - Docker Hub credentials (`DOCKER_USERNAME` and `DOCKER_PASSWORD`) are correct.
  - Kubernetes configuration (`KUBECONFIG`) is properly base64-encoded and valid.

---

