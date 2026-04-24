# Spring Boot Application Deployment with Docker, Kubernetes & ArgoCD on AWS EC2

## Prerequisites
- AWS EC2 instance (t3.medium or larger recommended)
- Ubuntu 22.04 LTS
- Docker Hub account
- GitHub account

## Step 1: Prepare AWS EC2 Instance

### 1.1 Launch EC2 Instance
```bash
# Instance specifications:
# - Type: t3.medium (2 vCPU, 4GB RAM minimum)
# - OS: Ubuntu 22.04 LTS
# - Storage: 30GB
# - Security Group: Open ports 22, 80, 443, 8080, 30000-32767
```

### 1.2 Connect to EC2
```bash
ssh -i your-key.pem ubuntu@<EC2_PUBLIC_IP>
```

### 1.3 Update System
```bash
sudo apt update && sudo apt upgrade -y
```

## Step 2: Install Docker

```bash
# Install Docker
sudo apt install -y apt-transport-https ca-certificates curl software-properties-common
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
echo "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io

# Add user to docker group
sudo usermod -aG docker $USER
newgrp docker

# Verify installation
docker --version
```

## Step 3: Install Kubernetes (k3s)

```bash
# Install k3s (lightweight Kubernetes)
curl -sfL https://get.k3s.io | sh -

# Configure kubectl
mkdir -p ~/.kube
sudo cp /etc/rancher/k3s/k3s.yaml ~/.kube/config
sudo chown $USER:$USER ~/.kube/config
export KUBECONFIG=~/.kube/config

# Verify installation
kubectl version
kubectl get nodes
```

## Step 4: Install ArgoCD

```bash
# Create ArgoCD namespace
kubectl create namespace argocd

# Install ArgoCD
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

# Wait for ArgoCD pods to be ready
kubectl wait --for=condition=Ready pods --all -n argocd --timeout=300s

# Expose ArgoCD server
kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "NodePort"}}'

# Get ArgoCD admin password
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d
echo ""

# Get ArgoCD server port
kubectl get svc argocd-server -n argocd -o jsonpath='{.spec.ports[0].nodePort}'
echo ""

# Access ArgoCD UI at: http://<EC2_PUBLIC_IP>:<NODEPORT>
# Username: admin
# Password: (from command above)
```

## Step 5: Build and Push Docker Image

### 5.1 On Your Local Machine or EC2

```bash
# Clone your repository
git clone <YOUR_REPO_URL>
cd <YOUR_REPO_NAME>

# Login to Docker Hub
docker login

# Build Docker image
docker build -t <YOUR_DOCKERHUB_USERNAME>/demo-app:latest .

# Push to Docker Hub
docker push <YOUR_DOCKERHUB_USERNAME>/demo-app:latest
```

## Step 6: Configure Kubernetes Manifests

### 6.1 Update deployment.yaml
```bash
# Edit k8s/deployment.yaml
# Replace <YOUR_DOCKERHUB_USERNAME> with your actual Docker Hub username
```

## Step 7: Push Code to GitHub

```bash
# Initialize git repository (if not already)
git init
git add .
git commit -m "Initial commit"
git branch -M main
git remote add origin <YOUR_GITHUB_REPO_URL>
git push -u origin main
```

## Step 8: Deploy with ArgoCD

### 8.1 Update ArgoCD Application Manifest
```bash
# Edit argocd/application.yaml
# Replace <YOUR_GIT_REPO_URL> with your GitHub repository URL
```

### 8.2 Deploy Application via ArgoCD
```bash
# Apply ArgoCD application
kubectl apply -f argocd/application.yaml

# Check application status
kubectl get applications -n argocd

# Watch deployment
kubectl get pods -w
```

### 8.3 Alternative: Deploy Directly with kubectl
```bash
# If you want to deploy without ArgoCD
kubectl apply -f k8s/deployment.yaml
kubectl apply -f k8s/service.yaml
```

## Step 9: Verify Deployment

```bash
# Check pods
kubectl get pods

# Check services
kubectl get svc

# Get application URL
kubectl get svc demo-app-service -o jsonpath='{.status.loadBalancer.ingress[0].ip}'

# For k3s, use NodePort
kubectl get svc demo-app-service -o jsonpath='{.spec.ports[0].nodePort}'
# Access at: http://<EC2_PUBLIC_IP>:<NODEPORT>

# Check logs
kubectl logs -l app=demo-app

# Test application
curl http://<EC2_PUBLIC_IP>:<PORT>/
```

## Step 10: ArgoCD UI Management

1. Access ArgoCD UI: `http://<EC2_PUBLIC_IP>:<ARGOCD_NODEPORT>`
2. Login with admin credentials
3. You should see your application
4. Click on the application to see deployment details
5. Use "Sync" button to manually sync if needed

## Continuous Deployment Workflow

1. Make changes to your code
2. Build new Docker image with new tag:
   ```bash
   docker build -t <YOUR_DOCKERHUB_USERNAME>/demo-app:v1.1 .
   docker push <YOUR_DOCKERHUB_USERNAME>/demo-app:v1.1
   ```
3. Update `k8s/deployment.yaml` with new image tag
4. Commit and push to GitHub
5. ArgoCD will automatically detect changes and deploy (if auto-sync enabled)

## Troubleshooting

### Check Pod Status
```bash
kubectl describe pod <POD_NAME>
kubectl logs <POD_NAME>
```

### Check ArgoCD Application
```bash
kubectl get application -n argocd
kubectl describe application demo-app -n argocd
```

### Restart Deployment
```bash
kubectl rollout restart deployment demo-app
```

### Delete and Redeploy
```bash
kubectl delete -f k8s/
kubectl apply -f k8s/
```

## Cleanup

```bash
# Delete application
kubectl delete -f argocd/application.yaml
kubectl delete -f k8s/

# Uninstall ArgoCD
kubectl delete -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
kubectl delete namespace argocd

# Uninstall k3s
/usr/local/bin/k3s-uninstall.sh
```

## Security Notes

- Change ArgoCD admin password after first login
- Use private Docker registry for production
- Configure proper RBAC in Kubernetes
- Use secrets management for sensitive data
- Enable HTTPS for ArgoCD and application endpoints
- Restrict EC2 security group rules to specific IPs

## Next Steps

- Set up SSL/TLS certificates
- Configure ingress controller
- Implement monitoring with Prometheus/Grafana
- Set up logging with ELK stack
- Configure backup and disaster recovery
- Implement CI/CD pipeline with Jenkins/GitHub Actions
