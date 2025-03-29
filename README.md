# Kubernetes Flask Application Deployment

This guide walks through deploying a Flask application on a local Kubernetes cluster using Minikube, including auto-scaling, rolling updates, and testing scenarios.

## Prerequisites
- Docker Desktop
- Minikube
- kubectl
- Python 3.11+
- Docker Hub account

---

## Step 1: Kubernetes Cluster Setup

### 1.1 Start Minikube Cluster
```powershell
minikube start --nodes=2
```

Verify cluster status:
```powershell
kubectl get nodes
```
**Expected Output:**
```
NAME           STATUS   ROLES           AGE   VERSION
minikube       Ready    control-plane   58s   v1.32.0
minikube-m02   Ready    <none>          34s   v1.32.0
```

---

## Step 2: Flask Application Deployment

### 2.1 Build Docker Image
```dockerfile
# Dockerfile
FROM python:3.11-slim
WORKDIR /app
COPY . .
RUN pip install -r requirements.txt
EXPOSE 8080
CMD ["python", "main.py"]
```

Build and test locally:
```powershell
docker build -t ishans-flask .
docker run -p 8080:8080 ishans-flask
```

### 2.2 Push to Docker Hub
```powershell
docker tag ishans-flask <your-dockerhub-username>/flask-app
docker push <your-dockerhub-username>/flask-app
```

---

## Step 3: Kubernetes Resources

### 3.1 Apply Configurations
```powershell
kubectl apply -f configmap.yaml
kubectl apply -f secret.yaml
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml
kubectl apply -f hpa.yaml
```

Verify deployment:
```powershell
kubectl get pods, svc, hpa
```

---

## Step 4: Testing Scenarios

### 4.1 Availability Test
```powershell
minikube service hello-world-flask-service --url
curl http://<EXTERNAL-IP>:<PORT>
```
**Expected Output:** `Hello World!`

### 4.2 Auto-scaling Test
Simulate load:
```powershell
kubectl run stress-test --image=busybox -- /bin/sh -c "while true; do wget -q -O- http://hello-world-flask-service; done"
```
Monitor scaling:
```powershell
kubectl get hpa -w
```

### 4.3 Rolling Update
Update image version:
```powershell
kubectl set image deployment/hello-world-flask hello-world-flask=ishans24/flask-app:v2
kubectl rollout status deployment/hello-world-flask
```

### 4.4 Self-Healing Test
Delete a pod:
```powershell
kubectl delete pod <pod-name>
kubectl get pods -w
```

---

## Step 5: Monitoring
Access Kubernetes dashboard:
```powershell
minikube dashboard
```

View application logs:
```powershell
kubectl logs <pod-name>
```

---

## Troubleshooting
- **Secret Creation Error**: Ensure values are base64 encoded
- **Image Pull Errors**: Verify Docker Hub permissions
- **Pending Pods**: Check resource allocation in Minikube

---

## YAML Files
Find the complete configuration files in the repository:
- Deployment
- Service
- ConfigMap
- Secret
- HPA

[GitHub Repository](https://github.com/ishans2404/flask-docker-kubernetes)

---

**References:**
- [Minikube Documentation](https://minikube.sigs.k8s.io/docs/)
- [Kubernetes HPA Guide](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/)