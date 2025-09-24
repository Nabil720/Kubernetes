# Kubernetes Multi-Namespace NGINX Deployment

## Task Overview

This document outlines the steps to:
- Create two namespaces
- Deploy NGINX applications into each
- Access them via Pod IP and Service
- Test inter-namespace communication
- Clean up all resources

---

## What is a Kubernetes Namespace?

A **Namespace** in Kubernetes is a way to divide cluster resources between multiple users or teams. Namespaces help:
- Organize objects
- Avoid name collisions
- Manage access control
- Enable multi-tenant environments

### Pros:
- **Isolation** of resources (Pods, Services, Deployments)
- **Scoped resource limits** (like CPU, memory quotas)
- Supports **multi-tenant** architecture
- Useful in **CI/CD pipelines** for ephemeral environments

### Cons:
- Adds **complexity** to cluster management
- **DNS resolution** across namespaces can be tricky
- RBAC (access control) must be explicitly configured

---

##  Step-by-Step Execution

### 1. Create Namespaces

```bash
kubectl create namespace ns1
kubectl create namespace ns2

#Create Deployments with NGINX


kubectl create deployment deploy-ns1 --image=nginx --replicas=1 -n ns1
kubectl create deployment deploy-ns2 --image=nginx --replicas=1 -n ns2


# Get Pod IPs

kubectl get pods -o wide -n ns1
kubectl get pods -o wide -n ns2

# Test Pod-to-Pod Communication

kubectl exec -n ns1 -it <pod-name-ns1> -- sh
curl <pod-ip-of-ns2>

# Scale Deployments


kubectl scale deployment deploy-ns1 --replicas=3 -n ns1
kubectl scale deployment deploy-ns2 --replicas=3 -n ns2


# Create Services


kubectl expose deployment deploy-ns1 --name=svc-ns1 --port=80 --target-port=80 -n ns1
kubectl expose deployment deploy-ns2 --name=svc-ns2 --port=80 --target-port=80 -n ns2

# Get Service IPs

kubectl get svc -n ns1
kubectl get svc -n ns2

#Exec into a Pod in ns1

kubectl exec -n ns1 -it <pod-name> -- sh
curl <svc-ns2 ClusterIP>

# Test with service name

curl svc-ns2
# Fails: cannot resolve host

#Test with FQDN

curl svc-ns2.ns2.svc.cluster.local

# Clean Up Resources

kubectl delete namespace ns1
kubectl delete namespace ns2

```