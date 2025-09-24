# Kubernetes Memory Stress Test Task

## Task Overview
- Create a new namespace `mem-example`.
- Install the metrics server using the provided YAML (assumed done).
- Deploy two pods running the `polinux/stress` image with different memory limits and requests to test memory stress.

---

## Step 1: Namespace Creation

```bash
kubectl create namespace mem-example
```
## Pod YAML manifest

```bash
apiVersion: v1
kind: Pod
metadata:
  name: memory-demo
  namespace: mem-example
spec:
  containers:
  - name: memory-demo-ctr
    image: polinux/stress
    resources:
      requests:
        memory: "100Mi"
      limits:
        memory: "200Mi"
    command: ["stress"]
    args: ["--vm", "1", "--vm-bytes", "150M", "--vm-hang", "1"]
---
apiVersion: v1
kind: Pod
metadata:
  name: memory-demo-2
  namespace: mem-example
spec:
  containers:
  - name: memory-demo-2-ctr
    image: polinux/stress
    resources:
      requests:
        memory: "50Mi"
      limits:
        memory: "100Mi"
    command: ["stress"]
    args: ["--vm", "1", "--vm-bytes", "250M", "--vm-hang", "1"]




    kubectl apply -f pod.yaml

```

## Commands

```bash
kubectl get pods -n mem-example
kubectl describe pod <pod-name> -n mem-example
kubectl logs <pod-name> -n mem-example
kubectl delete pod <pod-name> -n mem-example
kubectl delete namespace mem-example


```