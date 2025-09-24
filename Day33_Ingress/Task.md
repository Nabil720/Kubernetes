# Kubernetes Ingress Path-Based Routing with Two NGINX Apps

In this guide, we will deploy two NGINX applications in a Kubernetes cluster and use an Ingress Controller to perform path-based routing.

---

## Step 1: Install Ingress Controller

```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.10.1/deploy/static/provider/cloud/deploy.yaml

kubectl get pods -n ingress-nginx

```

## Deploy Two NGINX Apps with Services

```bash
# multi-app.yaml
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app1
spec:
  replicas: 1
  selector:
    matchLabels:
      app: app1
  template:
    metadata:
      labels:
        app: app1
    spec:
      containers:
      - name: nginx
        image: nginx:latest
        ports:
        - containerPort: 80

---
apiVersion: v1
kind: Service
metadata:
  name: app1
spec:
  selector:
    app: app1
  ports:
  - port: 80
    targetPort: 80
  type: ClusterIP

---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app2
spec:
  replicas: 1
  selector:
    matchLabels:
      app: app2
  template:
    metadata:
      labels:
        app: app2
    spec:
      containers:
      - name: nginx
        image: nginx:latest
        ports:
        - containerPort: 80

---
apiVersion: v1
kind: Service
metadata:
  name: app2
spec:
  selector:
    app: app2
  ports:
  - port: 80
    targetPort: 80
  type: ClusterIP


# Apply 
kubectl apply -f multi-app.yaml
kubectl get pods
kubectl get svc


```


## Create Ingress Resource with Path-Based Routing

```bash
# multi-app-ingress.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: multi-app-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  ingressClassName: nginx
  rules:
  - http:
      paths:
      - path: /app1
        pathType: Prefix
        backend:
          service:
            name: app1
            port:
              number: 80
      - path: /app2
        pathType: Prefix
        backend:
          service:
            name: app2
            port:
              number: 80

# Apply
kubectl apply -f multi-app-ingress.yaml
kubectl get ingress
kubectl get svc -n ingress-nginx

# chack
curl http://192.168.56.10:31678/app1
curl http://192.168.56.10:31678/app2

```
## Cleanup

```bash

kubectl delete -f multi-app.yaml
kubectl delete -f multi-app-ingress.yaml
kubectl delete -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.10.1/deploy/static/provider/cloud/deploy.yaml
```
