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
# 10. Update kubernetes config context : 
kubectl config set-context --current --namespace wanderlust

#
6) Enable DNS resolution on kubernetes cluster :

- Check coredns pod in kube-system namespace and you will find <i> Both coredns pods are running on master node </i>

```bash
kubectl get pods -n kube-system -o wide | grep -i core
```
![Alt text](https://github.com/DevMadhup/wanderlust/blob/devops/kubernetes/assets/get-coredns.png)

- Above step will run coredns pod on worker node as well for DNS resolution

```bash
kubectl edit deploy coredns -n kube-system -o yaml
```
<i> Make replica count from 2 to 4 </i>

![replica 4](https://github.com/DevMadhup/wanderlust/blob/devops/kubernetes/assets/edit-coredns.png)

#
7) Navigate to frontend directory :
```bash
cd frontend
```

#
8) Edit .env.docker file and change the public IP Address with your worker node public IP :
```bash
vi .env.docker
```
![IP](https://github.com/DevMadhup/wanderlust/blob/devops/kubernetes/assets/frontend.env.docker.png)

#
9) Build frontend docker image : 
```bash
docker build -t madhupdevops/frontend-wanderlust:v2.1.8 .
```
![Dockerfile frontend](https://github.com/DevMadhup/wanderlust/blob/devops/kubernetes/assets/docker%20frontend%20build.png)

#
10) Navigate to backend directory :
```bash
cd ../backend/
```

#
11) Open .env.docker file and edit below variables : 

    - MONGODB_URI: \<your-mongodb-servicename>
    - REDIS_URL: \<your-redis-servicename>
    - FRONTEND_URL: \<your-workernode-publicIP>

> Note: To get service names, check <u>mongodb.yaml, redis.yaml</u>

![Backend env file](https://github.com/DevMadhup/wanderlust/blob/devops/kubernetes/assets/backend.env.docker.png)

#
12) Build backend docker image : 
```bash
docker build -t madhupdevops/backend-wanderlust:v2.1.8 .
```
![Backend dockerfile](https://github.com/DevMadhup/wanderlust/blob/devops/kubernetes/assets/docker%20backend%20build.png)

#
13) Check docker images:
```bash
docker images
```
![docker images](https://github.com/DevMadhup/wanderlust/blob/devops/kubernetes/assets/docker%20images.png)

#
14) Login to DockerHub and push image to DockerHub
```bash
docker login
```
![docker login](https://github.com/DevMadhup/wanderlust/blob/devops/kubernetes/assets/docker%20login.png)

```bash
docker push madhupdevops/frontend-wanderlust:v2.1.8
docker push madhupdevops/backend-wanderlust:v2.1.8
```

#
15) Once, Image is pushed to DockerHub, navigate to kubernetes directory
```bash
cd ../kubernetes
```

#
16) Apply manifests file the below order:

    - Create persistent volume :
    ```bash
    kubectl apply -f persistentVolume.yaml 
    ```
    ![Peristent volume](https://github.com/DevMadhup/wanderlust/blob/devops/kubernetes/assets/pv.png)

    - Create persistent volume Claim :
    ```bash
    kubectl apply -f persistentVolumeClaim.yaml 
    ```
    ![Peristent volume Claim](https://github.com/DevMadhup/wanderlust/blob/devops/kubernetes/assets/pvc.png)

    - Create MongoDB deployment and service :
    ```bash
    kubectl apply -f mongodb.yaml 
    ```
    ![MongoDb](https://github.com/DevMadhup/wanderlust/blob/devops/kubernetes/assets/mongo.png)

    - Create Redis deployment and service :
    > Note: Wait for 3-4 mins to get mongodb, redis pods and service should be up, otherwise backend-service will not connect.
    ```bash
    kubectl apply -f redis.yaml 
    ```
    ![Redis](https://github.com/DevMadhup/wanderlust/blob/devops/kubernetes/assets/redis.png)

    - Create Backend deployment and service :
    ```bash
    kubectl apply -f backend.yaml 
    ```
    ![Backend](https://github.com/DevMadhup/wanderlust/blob/devops/kubernetes/assets/backend.png)

    - Create Frontend deployment and service :
    ```bash
    kubectl apply -f frontend.yaml
    ```
    ![Frontend](https://github.com/DevMadhup/wanderlust/blob/devops/kubernetes/assets/frontend.png)

#
17)  Check all deployments and services :
```bash 
kubectl get all
```
![all deployments and services](https://github.com/DevMadhup/wanderlust/blob/devops/kubernetes/assets/all-deps.png)

18) Check logs for all the pods :
> Note: This is mandatory to ensure all pods and services are connected or not, if not then recreate deployments
```bash
kubectl logs <pod-name>
```

20) Navigate to chrome and access your application at 31000 port :
```bash
http://<your-workernode-publicip>:31000/
```
![App](https://github.com/DevMadhup/wanderlust/blob/devops/kubernetes/assets/app.png)

#
