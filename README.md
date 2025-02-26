# Deployment of User Management and Order Management Applications with PostgreSQL in Kubernetes

## Project Overview
This project automates the deployment of two Spring Boot applications (`user-management` and `order-management`) along with a PostgreSQL database using Kubernetes, Jenkins, and Ansible.

## Project Structure
```
sample_project/
│-- Ansible/
│   │-- roles/
│   │   │-- device_setup/
│   │   │-- order_management/
│   │   │-- postgres_setup/
│   │   │-- user_management/
│   │   │-- vault_password/
│   │-- complete_deployment.yml
│-- kubernetes/
│   │-- kubernetes-secrets.yml
│   │-- order-management-deployment.yml
│   │-- order-management-service.yml
│   │-- postgres-deployment.yml
│   │-- postgres-service.yml
│   │-- user-management-deployment.yml
│   │-- user-management-service.yml
│-- order_management/
│-- user_management/
│-- .gitignore
│-- Jenkinsfile
│-- README.md
```

---
## Prerequisites
- **Docker** installed and running
- **Minikube** installed and configured
- **Ansible** installed
- **Jenkins** installed with necessary plugins
- **kubectl** installed

---
## Setup Instructions

### 1. Clone the Repository
```sh
git clone https://github.com/sagarkrishnasuresh/sample_project.git
cd sample_project
```

### 2. Configure Jenkins Pipeline
Ensure Jenkins has the required tools:
- **Maven** (`Maven3` configured in Jenkins)
- **Docker** with access to Docker Hub
- **Ansible** installed

Configure Jenkins pipeline with the provided **Jenkinsfile**.

### 3. Setup Ansible Roles

#### Device Setup
Installs **Docker**, **Ansible**, and **kubectl** if not already installed.

#### PostgreSQL Setup
- Generates Kubernetes secrets for database credentials.
- Deploys PostgreSQL in Kubernetes.
- Applies PostgreSQL service configuration.

#### Application Setup (User & Order Management)
- Builds the Docker image.
- Pushes it to Docker Hub.

### 4. Run the Jenkins Pipeline

Once configured, trigger the **Jenkins Pipeline** to:
1. Clone the repository.
2. Build the JAR files.
3. Run the Ansible playbook.
4. Deploy the applications and its services in **Minikube**.
5. Cleanup residual files.

---
## Kubernetes Resources

### 1. Secrets (Stored in Kubernetes Secrets)
- `DB_URL`, `DB_USER`, and `DB_PASSWORD` are securely stored.

### 2. Deployments
- **PostgreSQL**: Deployed with a persistent volume.
- **User Management & Order Management**: Retrieves credentials from Kubernetes secrets.

### 3. Services
- **PostgreSQL**: `ClusterIP` service for internal database communication.
- **User Management**: `NodePort` for external access.
- **Order Management**: `NodePort` for external access.

---
## Verifying Deployment

### 1. Check if all pods are running
```sh
kubectl get pods
```

### 2. Get the exposed service ports
```sh
kubectl get services
```

### 3. Access APIs
#### Fetch Users
```sh
curl http://<MINIKUBE_IP>:<USER_MANAGEMENT_NODEPORT>/api/users
```
#### Create User
```sh
curl -X POST http://<MINIKUBE_IP>:<USER_MANAGEMENT_NODEPORT>/api/users \
  -H 'Content-Type: application/json' \
  -d '{"name": "John Doe", "email": "johndoe@example.com"}'
```
#### Fetch Orders
```sh
curl http://<MINIKUBE_IP>:<ORDER_MANAGEMENT_NODEPORT>/api/orders
```
#### Create Order
```sh
curl -X POST http://<MINIKUBE_IP>:<ORDER_MANAGEMENT_NODEPORT>/api/orders \
  -H 'Content-Type: application/json' \
  -d '{"userId": 1, "product": "Laptop", "quantity": 2}'
```

---
## Cleanup & Maintenance
After deployment, unnecessary files are removed via Jenkins pipeline:
- **Maven target directories**
- **Ansible temporary files**
- **Docker build cache**
- **Kubernetes logs**

To manually clean up:
```sh
kubectl delete all --all
```
To stop Minikube:
```sh
minikube stop
```

---
## Conclusion
This project provides a complete CI/CD pipeline to deploy two microservices and a PostgreSQL database in Kubernetes using Jenkins, Ansible, and Docker.

