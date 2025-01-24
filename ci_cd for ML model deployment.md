# **Building a CI/CD Pipeline for Machine Learning Model Deployment**

In this blog, I’ll demonstrate how to build a robust CI/CD pipeline for deploying a machine learning (ML) model using GitHub, Jenkins, FastAPI, Docker, Kubernetes, and `.yaml` files for storing environment variables. The pipeline ensures seamless automation from code commit to deployment in a Kubernetes cluster.

---

### **Prerequisites**
- **GitHub:** Repository to host the code.
- **Jenkins:** CI/CD server to automate the pipeline.
- **FastAPI:** Python-based API framework for serving the ML model.
- **Docker:** For containerizing the application.
- **Kubernetes:** For orchestrating the deployment.
- **Environment Variables (.yaml):** Centralized configuration for easy management.

---

## **Step 1: Project Structure**

```
ml-deployment/
├── app/
│   ├── main.py  # FastAPI application
│   ├── model.py  # Model inference logic
│   ├── Dockerfile  # Docker instructions
├── configs/
│   ├── env.yaml  # Environment variables
│   ├── deployment.yaml  # Kubernetes deployment manifest
│   ├── service.yaml  # Kubernetes service manifest
├── tests/
│   ├── test_main.py  # Unit tests
├── Jenkinsfile  # Jenkins pipeline script
├── requirements.txt  # Python dependencies
├── .github/
│   ├── workflows/
│       ├── ci.yml  # GitHub Actions for CI
```

---

## **Step 2: Write the Code**

### **FastAPI Application**
`app/main.py`:
```python
from fastapi import FastAPI
from app.model import predict

app = FastAPI()

@app.get("/")
def health_check():
    return {"status": "ok"}

@app.post("/predict/")
def get_prediction(data: dict):
    return {"prediction": predict(data)}
```

`app/model.py`:
```python
def predict(data: dict) -> str:
    # Simulate a dummy prediction logic
    return "positive" if data.get("input") > 0 else "negative"
```

---

### **Dockerfile**
`app/Dockerfile`:
```dockerfile
# Use official Python image
FROM python:3.9-slim

WORKDIR /app

# Install dependencies
COPY requirements.txt .
RUN pip install -r requirements.txt

# Copy codebase
COPY . .

# Run the application
CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8000"]
```

---

### **Environment Variables**
`configs/env.yaml`:
```yaml
APP_NAME: "ML FastAPI App"
MODEL_VERSION: "v1.0"
```

---

### **Kubernetes Manifests**
`configs/deployment.yaml`:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: ml-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: ml-app
  template:
    metadata:
      labels:
        app: ml-app
    spec:
      containers:
      - name: ml-app
        image: <your-docker-image>:latest
        ports:
        - containerPort: 8000
        envFrom:
        - configMapRef:
            name: ml-config
```

`configs/service.yaml`:
```yaml
apiVersion: v1
kind: Service
metadata:
  name: ml-service
spec:
  selector:
    app: ml-app
  ports:
    - protocol: TCP
      port: 80
      targetPort: 8000
  type: LoadBalancer
```

---

## **Step 3: Set Up Jenkins Pipeline**

### **Install Jenkins Plugins**
- **Pipeline**
- **GitHub**
- **Docker**
- **Kubernetes**

### **Jenkinsfile**
`Jenkinsfile`:
```groovy
pipeline {
    agent any
    environment {
        DOCKER_IMAGE = 'your-docker-image:latest'
    }
    stages {
        stage('Clone Repository') {
            steps {
                git 'https://github.com/your-repo/ml-deployment.git'
            }
        }
        stage('Run Tests') {
            steps {
                sh 'pytest tests/'
            }
        }
        stage('Build Docker Image') {
            steps {
                sh 'docker build -t $DOCKER_IMAGE ./app'
            }
        }
        stage('Push Docker Image') {
            steps {
                withDockerRegistry([credentialsId: 'dockerhub-credentials']) {
                    sh 'docker push $DOCKER_IMAGE'
                }
            }
        }
        stage('Deploy to Kubernetes') {
            steps {
                sh 'kubectl apply -f configs/env.yaml'
                sh 'kubectl apply -f configs/deployment.yaml'
                sh 'kubectl apply -f configs/service.yaml'
            }
        }
    }
}
```

---

## **Step 4: Set Up GitHub Actions for CI**

### **GitHub Workflow**
`.github/workflows/ci.yml`:
```yaml
name: CI

on:
  push:
    branches:
      - main

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v3

      - name: Set up Python
        uses: actions/setup-python@v3
        with:
          python-version: '3.9'

      - name: Install dependencies
        run: |
          pip install -r requirements.txt

      - name: Run tests
        run: |
          pytest tests/
```

---

## **Step 5: Set Up Kubernetes ConfigMap**

### **Apply ConfigMap**
```bash
kubectl create configmap ml-config --from-file=configs/env.yaml
```

---

## **Step 6: Testing the Pipeline**

1. **Commit Code:** Push the code to GitHub.
2. **Trigger Jenkins:** Jenkins will:
   - Run tests
   - Build and push the Docker image
   - Deploy to Kubernetes
3. **Verify Deployment:**
   ```bash
   kubectl get pods
   kubectl get services
   ```

---

## **Conclusion**
This pipeline ensures smooth and automated deployment for ML models using GitHub, Jenkins, FastAPI, Docker, Kubernetes, and YAML-based environment management. By incorporating CI/CD principles, your team can improve productivity and reliability for ML projects.
