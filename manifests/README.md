# E-Commerce Kubernetes Deployment

## 📌 Overview
This project demonstrates how to deploy a production-grade e-commerce microservices application on Kubernetes (EKS-ready). The architecture includes a React + Nginx frontend, Node.js backend microservices, MongoDB, and RabbitMQ. The solution emphasizes scalability, security, and real-world operational readiness.

---

## 🧱 Architecture Overview

+--------------------+ +--------------------+
| Internet Client | <---> | Ingress Nginx |
+--------------------+ +--------------------+
|
v
+----------------+
| Frontend |
| (React+Nginx) |
+----------------+
|
---------------------------------------------
| | | |
v v v v
+-----------+ +------------+ +------------+ +------------+
| User Svc | | Order Svc | | Inventory | | RabbitMQ |
| (Node.js) | | (Node.js) | | (Node.js) | | Queue |
+-----------+ +------------+ +------------+ +------------+
\ | | /
_________|______________|_____________/
|
v
+---------------+
| MongoDB |
+---------------+


---

## 📦 Components Description

### 1. **Frontend (React + Nginx)**
- Serves static frontend assets.
- Uses an Ingress controller to expose the application at `http://ecommerce.local`.
- Resources are constrained to ensure performance.

### 2. **Backend Microservices (Node.js)**
- **User Service**: Manages user registration and profiles.
- **Order Service**: Handles order placement and status.
- **Inventory Service**: Tracks product inventory levels.
- Communicates internally via Kubernetes DNS service discovery.
- Uses RabbitMQ for asynchronous operations and MongoDB for data persistence.

### 3. **MongoDB**
- Acts as the main data store for all backend services.
- Mounted on a Persistent Volume (`mongo-pvc`).
- Secured via Secrets for username and password.

### 4. **RabbitMQ**
- Facilitates asynchronous order processing and decoupling between services.
- Mounted on a Persistent Volume (`rabbitmq-pvc`).
- Exposes management UI on port 15672 (optional).

### 5. **Security**
- Uses Pod Security Policies to enforce least-privilege container execution.
- RBAC roles and bindings restrict PSP access.
- Secrets and ConfigMaps manage sensitive configuration.
- NetworkPolicies restrict pod-to-pod communication.

### 6. **Scalability**
- Horizontal Pod Autoscalers added for all backend services.
- Default scaling based on CPU usage (70% threshold).

---

## 🚀 Deployment Instructions

### Prerequisites
- A running Kubernetes cluster (EKS recommended).
- `kubectl` configured with appropriate context.
- NGINX Ingress controller installed.

### Step-by-Step

```sh
# 1. Apply RBAC and PodSecurityPolicy
kubectl apply -f psp-rbac.yaml

# 2. Create Secrets and ConfigMaps
kubectl apply -f mongodb-secret.yaml

# 3. Deploy MongoDB and RabbitMQ
kubectl apply -f pvc.yaml
kubectl apply -f mongodb-service.yaml
kubectl apply -f mongodb-statefulset.yaml
kubectl apply -f rabbitmq.yaml

# 4. Deploy Backend Services
kubectl apply -f backend-microservice.yaml

# 5. Deploy Frontend
kubectl apply -f frontend.yaml

# 6. Set up Ingress
kubectl apply -f ingress.yaml

# 7. Deploy Autoscalers
kubectl apply -f frontend-hpa.yaml
kubectl apply -f backend-hpa.yaml


# 8. Apply Network Policies
kubectl apply -f network-policy.yaml


## Post-Deployment

Add the following entry to your /etc/hosts for local testing:

<LOAD_BALANCER_IP>  ecommerce.local

Access the app at: http://ecommerce.local

Monitor HPA using:

kubectl get hpa

## Validation Checklist

All pods running and healthy (kubectl get pods)

Services accessible internally and via Ingress

MongoDB and RabbitMQ use persistent volumes

HPA scales pods under load

PodSecurityPolicy and NetworkPolicy are enforced