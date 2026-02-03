# Jenkins CI/CD Deployment to Amazon ECS (End-to-End)

## 1. Project Overview

This project demonstrates an end-to-end CI/CD pipeline using Jenkins to deploy a containerized application to Amazon ECS behind an Application Load Balancer. Source code is stored in GitHub, Docker images are built in Jenkins, pushed to Amazon ECR, and deployed to ECS with zero or minimal downtime.

---

## 2. Architecture Flow

1. Developer pushes code to GitHub
2. Jenkins pipeline is triggered
3. Jenkins builds Docker image
4. Jenkins pushes image to Amazon ECR
5. Jenkins updates ECS service
6. ECS deploys new task revision
7. ALB routes traffic to healthy ECS tasks

---

## 3. Prerequisites

### 3.1 AWS Resources

* VPC with public subnets
* Internet Gateway attached to VPC
* Application Load Balancer (ALB)
* Target Group (type: ip)
* ECS Cluster (Fargate)
* ECS Service
* ECS Task Definition
* Amazon ECR Repository

### 3.2 IAM Requirements

* Jenkins IAM user or role with:

  * AmazonECSFullAccess
  * AmazonEC2ContainerRegistryFullAccess
  * CloudWatchLogsFullAccess

### 3.3 Jenkins Server

* Jenkins installed on EC2
* Docker installed and running
* Required plugins:

  * Git
  * Pipeline
  * Docker Pipeline
  * AWS Credentials

---

## 4. Application Setup

### 4.1 Sample Application Structure

```
project-root/
├── app/
│   └── index.html
├── Dockerfile
└── Jenkinsfile
```

### 4.2 Sample index.html

```
Hello from Jenkins + ECS Deployment
```

---

## 5. Docker Configuration

### 5.1 Dockerfile

```
FROM nginx:alpine
COPY app/ /usr/share/nginx/html
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

Key Point:

* Application listens on 0.0.0.0:80

---

## 6. Amazon ECR Setup

### 6.1 Create ECR Repository

```
aws ecr create-repository --repository-name jenkins-ecs-app
```

### 6.2 Note Repository URI

```
<account-id>.dkr.ecr.<region>.amazonaws.com/jenkins-ecs-app
```

---

## 7. ECS Configuration

### 7.1 Task Definition

* Launch type: Fargate
* Network mode: awsvpc
* Container port: 80
* Protocol: TCP
* Log driver: awslogs

### 7.2 ECS Service

* Desired tasks: 2
* Minimum healthy percent: 100
* Maximum percent: 200
* Load balancer:

  * Listener: HTTP 80
  * Target group: tg-for-ecs

---

## 8. Security Group Configuration

### 8.1 ALB Security Group

Inbound:

* HTTP 80 from 0.0.0.0/0

Outbound:

* All traffic

### 8.2 ECS Task Security Group

Inbound:

* HTTP 80 from ALB Security Group

Outbound:

* All traffic

---

## 9. Jenkins Pipeline Configuration

### 9.1 Jenkinsfile

```
pipeline {
  agent any

  environment {
    AWS_REGION = "us-east-1"
    ECR_REPO = "<account-id>.dkr.ecr.us-east-1.amazonaws.com/jenkins-ecs-app"
    IMAGE_TAG = "${BUILD_NUMBER}"
  }

  stages {
    stage('Checkout') {
      steps {
        git branch: 'main', url: 'https://github.com/username/repo.git'
      }
    }

    stage('Build Image') {
      steps {
        sh 'docker build -t $ECR_REPO:$IMAGE_TAG .'
      }
    }

    stage('Login to ECR') {
      steps {
        sh 'aws ecr get-login-password --region $AWS_REGION | docker login --username AWS --password-stdin $ECR_REPO'
      }
    }

    stage('Push Image') {
      steps {
        sh 'docker push $ECR_REPO:$IMAGE_TAG'
      }
    }

    stage('Deploy to ECS') {
      steps {
        sh 'aws ecs update-service --cluster jenkins-ecs-cluster --service jenkins-ecs-service --force-new-deployment'
      }
    }
  }
}
```

---

## 10. Deployment Verification

### 10.1 Check ECS Service

* Task status: RUNNING
* Desired tasks met

### 10.2 Check Target Group

* Targets: Healthy

### 10.3 Access Application

```
http://<alb-dns-name>
```

---

## 11. Common Issues & Fixes

| Issue                  | Root Cause                  | Fix                            |
| ---------------------- | --------------------------- | ------------------------------ |
| ALB timeout            | HTTPS used without listener | Use HTTP or add HTTPS listener |
| Task IP not reachable  | App listening on localhost  | Bind to 0.0.0.0                |
| Target unhealthy       | Health check path mismatch  | Fix path                       |
| Downtime during deploy | Desired count = 1           | Set desired count >= 2         |

---

## 12. Interview Explanation (Short)

"I built a Jenkins CI/CD pipeline that builds Docker images from GitHub source code, pushes them to Amazon ECR, and deploys them to ECS Fargate behind an Application Load Balancer with zero-downtime deployment settings."

---

## 13. Resume Bullet Point

* Designed and implemented an end-to-end CI/CD pipeline using Jenkins, Docker, Amazon ECR, Amazon ECS (Fargate), and ALB for automated container deployments.

---

## 14. Conclusion

This project demonstrates real-world DevOps skills including containerization, CI/CD automation, AWS networking, load balancing, and production-grade deployment strategies.

