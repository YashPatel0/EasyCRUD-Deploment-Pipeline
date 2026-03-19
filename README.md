# EasyCRUD Full Stack Deployment on AWS (EKS + RDS + S3)

## Project Overview

This project demonstrates deployment of a **full-stack CRUD application** on AWS using modern DevOps tools.

* **Frontend**: Vite-based web application hosted on **Amazon S3**
* **Backend**: Spring Boot REST API deployed on **Amazon EKS**
* **Database**: **Amazon RDS (MariaDB/MySQL)**
* **Containerization**: Docker
* **Orchestration**: Kubernetes (EKS)
* **CI/CD**: Jenkins
* **Code Testing**: SonarQube

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
* Jenkins
* SonarQube

---

# Step 1: Create Infrastructure

Create the following AWS resources:

* Amazon **EKS Cluster**
* Amazon **RDS Database**
* Amazon **S3 Bucket**

Example S3 bucket name used in this project:

```
my-demo-buxxx
```

---

# step 2: Launch SonarQube Server

* Enable port No- 9000
* take ssh

 ### Step 2.1 :- Update the instance:
 ```
 apt update
 ```

  ### Step 2.2 :- Install SonarQube :
  ```
apt install openjdk-17-jdk -y
apt install postgresql -y
systemctl start postgresql
sudo -u postgres psql
```
```
>> CREATE USER linux PASSWORD 'redhat';
>> CREATE DATABASE sonarqube;
>> GRANT ALL PRIVILEGES ON DATABASE sonarqube TO linux;
>> \c sonarqube;
>> GRANT ALL PRIVILEGES ON SCHEMA public TO linux;
>> \q
  ```

  ### Step 2.3 :- Configure Linux Machine:
```
sysctl -w vm.max_map_count=524288
sysctl -w fs.file-max=131072
ulimit -n 131072
ulimit -u 8192
```

### Step 2.4 :- Install and Configure Sonarqube
```
wget https://binaries.sonarsource.com/Distribution/sonarqube/sonarqube-25.5.0.107428.zip
apt install unzip -y
unzip sonarqube-25.5.0.107428.zip
mv sonarqube-25.5.0.107428 /opt/sonar
cd /opt/sonar
vim conf/sonar.properties
>> sonar.jdbc.username=linux
>> sonar.jdbc.password=redhat
>> sonar.jdbc.url=jdbc:postgresql://localhost/sonarqube
useradd sonar -m
chown sonar:sonar -R /opt/sonar
su sonar
cd /opt/sonar/bin/linux-x86-64
./sonar.sh start
./sonar.sh status 
```  

### Step 2.5 :- Take Access Of Sonarqube
```
http://< public IP of sonarqube instance >:9000
```

### Step 2.6 :- Login to Sonarqube

* Initial Username and Password is
  * Username:- admin
  * password:- admin
* Change The Password according to requirment

### Step 2.7 :- Folow the steps

* Go to _ Create a local project _
* Give the _ project name _ and click on Next
* Click on _ Use the global setting _ and click on create Project
* click on _ Locally _
* generate Token name (save the token in local device ) and click on continue
* Click on _ MAVEN _ (Save Gunerated MAVEN script in local device)
 
---
# Step 3: Launch Jenkins Instance 

* Enable Port 8080
* Make sure to fulfill Jenkins requirement (Use instance c7i-flex.large)
* take ssh

### Step 3.1 :- Update the instance
```
apt update
```

### Step 3.2 :- Install Jenkins
```
apt install openjdk-17-jdk -y

sudo wget -O /etc/apt/keyrings/jenkins-keyring.asc \
  https://pkg.jenkins.io/debian-stable/jenkins.io-2026.key
echo "deb [signed-by=/etc/apt/keyrings/jenkins-keyring.asc]" \
  https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null

sudo apt update

sudo apt install jenkins

systemctl start jenkins

systemctl status jebkins

```
### Step 3.3 :- Take access of jenkins
```
http://< Jenkins instance public IP >:8080
``` 
### Step 3.4 :- get jenkins initial password inside jenkins instance
```
cat /var/lib/jenkins/secrets/initialAdminPassword
```

### Step 3.5 :- Folow the steps

*  Click on _ select plugins to install _ and clock on next
* Fill the creadential like username password and click on _ save nad Continue _ then click on ok and then click on _ start using jenkins_
* Go to settings > Plugins > install all required plugins
  * pipeline
  * pipeline stage view
  * git
  * github
  * sonarqube

### Step 3.6 :- Configure sonarqube
  * Go to settings
  * Click on _ System _
  * Scroll down to _ SonarQube servers _
  * Click on _ Add SonarQube _
  * Fill the Credentials
  * Click on _ Add > Secret Text _
  * Give the name and pest token which was generated while creating sonarqube and click on create
  * Select the secret text and click on save and apply
---

# Step 4: Go to Jenkins Instance And Install
* Docker
* AWS-CLI  
* kubectl
* maven
* Login to Docker-Hub

```
## Install Docker
apt install docker.io -y
docker -v
# Add jenkins to Docker group 
usermod -aG docker jenkins
```
```
## Install AWS-CLI
# Generate _ Access Key _ and _ Secret Key _ Of AWS account Or IAM role

curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
apt install unzip -y
unzip awscliv2.zip
sudo ./aws/install
aws configure

# AWS Access Key ID = < Pest Access Key ID >
# AWS Secret Access Key = < Pest Secret Access Key >
# Default region name = < Give region name >
# Default output format = json

# Check Whether .aws File is created or not
la -a

# Give AWS-CLI Permission to jenkins user
cp -rv .aws/ /var/lib/jenkins/
chown -R jenkins /var/lib/jenkins/.aws/
```
```
## Install kubectl

# Enable HTTP and HTTPS in security-group

curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"

sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl

kubectl version --client

# Take a Access Of EKS cluster
aws eks update-kubeconfig --name < cluster name > --region < region >

# check whether nodes are visible or not
kubectl get nodes

# Check whether .kube file is created or not
ls -a
cp -rv .kube/ /var/lib/jenkins/
chown -R jenkins /var/lib/jenkins/.kube/
```
```
## Install maven
apt install maven
```
```
## Login Docker-Hub inside jenkins user
docker login
# give all Credentials and login to docker hub and login
```
All the prerequisite of Jenkins is completed

---

# Start Implementing Project

### Step 5: Take access of RDS database

After creating the RDS instance, create a database.

Example database name:

```
studentapp_db
```

 Do changes in backend application.properties
```
server.port=8080

spring.datasource.url=jdbc:mariadb://<rds_endpoint>:3306/<database-name>?sslMode=required
spring.datasource.username=user
spring.datasource.password=pass

spring.jpa.hibernate.ddl-auto=update
spring.jpa.show-sql=true
spring.jpa.database-platform=org.hibernate.dialect.MariaDBDialect
```
### Step 6: Create Bakcend Pipeline
```
pipeline {
    agent any

    parameters {
        string(name: 'BACKEND_URL', defaultValue: '', description: 'Backend URL')
    }

    stages {

        stage('PULL'){
            steps{
                git branch: 'main', url: 'https://github.com/YashPatel0/EasyCRUD-Deploment-Pipeline.git'
            }
        }

        stage('SET ENV'){
            steps{
                sh """
                echo 'VITE_API_URL="http://${params.BACKEND_URL}:8080/api"' > frontend/.env
                """
            }
        }

        stage('BUILD FRONTEND'){
            steps{
                sh '''
                    cd frontend
                    npm install
                    npm run build
                '''
            }
        }  


        stage('Move to S3'){
            steps{
                sh '''
                    cd frontend
                    aws s3 sync dist/ s3://my-demo-buxxx/ --delete
                '''
            }
        }  

    }
}

``` 
* Update the Deployment.yaml file
* Add image name inside Deployment.yaml

### Step 7: Create Frontend Pipeline
```
pipeline {
    agent any

    parameters {
        string(name: 'BACKEND_URL', defaultValue: '', description: 'Backend URL')
    }

    stages {

        stage('PULL'){
            steps{
                git branch: 'main', url: 'https://github.com/YashPatel0/EasyCRUD-Deploment-Pipeline.git'
            }
        }

        stage('SET ENV'){
            steps{
                sh """
                echo 'VITE_API_URL="http://${params.BACKEND_URL}:8080/api"' > frontend/.env
                """
            }
        }

        stage('BUILD FRONTEND'){
            steps{
                sh '''
                    cd frontend
                    npm install
                    npm run build
                '''
            }
        }  


        stage('Move to S3'){
            steps{
                sh '''
                    cd frontend
                    aws s3 sync dist/ s3://my-demo-buxxx/ --delete
                '''
            }
        }  

    }
}

```
* go to jenkins dashboard 
  * Build Backend Pipeline [ It will automatically build the Frontend Pipeline also because i use build with parameter concept so after sucessfully build backend pipeline ,frontend will automatically build ]

---

### Step 8: Enable S3 Static Website Hosting

Go to:

S3 → my-demo-buxxx → Properties

Enable Static Website Hosting

Configuration:

```
Index document: index.html
Error document: index.html
```

### Allow Public Access to Bucket

Add the following bucket policy:
```
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::my-demo-buxxx/*"
    }
  ]
}
```
---
## Go to S3 → my-demo-buxxx → Properties → Static Website Hosting [ get Endpoint URL ]