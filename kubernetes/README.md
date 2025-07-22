# Wanderlust Deployment on Kubernetes

### In this project, we will learn about how to deploy wanderlust application on Kubernetes.

### Pre-requisites to implement this project:

- [x] Minikube installed and running
- [x] Docker installed
- [x] Jenkins installed (locally or on a remote server)
- [x] DockerHub account
- [x] ArgoCD installed in Minikube
- [x] GitHub repository with your project
# Minikube Installation and Setup Guide (Linux)

```bash
# 1. Install Docker (if not installed)
sudo apt update
sudo apt install -y docker.io
sudo systemctl enable docker
sudo systemctl start docker
sudo usermod -aG docker $USER
newgrp docker  # apply group change immediately

# 2. Install Minikube
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube

# 3. Install kubectl
curl -LO "https://dl.k8s.io/release/$(curl -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
chmod +x kubectl
sudo mv kubectl /usr/local/bin/

# 4. Start Minikube with Docker driver
minikube start --driver=docker

# 5. Check Minikube status
minikube status

# 6. Verify Kubernetes nodes
kubectl get nodes

# 7. Verify all pods (in all namespaces)
kubectl get pods -A

# 8. Open Minikube dashboard (optional)
minikube dashboard

# 9. Create 'wanderlust' namespace if it doesn't exist
kubectl create namespace wanderlust

# 10. Set current kubectl context to use 'wanderlust' namespace by default
kubectl config set-context --current --namespace=wanderlust


# 11. Enable DNS resolution on Kubernetes cluster

# Check CoreDNS pods and their node placement:
kubectl get pods -n kube-system -o wide | grep -i coredns

# Edit CoreDNS deployment to scale replicas from 2 to 4 (run pods on worker nodes too)
kubectl edit deploy coredns -n kube-system

# In the editor, find the replicas field and change:
# replicas: 2
# to
# replicas: 4

# Save and exit editor to apply changes

# Verify updated CoreDNS pods:
kubectl get pods -n kube-system -o wide | grep -i coredns
