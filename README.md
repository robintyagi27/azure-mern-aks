# MERN Microservices Deployment on Azure Kubernetes Service (AKS)

##  Assignment Objective

This project demonstrates the complete deployment of a **MERN (MongoDB, Express, React, Node.js) Microservices application** into **Azure Kubernetes Service (AKS)** using containerization and Kubernetes orchestration.

The goal of this assignment:

- Build and containerize frontend and backend services
- Create and configure AKS cluster
- Deploy application images to Kubernetes
- Expose application using Kubernetes services
- Validate end-to-end application functionality
- Document the entire process with screenshots

# GitHub Repo Used:

https://github.com/UnpredictablePrashant/SampleMERNwithMicroservices

---

## Architecture Overview


User
|
Azure LoadBalancer Service
|
Frontend Pod Backend API Pods

(React) (Node.js Microservices)
|
MongoDB

- React UI served via Kubernetes Service
- Node.js APIs deployed as microservices
- MongoDB running as containerized pod
- Azure AKS LoadBalancer used for public access

---

##  Technology Stack

| Category     | Tools |
|---------------|------|
| Cloud         | Microsoft Azure |
| Container     | Docker |
| Orchestration| Azure Kubernetes Service (AKS) |
| Runtime       | Node.js |
| Database      | MongoDB |
| Frontend      | React |
| CI Support    | GitHub |
| CLI Tools     | Azure CLI, kubectl |

---

---

##  Deployment Steps

---

###  Step 1: Clone the Application Repository

```bash

git clone https://github.com/UnpredictablePrashant/SampleMERNwithMicroservices.git
cd SampleMERNwithMicroservices

# Step 2: Create Azure AKS Cluster

Login:

az login


Create Resource Group:

az group create --name aks-mern-rg --location eastus


Create AKS Cluster:

az aks create \
  --resource-group aks-mern-rg \
  --name mern-aks-cluster \
  --node-count 2 \
  --enable-addons monitoring \
  --generate-ssh-keys


Connect kubectl to AKS:

az aks get-credentials --resource-group aks-mern-rg --name mern-aks-cluster


Verification:

kubectl get nodes

# Step 3: Build Docker Images

Navigate to service directories:

cd frontend
docker build -t mern-frontend .

cd backend
docker build -t mern-backend .

# Step 4: Push Images to Azure Container Registry (ACR)

Create ACR:

az acr create --resource-group aks-mern-rg \
   --name mernacr123 \
   --sku Basic


Login:

az acr login --name mernacr123


Tag & push images:

docker tag mern-frontend mernacr123.azurecr.io/frontend:v1
docker tag mern-backend mernacr123.azurecr.io/backend:v1

docker push mernacr123.azurecr.io/frontend:v1
docker push mernacr123.azurecr.io/backend:v1


# Step 5: Create Kubernetes Deployment Manifests

Frontend deployment

frontend-deployment.yaml

apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend-deployment
spec:
  replicas: 2
  selector:
    matchLabels:
      app: frontend
  template:
    metadata:
      labels:
        app: frontend
    spec:
      containers:
      - name: frontend
        image: mernacr123.azurecr.io/frontend:v1
        ports:
        - containerPort: 80

Backend deployment

backend-deployment.yaml

apiVersion: apps/v1
kind: Deployment
metadata:
  name: backend-deployment
spec:
  replicas: 2
  selector:
    matchLabels:
      app: backend
  template:
    metadata:
      labels:
        app: backend
    spec:
      containers:
      - name: backend
        image: mernacr123.azurecr.io/backend:v1
        ports:
        - containerPort: 3000

# Step 6: Create Kubernetes Services

Frontend Service
apiVersion: v1
kind: Service
metadata:
  name: frontend-service
spec:
  type: LoadBalancer
  selector:
    app: frontend
  ports:
  - port: 80
    targetPort: 80

Backend Service
apiVersion: v1
kind: Service
metadata:
  name: backend-service
spec:
  type: ClusterIP
  selector:
    app: backend
  ports:
  - port: 3000
    targetPort: 3000


Apply all manifests:

kubectl apply -f frontend-deployment.yaml
kubectl apply -f backend-deployment.yaml
kubectl apply -f frontend-service.yaml
kubectl apply -f backend-service.yaml

# Step 7: Verify Application

Check Pods:

kubectl get pods


Check Services:

kubectl get svc


Copy frontend LoadBalancer External IP

Open in browser:

http://<LoadBalancer-External-IP>


# MERN application UI should load successfully
