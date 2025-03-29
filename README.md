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

### 3.1 Configuration Files

1. **configmap.yaml**
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: flask-config
data:
  FLASK_ENV: "production"
```

2. **secret.yaml** (Base64 encoded values)
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: flask-secret
type: Opaque
data:
  SECRET_KEY: "base64_encoded_value"
```

3. **deployment.yaml**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hello-world-flask
spec:
  replicas: 3
  selector:
    matchLabels:
      app: hello-world-flask
  template:
    metadata:
      labels:
        app: hello-world-flask
    spec:
      containers:
      - name: hello-world-flask
        image: ishans24/flask-app
        ports:
        - containerPort: 8080
        envFrom:
        - configMapRef:
            name: flask-config
        - secretRef:
            name: flask-secret
```

4. **service.yaml**
```yaml
apiVersion: v1
kind: Service
metadata:
  name: hello-world-flask-service
spec:
  type: LoadBalancer
  selector:
    app: hello-world-flask
  ports:
    - protocol: TCP
      port: 80
      targetPort: 8080
```

5. **hpa.yaml**
```yaml
apiVersion: autoscaling/v1
kind: HorizontalPodAutoscaler
metadata:
  name: hello-world-flask-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: hello-world-flask
  minReplicas: 2
  maxReplicas: 5
  targetCPUUtilizationPercentage: 50
```

### 3.2 Apply Configurations
```powershell
kubectl apply -f configmap.yaml
kubectl apply -f secret.yaml
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml
kubectl apply -f hpa.yaml
```

Verify deployment:
```powershell
kubectl get pods,svc,hpa
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

[GitHub Repository](https://github.com/ishans2404/flask-kubernetes-deployment)

---

**References:**
- [Minikube Documentation](https://minikube.sigs.k8s.io/docs/)
- [Kubernetes HPA Guide](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/)