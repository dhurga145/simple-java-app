# 🚀 Jenkins – Docker – Kubernetes (CI/CD Demo - Java Application)

---

## 📌 Project Overview

This project demonstrates a **complete CI/CD pipeline** using:

* Jenkins (Automation)
* Docker (Containerization)
* Kubernetes (Deployment)

A simple Java application is built, containerized, pushed to Docker Hub, and deployed on Kubernetes.

---

## 🛠️ Tech Stack

* Java (JDK 17)
* Maven
* Jenkins
* Docker
* Kubernetes

---

## ⚙️ Step 1: Install Docker Desktop

* Download and install Docker Desktop
* Ensure Docker is running

---

## ☸️ Step 2: Enable Kubernetes in Docker Desktop

1. Open Docker Desktop
2. Go to **Settings (⚙️)**
3. Select **Kubernetes Tab**
4. Enable **"Enable Kubernetes"**
5. Click **Apply & Restart**

⏳ It may take a few minutes to initialize the cluster.

---

## ⚙️ Step 3: Install Jenkins Plugins

Go to:

```
Manage Jenkins → Manage Plugins
```

Install:

* Docker Plugin
* Docker Pipeline Plugin

---

## 💻 Step 4: Create Simple Java Application

### 📄 App.java

```java
public class App {
    public static void main(String[] args) throws Exception {
        System.out.println("Java CI/CD Application Started...");
        while (true) {
            Thread.sleep(5000);
            System.out.println("Hello from Java CI/CD Pipeline!");
        }
    }
}
```

---

### 📄 pom.xml

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
         http://www.w3.org/xsd/maven-4.0.0.xsd">

    <modelVersion>4.0.0</modelVersion>
    <groupId>com.demo</groupId>
    <artifactId>simple-java-app</artifactId>
    <version>1.0</version>

    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.10.1</version>
                <configuration>
                    <source>17</source>
                    <target>17</target>
                </configuration>
            </plugin>
        </plugins>
    </build>

</project>
```

---

## 🧪 Step 5: Build Locally

```bash
mvn clean package
```

📦 Output:

```
target/simple-java-app-1.0.jar
```

---

## 🐳 Step 6: Create Dockerfile

```dockerfile
FROM eclipse-temurin:17-jdk

WORKDIR /app

COPY target/simple-java-app-1.0.jar app.jar

CMD ["java", "-jar", "app.jar"]
```

---

## 📤 Step 7: Push Code to GitHub

```bash
git init
git add .
git commit -m "Initial Java CI/CD commit"
git branch -M main
git remote add origin https://github.com/your-reponame/simple-java-app.git
git push -u origin main
```

---

## ☸️ Step 8: Kubernetes Deployment File

### 📄 deployment.yaml

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: java-app
spec:
  replicas: 2
  selector:
    matchLabels:
      app: java-app
  template:
    metadata:
      labels:
        app: java-app
    spec:
      containers:
        - name: java-app
          image: <your-dockerhub-username>/java-app:latest
---
apiVersion: v1
kind: Service
metadata:
  name: java-service
spec:
  type: NodePort
  selector:
    app: java-app
  ports:
    - port: 80
      targetPort: 8080
      nodePort: 30008
```

Push it:

```bash
git add deployment.yaml
git commit -m "Added Kubernetes deployment file"
git push
```

---

## ⚙️ Step 9: Configure Jenkins

* Start Jenkins
* Install **Kubernetes CLI Plugin**

---

## 🔐 Step 10 – Add Credentials in Jenkins (DockerHub + Kubernetes)

### 📍 Go to:

Manage Jenkins → Manage Credentials → (System) → Global credentials (unrestricted) → Add Credentials

---

## ➤ 1. Add DockerHub Credentials

| Field       | Value                   |
| ----------- | ----------------------- |
| Kind        | Username with password  |
| Scope       | Global                  |
| Username    | your DockerHub username | dhurgajai
| Password    | your DockerHub password | Balaji.2609
| ID          | dockerhub-creds         |
| Description | DockerHub Login         |

---

## ➤ 2. Add Kubernetes Config File (.kubeconfig)

### 📂 Where to get `.kubeconfig`?

If using Docker Desktop:

📍 Location (Windows):
C:\Users<your-username>.kube\config

👉 Example:
C:\Users\djg13.kube\config

✔ This file is automatically created when Kubernetes is enabled in Docker Desktop.

---

### ⚙️ Steps to Upload

1. Click **Add Credentials**
2. Fill the following:

| Field       | Value                                              |
| ----------- | -------------------------------------------------- |
| Kind        | Secret file                                        |
| File        | Upload the `config` file (or rename to kubeconfig) |
| ID          | kuberconfig                                        |
| Description | Kubernetes Config                                  |

---

### 🔍 Verify Kubernetes Config (Optional but Recommended)

Run:

```bash
kubectl config view
```

✔ If it shows output → configuration is correct

---

### ⚠️ Important

* Without this file → Kubernetes deployment will FAIL
* This file is used by Jenkins to connect to the cluster

---

## ⚙️ Step 11 – Create Jenkins Pipeline Job

### 📍 Steps

1. Go to Jenkins Dashboard
2. Click **New Item**
3. Enter name: `Java-CICD`
4. Select **Pipeline**
5. Click **OK**

---

## 🧾 Pipeline Script

```groovy
pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "yourdockerhubusername/java-app:latest"
    }

    stages {

        stage('Clone Repository') {
            steps {
                git branch: 'main',
                url: 'https://github.com/your-reponame/simple-java-app.git'
            }
        }

        stage('Build with Maven') {
            steps {
                bat 'mvn clean package'
            }
        }

        stage('Build Docker Image') {
            steps {
                bat "docker build -t %DOCKER_IMAGE% ."
            }
        }

        stage('Push to DockerHub') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    bat """
                    docker login -u %DOCKER_USER% -p %DOCKER_PASS%
                    docker push %DOCKER_IMAGE%
                    """
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                withCredentials([file(
                    credentialsId: 'kuberconfig',
                    variable: 'KUBECONFIG'
                )]) {
                    bat '''
                    set KUBECONFIG=%KUBECONFIG%
                    kubectl apply -f deployment.yaml
                    '''
                }
            }
        }
    }

    post {
        success {
            echo 'CI/CD Pipeline executed successfully!'
        }
        failure {
            echo 'Pipeline failed. Please check logs.'
        }
    }
}
```

---

## ▶️ Run Pipeline

Click:
Save → Build Now

---



## 🔍 Step 12: Verify Deployment

### Check Kubernetes Services

```bash
kubectl get svc
```

---

## 📊 CI/CD Flow Explanation

### 🔹 Continuous Integration (CI)

* Developer pushes Java code
* Jenkins builds using Maven
* Docker image is created

### 🔹 Continuous Deployment (CD)

* Image pushed to Docker Hub
* Kubernetes deploys automatically
* Pods run inside cluster

---

## 🔄 Workflow Summary

* Jenkins clones repository
* Maven compiles Java
* Docker builds container
* Docker Hub stores image
* Kubernetes pulls image
* Pods are created and scaled

---

## 🎯 Key Features

* Fully automated CI/CD pipeline
* Scalable Kubernetes deployment (2 replicas)
* Real-time Java application execution

---

## 👨‍💻 Author

Your Name
