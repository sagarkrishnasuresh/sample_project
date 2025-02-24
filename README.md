# 🚀 Spring Boot Deployment Using Jenkins, Docker, Ansible, and Minikube

This project demonstrates deploying two **Spring Boot** applications — **User Management** and **Order Management** — with a shared **PostgreSQL** database using **Docker**, **Ansible**, and **Minikube**, all orchestrated by **Jenkins CI/CD pipeline**.

---

## 💁️ **Project Directory Structure**
```
.
sample_project
├── Ansible
│   ├── ansible.cfg
│   ├── complete_deployment.yml
│   ├── vault_password
│   └── roles
│       ├── device_setup
│       │   └── tasks
│       │       └── main.yml
│       ├── postgres_setup
│       │   ├── defaults
│       │   │   └── main.yml (Encrypted with Ansible Vault)
│       │   ├── tasks
│       │   │   └── main.yml
│       │   └── templates
│       │       └── kubernetes-secrets.yml.j2
│       ├── user_management
│       │   └── tasks
│       │       └── main.yml
│       └── order_management
│           └── tasks
│               └── main.yml
├── Jenkinsfile
├── kubernetes
│   ├── kubernetes-secrets.yml
│   ├── postgres-deployment.yml
│   ├── postgres-service.yml
│   ├── user_management-deployment.yml
│   ├── user_management-service.yml
│   ├── order_management-deployment.yml
│   └── order_management-service.yml
├── order_management
│   ├── Dockerfile
│   └── src
└── user_management
    ├── Dockerfile
    └── src
```

---

## ✅ **1. Prerequisites**
Ensure the following tools are installed on your system:
- **Docker** (with Docker daemon running)
- **Jenkins** (installed and configured)
- **Ansible** (with Ansible Vault)
- **Minikube** (for Kubernetes)
- **kubectl** (Kubernetes CLI)
- **Git** (for version control)

---

## 📺 **2. Docker Hub Setup**
1. **Create a Docker Hub account:**  
   [Docker Hub](https://hub.docker.com/)

2. **Create two repositories:**
    - `docker_username/user_management`
    - `docker_username/order_management`

3. **Login to Docker locally:**
```bash
docker login
```

---

## 🔑 **3. Store Sensitive Information Using Ansible Vault**
1. **Encrypt Database Credentials and Docker Hub Credentials:**
```yaml
# Ansible/roles/postgres_setup/defaults/main.yml
POSTGRES_USER: "db_username"
POSTGRES_PASSWORD: "db_password"
POSTGRES_DB: "db_name"
SPRING_DATASOURCE_URL: "jdbc:postgresql://postgres-service:5432/db_name"
dockerhub_username: "docker_username"
dockerhub_password: "YOUR_DOCKERHUB_PASSWORD"
dockerhub_email: "YOUR_EMAIL"
```

2. **Encrypt the file using Ansible Vault:**
```bash
ansible-vault encrypt roles/postgres_setup/defaults/main.yml
```

3. **Store the vault password:**
```bash
echo "YourVaultPassword" > Ansible/vault_password/.vault_pass
```

---

## 💪 **4. Build and Push Docker Images**
1. **Dockerfile for both applications:**
```Dockerfile
FROM openjdk:17-jdk-slim
WORKDIR /app
COPY target/app.jar app.jar
ENTRYPOINT ["java","-jar","app.jar"]
```

2. **Build and Push Docker Images:**
```bash
docker build -t docker_username/user_management:latest .
docker push docker_username/user_management:latest

docker build -t docker_username/order_management:latest .
docker push docker_username/order_management:latest
```

---

## 🛠 **5. Kubernetes Deployment and Services**
### PostgreSQL Deployment:
- `postgres-deployment.yml`
- `postgres-service.yml`

### Applications Deployment:
- `user_management-deployment.yml`
- `user_management-service.yml`
- `order_management-deployment.yml`
- `order_management-service.yml`

---

## 📄 **6. Ansible Playbook Execution**
Run the **Ansible playbook**:
```bash
ansible-playbook complete_deployment.yml
```

---

## 🌐 **7. Jenkins Pipeline Setup**
1. **Configure Jenkins Pipeline with `Jenkinsfile`**
2. **Store Docker credentials in Jenkins**
3. **Trigger Pipeline Execution**

---

## 🚀 **8. Deploy Applications to Minikube**
```bash
kubectl get pods
kubectl get services
kubectl port-forward deployment/user-management 8080:8080
kubectl port-forward deployment/order-management 8082:8082
```
- **User Management:** [http://localhost:8080](http://localhost:8080)
- **Order Management:** [http://localhost:8082](http://localhost:8082)

---

## 🚚 **9. Clean Up Resources**
```bash
kubectl delete deployment user-management order-management postgres
kubectl delete service user-management-service order-management-service postgres-service
minikube stop
minikube delete
```

---

## 📈 **10. Summary**
- **CI/CD Pipeline:** Jenkins automates build and deployment.
- **Kubernetes:** Applications and PostgreSQL deployed on Minikube.
- **Ansible Vault:** Ensures security for credentials.
- **Docker Hub:** Stores and pulls images securely.

🚀 **Happy DevOps Journey!** 🐋🔥

