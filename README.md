# Spring Boot Demo Application

A simple Spring Boot application deployed on Kubernetes with ArgoCD on AWS EC2.

## Quick Start

1. Follow the detailed deployment guide in `DEPLOYMENT_GUIDE.md`
2. Update Docker Hub username in `k8s/deployment.yaml`
3. Update Git repository URL in `argocd/application.yaml`
4. Build and push Docker image
5. Deploy with ArgoCD or kubectl

## Application Endpoints

- `/` - Home endpoint
- `/health` - Health check
- `/actuator/health` - Spring Boot actuator health

## Technology Stack

- Java 17
- Spring Boot 3.2.0
- Docker
- Kubernetes (k3s)
- ArgoCD
- AWS EC2
