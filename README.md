# EasyCRUD Full Stack Deployment on AWS (EKS + RDS + S3)

## Project Overview

This project demonstrates deployment of a **full-stack CRUD application** on AWS using modern DevOps tools.

* **Frontend**: Vite-based web application hosted on **Amazon S3**
* **Backend**: Spring Boot REST API deployed on **Amazon EKS**
* **Database**: **Amazon RDS (MariaDB/MySQL)**
* **Containerization**: Docker
* **Orchestration**: Kubernetes (EKS)

The goal of this project is to showcase a **real-world DevOps deployment workflow** using AWS cloud services.

---

# Architecture

User → S3 Static Website → AWS LoadBalancer → EKS Backend Pods → Amazon RDS Database

---

# Technologies Used

* AWS EKS (Kubernetes)
* AWS RDS
* AWS S3 Static Website Hosting
* Docker
* Kubernetes
* Spring Boot
* Vite (Frontend)
* DockerHub
* AWS CloudShell

---

# Step 1: Create Infrastructure

Create the following AWS resources:

* Amazon **EKS Cluster**
* Amazon **RDS Database**
* Amazon **S3 Bucket**

Example S3 bucket name used in this project:

```
yash-buxx
```

---

# Step 2: Create Database in RDS

After creating the RDS instance, create a database.

Example database name:

```
studentapp_db
```

---

# Step 3: Clone the Application

Clone the project repository:

```bash
git clone <repository-url>
cd project-folder
```

---

# Step 4: Configure Backend Database Connection

Update the **application.properties** file.

```
server.port=8080

spring.datasource.url=jdbc:mariadb://<rds_endpoint>:3306/<database-name>?sslMode=required
spring.datasource.username=user
spring.datasource.password=pass

spring.jpa.hibernate.ddl-auto=update
spring.jpa.show-sql=true
spring.jpa.database-platform=org.hibernate.dialect.MariaDBDialect
```

---

# Step 5: Build Backend Docker Image

Build and push the backend image to DockerHub.

```bash
docker build -t easycrud_backend .
docker tag easycrud_backend <dockerhub-username>/easycrud_backend:v1
docker push <dockerhub-username>/easycrud_backend:v1
```

---

# Step 6: Deploy Backend to Kubernetes (EKS)

Create a **Kubernetes manifest file** for backend deployment and service.

Use **LoadBalancer service** because the backend must be accessible **outside the Kubernetes cluster**.

Example deployment:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: studentapp
spec:
  replicas: 1
  selector:
    matchLabels:
      app: studentapp
  template:
    metadata:
      labels:
        app: studentapp
    spec:
      containers:
      - name: studentapp
        image: <dockerhub-username>/easycrud_backend:v1
        ports:
        - containerPort: 8080
```

Example service:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: studentapp-svc
spec:
  type: LoadBalancer
  selector:
    app: studentapp
  ports:
  - port: 8080
    targetPort: 8080
```

Apply manifest:

```bash
kubectl apply -f backend-manifest.yaml
```

Verify backend pod:

```bash
kubectl get pods
kubectl get svc
```

Copy the **LoadBalancer DNS or IP** for frontend configuration.

---

# Step 7: Configure Frontend

Inside the **frontend folder**, update the `.env` file.

```
VITE_API_URL="http://<load-balancer-ip>:8080/api"
```

---

# Step 8: Build Frontend Docker Image

```bash
docker build -t easycrud_frontend .
docker tag easycrud_frontend <dockerhub-username>/easycrud_frontend:v1
docker push <dockerhub-username>/easycrud_frontend:v1
```

---

# Step 9: Deploy Frontend in Kubernetes

Create a Kubernetes manifest for frontend deployment.

```bash
kubectl apply -f frontend-manifest.yaml
```

Verify:

```bash
kubectl get pods
```

---

# Step 10: Build Static Files for S3

Inside the frontend folder run:

```bash
npm install
npm run build
```

This creates a **dist** folder.

Example structure:

```
dist/
 ├── index.html
 ├── vite.svg
 └── assets/
```

---

# Step 11: Upload Frontend to S3

Navigate to the dist folder:

```bash
cd dist
ls
```

Expected output:

```
assets  index.html  vite.svg
```

Upload files to S3:

```bash
aws s3 cp . s3://yash-buxx/ --recursive
```

---

# Step 12: Enable S3 Static Website Hosting

Go to:

```
S3 → yash-buxx → Properties
```

Enable **Static Website Hosting**

Configuration:

```
Index document: index.html
Error document: index.html
```

---

# Step 13: Allow Public Access to Bucket

Add the following **bucket policy**:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::yash-buxx/*"
    }
  ]
}
```

---

# Final Result

Frontend hosted on **Amazon S3**

Backend running on **Amazon EKS**

Database running on **Amazon RDS**

Architecture:

User → S3 Static Website → AWS LoadBalancer → EKS Backend → RDS Database

---

# Author

Yash Patel
DevOps / Cloud Engineer
