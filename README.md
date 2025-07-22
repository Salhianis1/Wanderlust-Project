# 🚀 Deploy Wanderlust on Minikube using Jenkins and ArgoCD

This guide demonstrates how to automate the CI/CD process for the Wanderlust application using **Jenkins** (for building and pushing Docker images) and **ArgoCD** (for GitOps-based Kubernetes deployment).

---

## ✅ Prerequisites

- [x] Minikube installed and running
- [x] Docker installed
- [x] Jenkins installed (locally or on a remote server)
- [x] DockerHub account
- [x] ArgoCD installed in Minikube
- [x] GitHub repository with your project

---

## 📦 Project Structure

Wanderlust-Project/
├── frontend/
├── backend/
├── kubernetes/
├── manifests/ <-- ArgoCD watches this folder
│ ├── frontend.yaml
│ ├── backend.yaml
│ ├── mongodb.yaml
│ ├── redis.yaml
│ ├── persistentVolume.yaml
│ └── persistentVolumeClaim.yaml
└── Jenkinsfile





---

## ⚙️ Step 1: Start Minikube and Create Namespace

```bash
minikube start --memory=4096 --cpus=2
kubectl create namespace wanderlust
