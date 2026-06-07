# Blue-Green Deployment Project
Overview

This project demonstrates the implementation of a Blue-Green Deployment strategy using Docker and Kubernetes. The application consists of a Node.js backend, MongoDB database, and two frontend environments (Blue and Green) that enable seamless application updates with zero downtime.

Objectives
Containerize application components using Docker.
Orchestrate containers using Kubernetes.
Implement Blue-Green Deployment for zero-downtime releases.
Demonstrate service-based traffic switching between application versions.
Ensure application availability during deployment and rollback operations.
Architecture

The project consists of the following components:

Frontend Blue – Current production version of the application.
Frontend Green – New version deployed alongside the Blue version.
Backend Service – Node.js API server.
MongoDB – Database service.
Kubernetes Services – Provide service discovery and traffic routing.
Kubernetes Deployments – Manage application replicas and self-healing.
Blue-Green Deployment Workflow
Deploy the current application version to the Blue environment.
Deploy the updated application version to the Green environment.
Verify the Green environment independently.
Switch user traffic from Blue to Green by updating the Kubernetes Service selector.
If issues are detected, instantly roll back traffic to the Blue environment.
Traffic Flow Before Switch

Users → Frontend Service → Frontend Blue Pods

Traffic Flow After Switch

Users → Frontend Service → Frontend Green Pods

## Prerequisites
- Docker Desktop
- Minikube
- kubectl
- Helm
- Node.js
- Git

## Project Setup

### 1. Clone the Repository

<img width="1249" height="103" alt="image" src="https://github.com/user-attachments/assets/61fabdbf-4315-4d55-9600-0ce2f554d11c" />


### 2. Local Development

#### Backend Setup
1. Navigate to backend directory
2. Install dependencies
<img width="1248" height="143" alt="image" src="https://github.com/user-attachments/assets/2a5f2dc2-bb75-444c-b4c2-4a9a4c017cf3" />

3. Create `.env` file with:
<img width="1253" height="57" alt="image" src="https://github.com/user-attachments/assets/a89a5465-21c7-4140-95f8-fd80b5d01551" />

4. Start backend server
<img width="1250" height="352" alt="image" src="https://github.com/user-attachments/assets/9b90de6d-f743-47d2-b123-375133ecc8de" />


#### Frontend Setup
1. Setup Blue Frontend
<img width="1251" height="122" alt="image" src="https://github.com/user-attachments/assets/4b656c66-109a-4ec3-bb67-1ca8c830b6e1" />

2. Create `.env` file:
<img width="1251" height="95" alt="image" src="https://github.com/user-attachments/assets/4a672d10-f6ea-4a57-958b-ae899e4c99d3" />

3. Start blue frontend
```bash
npm start
```

3. Repeat similar steps for Green Frontend (with PORT=3200)

### 3. Dockerization

#### Build Docker Images
```bash
# Build Backend Image
docker build -t your-username/backend:v1 ./backend

# Build Blue Frontend Image
docker build -t your-username/frontend-blue:v1 ./frontend-blue

# Build Green Frontend Image
docker build -t your-username/frontend-green:v1 ./frontend-green
```
<img width="1254" height="168" alt="image" src="https://github.com/user-attachments/assets/44103773-498a-43fc-8936-d83438f60dd8" />


### 4. Kubernetes Deployment

#### Minikube Setup
1. Start Minikube
```bash
minikube start
```
<img width="1251" height="343" alt="image" src="https://github.com/user-attachments/assets/46fc2db0-646a-48bb-ba2d-6c7fc9d61def" />


2. Enable Required Addons
```bash
minikube addons enable metrics-server
minikube addons enable ingress
```
<img width="1252" height="158" alt="image" src="https://github.com/user-attachments/assets/1d1e4552-5264-4c3a-ae01-07b8134a8213" />

### 5. Create Kubernetes Manifest Files

#### Required Manifest Files
Create following files in `k8s/` directory:
- `backend-deployment.yaml`
- `frontend-blue-deployment.yaml`
- `frontend-green-deployment.yaml`
- `frontend-service.yaml`
- `ingress.yaml`

<img width="1248" height="141" alt="image" src="https://github.com/user-attachments/assets/c292be27-23af-446d-9ff1-472476176cf4" />

#### Service File Key Concepts
Your `frontend-service.yaml` should:
- Use selector to route traffic
- Define version (blue/green)
- Map ports correctly

<img width="1254" height="211" alt="image" src="https://github.com/user-attachments/assets/4c89738c-178d-4764-a26d-acb1659ea4e6" />

### 6. Deploy to Minikube
```bash
# Apply all manifests
kubectl apply -f k8s/


# Verify deployments
kubectl get deployments
kubectl get services
kubectl get pods
```
<img width="1254" height="383" alt="image" src="https://github.com/user-attachments/assets/4eda1a4b-6f6e-4f80-ad80-d7e76e8ce73c" />
<img width="1254" height="89" alt="image" src="https://github.com/user-attachments/assets/c59d52f1-3d7f-4796-aeb9-1c30d3561050" />


### 7. Blue-Green Switching

#### Switch Traffic Methods

1. Basic Patch Command
```bash
# Switch to Green
kubectl patch service frontend \
-p '{"spec":{"selector":{"app":"frontend","version":"green"}}}'

# Switch back to Blue
kubectl patch service frontend \
-p '{"spec":{"selector":{"app":"frontend","version":"blue"}}}'
```
<img width="1254" height="161" alt="image" src="https://github.com/user-attachments/assets/22d6becc-ab88-44d7-b958-634b898a97f8" />


2. Detailed Patch Command
```bash
kubectl patch service frontend-service --type='merge' -p '{
  "spec":{
    "selector":{
      "app":"frontend",
      "version":"green"
    }
  }
}'
```

### 8. Verification
- Check service endpoints
- Verify traffic routing
- Monitor application logs

### Troubleshooting
- `l` - Check pod status
- `kubectl logs <pod-name>` - View logs
- `kubectl describe service frontend-service` - Service details

<img width="1250" height="106" alt="image" src="https://github.com/user-attachments/assets/7ad2430b-48f8-42e7-bf0d-05a58ae41372" />
<img width="1250" height="224" alt="image" src="https://github.com/user-attachments/assets/059f0cc1-54ab-4062-bd92-f5add6405861" />

![Uploading image.png…]()

### Cleanup
```bash
# Remove deployments
kubectl delete -f k8s/

# Stop Minikube
minikube stop
```

## Blue-Green Deployment Flow Chart

```mermaid
graph TD
    A[Blue Environment Running] -->|Deploy Green| B[Green Environment Prepared]
    B -->|Validate Green| C{Green Ready?}
    C -->|Yes| D[Update Service Selector]
    C -->|No| B
    D -->|Redirect Traffic| E[Green Now Active]
    E -->|Rollback Option| A
```

### Flow Explanation
1. Blue environment is initial production
2. Green environment deployed alongside
3. Validate green environment 
4. Update service selector
5. Redirect traffic to green
6. Blue remains as rollback option

## Best Practices
- Implement health checks
- Use resource limits
- Configure monitoring
- Validate before switching
- Maintain rollback strategy


## License
This project is licensed under the MIT License
