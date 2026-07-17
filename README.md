# Cloud Native CI/CD Pipeline on AWS EKS

An end-to-end DevOps project demonstrating how to provision AWS infrastructure, build and deploy containerized applications, and automate CI/CD using Terraform, Jenkins, Docker and Kubernetes on Amazon EKS.

---

# Architecture

```text
Developer
    │
    ▼
GitHub
    │
    ▼
Jenkins Pipeline
    │
    ▼
Build Docker Image
    │
    ▼
Trivy Security Scan
    │
    ▼
Push Amazon ECR
    │
    ▼
Deploy Amazon EKS
    │
    ▼
Verify Rollout
```

---

# Tech Stack

## Cloud

- Amazon Web Services (AWS)
- Amazon EKS
- Amazon EC2
- Amazon ECR
- Amazon RDS
- IAM
- VPC

## DevOps

- Terraform
- Jenkins
- Docker
- Kubernetes
- Git
- GitHub

## Security

- Trivy

## Application

- Python
- Flask
- MySQL

---

# Project Features

- Infrastructure provisioning with Terraform
- Docker containerization
- Jenkins CI/CD Pipeline
- Trivy container image vulnerability scanning
- Amazon ECR image management
- Amazon EKS deployment
- Kubernetes Deployment
- Kubernetes Service
- Kubernetes Secret
- Kubernetes Ingress
- Readiness Probe
- Liveness Probe
- Automatic Rollout Verification
- Amazon RDS database integration

---

# Infrastructure

Terraform provisions the following AWS resources:

- VPC
- Public Subnets
- Private Subnets
- Internet Gateway
- Route Tables
- Security Groups
- IAM Roles
- Amazon EC2
- Amazon EKS Cluster
- Amazon ECR Repository
- Amazon RDS MySQL

---

# Kubernetes Resources

The application is deployed using:

- Namespace
- Secret
- Deployment
- Service
- Ingress

Application health is monitored through:

- Readiness Probe
- Liveness Probe

---

# CI/CD Workflow

1. Developer pushes source code to GitHub.
2. Jenkins automatically starts the pipeline.
3. Build Docker image.
4. Scan Docker image using Trivy.
5. Stop the pipeline if HIGH or CRITICAL vulnerabilities are detected.
6. Push Docker image to Amazon ECR.
7. Deploy application to Amazon EKS.
8. Verify Kubernetes rollout status.

---

# Jenkins Pipeline

Pipeline stages:

- Build Docker Image
- Trivy Security Scan
- Push Docker Image to Amazon ECR
- Deploy to Amazon EKS
- Verify Kubernetes Rollout

---

# Trivy Security Scan

The Jenkins Pipeline integrates Trivy to improve container security.

Features:

- Scan Docker images before deployment
- Detect HIGH and CRITICAL vulnerabilities
- Prevent vulnerable images from being deployed
- Improve CI/CD security

Example command:

```bash
trivy image \
  --severity HIGH,CRITICAL \
  --exit-code 1 \
  IMAGE_NAME
```

---

# Deployment Verification

After deployment Jenkins executes:

```bash
kubectl rollout status deployment/app \
-n app \
--timeout=180s
```

The deployment is considered successful only after Kubernetes reports a successful rollout.

---

# Application Flow

```text
GitHub

↓

Jenkins

↓

Docker Build

↓

Trivy Scan

↓

Amazon ECR

↓

Amazon EKS

↓

Kubernetes Deployment

↓

Application Running
```

---

# Repository Structure

```
.
├── terraform/
├── jenkins/
├── docker/
├── k8s/
│   ├── app.yaml
├── app/
├── Jenkinsfile
├── Dockerfile
└── README.md
```

---

# Skills Demonstrated

- Linux Administration
- Git & GitHub
- Docker
- Kubernetes
- Jenkins
- Terraform
- AWS Cloud
- Amazon EKS
- Amazon ECR
- Amazon RDS
- CI/CD
- Infrastructure as Code
- Container Security
- DevSecOps
- NGINX Ingress
- Readiness/Liveness Probes

---

# Future Improvements

- Helm Charts
- ArgoCD GitOps
- Horizontal Pod Autoscaler
- Prometheus Monitoring
- Grafana Dashboard
- Slack Notifications

---

# Author

Bui Tan Tai

GitHub:
https://github.com/asbass/cicd
