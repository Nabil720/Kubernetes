# Kubernetes Node Affinity Task

## Task Objectives

1. Create a Pod using the **nginx** image.
2. Apply **nodeAffinity** with `requiredDuringSchedulingIgnoredDuringExecution` condition: `disktype = ssd`.
3. Check pod status and verify it doesn't get scheduled initially.
4. Label `worker1` with `disktype=ssd` and verify that pod gets scheduled on `worker1`.
5. Create a second Pod using the **redis** image.
6. Apply **nodeAffinity** with `requiredDuringSchedulingIgnoredDuringExecution` and `operator: Exists` for key `disktype` (without any value).
7. Label `worker2` with just the key `disktype` (no value).
8. Ensure that the redis pod gets scheduled on `worker2`.

---

## Step-by-Step Execution   

### Step 1: Create nginx Pod with nodeAffinity

**`nginx-pod.yaml`**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
    app: nginx
spec:
  containers:
  - name: nginx
    image: nginx
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: disktype
            operator: In
            values:
            - ssd


kubectl apply -f nginx-pod.yaml
kubectl get pods
kubectl describe pod nginx

## Pod will be in Pending state because no node has the required label disktype=ssd.

```

## Label worker1 with disktype=ssd
```
kubectl label node worker1 disktype=ssd
kubectl get pods -o wide

## Now the nginx pod gets scheduled on worker1

```

## Create redis Pod with operator: Exists

```
apiVersion: v1
kind: Pod
metadata:
  name: redis-3
  labels:
    run: redis
spec:
  containers:
  - name: redis
    image: redis
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: disktype
            operator: Exists


kubectl apply -f redis-pod.yaml
## Initially the redis pod may not get scheduled if no node matches the Exists rule.

```

## Label worker2 with just key disktype (no value)
```
kubectl label node worker2 disktype=

##Taints & Tolerations 
kubectl taint node worker1 gpu=true:NoSchedule

kubectl get pods -o wide
```
