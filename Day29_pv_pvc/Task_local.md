# Kubernetes Persistent Volume (PV) & Persistent Volume Claim (PVC) Practice Guide

This guide walks you through creating a Persistent Volume, Persistent Volume Claim, and a Pod that uses the PVC in a Kubernetes cluster using `hostPath`.

---

## Step 1: Create Persistent Volume (PV)

Create a file named `pv.yaml` with the following content:

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: my-pv
spec:
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/home/vagrant/PV"  # This directory must exist on the node where the pod runs

```

## Create Persistent Volume Claim

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 500Mi
```

## Create Pod that Uses the PVC

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  containers:
  - name: my-container
    image: busybox
    command: ["sleep", "3600"]
    volumeMounts:
    - mountPath: "/data"
      name: storage
  volumes:
  - name: storage
    persistentVolumeClaim:
      claimName: my-pvc
```

## commands:

```bash
kubectl apply -f pv.yaml
kubectl apply -f pvc.yaml
kubectl apply -f pod.yaml

## Verify Resource Status
kubectl get pv,pvc
kubectl get pod my-pod
```

## Test Inside the Pod

```bash
kubectl exec -it my-pod -- sh

cd /data
echo "Hello PV" > testfile.txt
cat testfile.txt
```

# Important Note:

Always check the node where the Pod is running to see the actual files in the PV hostPath.
Since the PV uses hostPath, the volume is mounted from the local filesystem of the node where the Pod is scheduled.

* kubectl get pod my-pod -o wide
* vagrant ssh <pod_running_node>
```bash
ls -l /home/vagrant/PV
cat /home/vagrant/PV/testfile.txt
```
