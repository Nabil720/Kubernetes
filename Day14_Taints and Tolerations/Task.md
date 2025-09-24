#  Kubernetes Taints & Tolerations Lab

This lab demonstrates how to use **taints** and **tolerations** in Kubernetes to control pod scheduling across different nodes.

---

## Task Overview

1. Taint both worker nodes:
   - `worker1` → `gpu=true:NoSchedule`
   - `worker2` → `gpu=false:NoSchedule`

2. Deploy an `nginx` pod and observe why it is not scheduled.

3. Add a toleration to match `gpu=true:NoSchedule`, allowing the pod to be scheduled on `worker1`.

4. Remove the taint on the control plane node.

5. Deploy a `redis` pod and observe that it is scheduled on the control plane.

6. Re-add the taint on the control plane node.

---

##  Commands & Output

### Tainting the Worker Nodes

```bash
kubectl taint node worker1 gpu=true:NoSchedule
kubectl taint node worker2 gpu=false:NoSchedule
kubectl run nginx-pod --image=nginx:latest
kubectl describe pod nginx-pod

# Output:
# 0/3 nodes are available: 
# 1 node(s) had untolerated taint {gpu: false}
# 1 node(s) had untolerated taint {gpu: true}
# 1 node(s) had untolerated taint {node-role.kubernetes.io/control-plane: }

```

## Add Toleration to nginx Pod

```
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
  labels:
    run: nginx
spec:
  containers:
    - name: nginx
      image: nginx:latest
  tolerations:
    - key: "gpu"
      operator: "Equal"
      value: "true"
      effect: "NoSchedule"

kubectl delete pod nginx-pod
kubectl apply -f pod.yaml
kubectl get pods  # nginx-pod now Running on worker1


```

## Untaint Control Plane Node

```
kubectl taint node master node-role.kubernetes.io/control-plane-
kubectl run redis --image=redis
kubectl get pods  # redis Running on master
```

## Re-Add Taint to Control Plane
```
kubectl taint node master node-role.kubernetes.io/control-plane:NoExecute
```
