# Cloud Native CI/CD Pipeline on AWS EKS

![AWS](https://img.shields.io/badge/AWS-EKS-orange)
![Terraform](https://img.shields.io/badge/Terraform-IaC-623CE4)
![Docker](https://img.shields.io/badge/Docker-Container-2496ED)
![Kubernetes](https://img.shields.io/badge/Kubernetes-EKS-326CE5)
![Jenkins](https://img.shields.io/badge/Jenkins-CI%2FCD-D24939)
![Trivy](https://img.shields.io/badge/Trivy-Security-1904DA)

## Overview

This project demonstrates a complete cloud-native DevOps workflow on AWS.

The infrastructure is provisioned using Terraform, applications are containerized with Docker, CI/CD is automated through Jenkins, container images are stored in Amazon ECR, and workloads are deployed to Amazon EKS. The deployment pipeline also integrates Trivy for container image vulnerability scanning and performs Kubernetes rollout verification before completing deployment.

---

# Project Architecture

> 📷 **TODO:** Add AWS Architecture Diagram here.

Example:

```
Developer
     │
 Git Push
     │
 GitHub Repository
     │
 Webhook
     │
 Jenkins Pipeline
     │
 ├── Build Docker Image
 ├── Trivy Scan
 ├── Push Amazon ECR
 └── Deploy Amazon EKS
              │
        NGINX Ingress
              │
      Kubernetes Service
              │
        Application Pods
              │
          Amazon RDS
```

---

# CI/CD Workflow

> 📷 **TODO:** Add Jenkins Pipeline Workflow Diagram here.

```
Git Push
    │
    ▼
GitHub
    │
Webhook
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
Push Image to Amazon ECR
    │
    ▼
Deploy to Amazon EKS
    │
    ▼
Verify Kubernetes Rollout
    │
    ▼
Application Running
```

---

# Technology Stack

| Category | Technology |
|------------|------------|
| Cloud | AWS |
| IaC | Terraform |
| Container | Docker |
| CI/CD | Jenkins |
| Registry | Amazon ECR |
| Orchestration | Amazon EKS |
| Security | Trivy |
| Networking | NGINX Ingress |
| Database | Amazon RDS |
| OS | Ubuntu Linux |

---

# Infrastructure Provisioning

Terraform provisions the following resources:

- VPC
- Public Subnets
- Private Subnets
- Internet Gateway
- Route Tables
- Security Groups
- IAM Roles
- Amazon EKS
- Amazon ECR
- Amazon RDS
- EC2 (Jenkins)

---

# Jenkins Pipeline

The CI/CD pipeline consists of the following stages.

| Stage | Description |
|--------|-------------|
| Build | Build Docker Image |
| Scan | Trivy Security Scan |
| Push | Push Docker Image to Amazon ECR |
| Deploy | Apply Kubernetes Manifest |
| Verify | Verify Kubernetes Rollout |
| Cleanup | Remove Local Docker Image |

---

# Trivy Security Scan

Trivy is integrated into the Jenkins Pipeline to improve deployment security.

Features:

- Scan Docker Images
- Detect HIGH Vulnerabilities
- Detect CRITICAL Vulnerabilities
- Stop deployment if critical vulnerabilities are detected

---

# Kubernetes Deployment

The application is deployed to Amazon EKS using Kubernetes manifests.

Resources:

- Namespace
- Secret
- Deployment
- Service
- Ingress

Deployment Strategy

- Rolling Update
- Rollout Verification

Health Checks

- Readiness Probe
- Liveness Probe

---

# Repository Structure

```
.
├── Dockerfile
├── Jenkinsfile
├── k8s/
│   └── app.yaml
├── terraform/
├── templates/
├── requirements.txt
├── README.md
└── Images/
```

> ⚠️ **TODO:** Update this structure if you rename folders (recommended).

---

# Project Outcome

- Successfully provisioned AWS infrastructure using Terraform.
- Built and deployed a containerized Python application on Amazon EKS.
- Automated CI/CD using Jenkins.
- Integrated Trivy image vulnerability scanning.
- Implemented Kubernetes rollout verification.
- Published Infrastructure as Code and deployment workflow on GitHub.

---

# Project Metrics

| Metric | Value |
|---------|------:|
| AWS Resources Provisioned | 10+ |
| Jenkins Pipeline Stages | 6 |
| Kubernetes Resources | 5 |
| Container Registry | Amazon ECR |
| Security Scanner | Trivy |
| Deployment Platform | Amazon EKS |

---

# Screenshots

## Jenkins Pipeline

> 📷 **TODO:** Add successful Jenkins Pipeline screenshot.

---

## Trivy Scan Result

> 📷 **TODO:** Add Trivy Scan screenshot.

---

## Amazon EKS Pods

> 📷 **TODO:** Add kubectl get pods screenshot.

---

## Application

> 📷 **TODO:** Add application running screenshot.

---

# Future Improvements

- GitOps with ArgoCD
- Helm Chart
- Prometheus Monitoring
- Grafana Dashboard
- Horizontal Pod Autoscaler (HPA)
- SonarQube Integration
- Slack / Telegram Notification

---

# Skills Demonstrated

- Infrastructure as Code (Terraform)
- AWS Cloud Infrastructure
- Docker Containerization
- Jenkins CI/CD
- Kubernetes Deployment
- Amazon EKS
- Amazon ECR
- Trivy Security Scanning
- Rollout Verification
- Git & GitHub
