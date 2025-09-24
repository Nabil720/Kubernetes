# Kubernetes Nginx Deployment Walkthrough

This document covers a full deployment lifecycle of an `nginx` application in Kubernetes, including creation, updates, scaling, rollout history, and rollback.

---

##  Step 1: Create Deployment

Deployment file: `dy.yaml`

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
  labels:
    tier: backend
spec:
  replicas: 3
  selector:
    matchLabels:
      app: v1
  template:
    metadata:
      labels:
        app: v1
    spec:
      containers:
      - name: nginx
        image: nginx:1.23.4
        ports:
        - containerPort: 80



kubectl apply -f dy.yaml
kubectl get pods

# Verify 3 Replicas Are Running

kubectl get pods -l app=v1 -o wide

# Update Image
# You updated the image by modifying dy.yaml

kubectl apply -f dy.yaml
kubectl rollout status deployment/nginx

#Scale the Deployment

kubectl scale deployment nginx --replicas=5
kubectl get pods -o wide

# Check Rollout History

kubectl rollout history deployment/nginx

# Rollback to Revision 1

kubectl rollout undo deployment nginx --to-revision=1
kubectl rollout status deployment/nginx


```bash
nano nginx-deployment.md