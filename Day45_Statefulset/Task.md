
# MongoDB StatefulSet Deployment on Kubernetes (Minikube / Local Setup)

## 1. Create Directory Structure for PersistentVolumes

If using minikube or a local Kubernetes setup, create the required directories:

```bash
# SSH into the worker node

# Create directories for PersistentVolumes
sudo mkdir -p /mnt/data/mongodb-{0..4}
sudo chmod 777 /mnt/data/mongodb-{0..4}
exit
```

## 2. Deploy the StatefulSet Components

### Step 2.1: Create a Headless Service

Create a file named `mongodb-service.yaml`:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: mongodb-service
  labels:
    app: mongodb
spec:
  clusterIP: None  # This makes it a headless service
  selector:
    app: mongodb
  ports:
  - port: 27017
    targetPort: 27017
```

Apply the service:

```bash
kubectl apply -f mongodb-service.yaml
```

### Step 2.2: Create StorageClass

Create a file named `mongodb-storageclass.yaml`:

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: mongodb-sc
provisioner: kubernetes.io/no-provisioner  # No dynamic provisioning
volumeBindingMode: WaitForFirstConsumer
reclaimPolicy: Retain
```

Apply the StorageClass:

```bash
kubectl apply -f mongodb-storageclass.yaml
```

### Step 2.3: Create PersistentVolumes

Create a file named `mongodb-pv.yaml`:

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-mongodb-0
  labels:
    type: local
spec:
  storageClassName: mongodb-sc
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/mnt/data/mongodb-0"
  persistentVolumeReclaimPolicy: Retain
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-mongodb-1
  labels:
    type: local
spec:
  storageClassName: mongodb-sc
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/mnt/data/mongodb-1"
  persistentVolumeReclaimPolicy: Retain
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-mongodb-2
  labels:
    type: local
spec:
  storageClassName: mongodb-sc
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/mnt/data/mongodb-2"
  persistentVolumeReclaimPolicy: Retain
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-mongodb-3
  labels:
    type: local
spec:
  storageClassName: mongodb-sc
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/mnt/data/mongodb-3"
  persistentVolumeReclaimPolicy: Retain
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-mongodb-4
  labels:
    type: local
spec:
  storageClassName: mongodb-sc
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/mnt/data/mongodb-4"
  persistentVolumeReclaimPolicy: Retain
```

Apply the PersistentVolumes:

```bash
kubectl apply -f mongodb-pv.yaml
```

### Step 2.4: Create the StatefulSet

Create a file named `mongodb-statefulset.yaml`:

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: mongodb
spec:
  serviceName: "mongodb-service"
  replicas: 3
  selector:
    matchLabels:
      app: mongodb
  template:
    metadata:
      labels:
        app: mongodb
    spec:
      containers:
      - name: mongodb
        image: mongo:4.4
        ports:
        - containerPort: 27017
        volumeMounts:
        - name: data
          mountPath: /data/db
  volumeClaimTemplates:
  - metadata:
      name: data
    spec:
      accessModes: [ "ReadWriteOnce" ]
      storageClassName: "mongodb-sc"
      resources:
        requests:
          storage: 1Gi
```

Apply the StatefulSet:

```bash
kubectl apply -f mongodb-statefulset.yaml
```

## 3. Verification and Testing

### Step 4.1: Verify Pod Deployment

Watch the Pods being created in order:

```bash
kubectl get pods -w
```

You should see something like:

```
NAME        READY   STATUS    RESTARTS   AGE
mongodb-0   1/1     Running   0          2m
mongodb-1   1/1     Running   0          1m30s
mongodb-2   1/1     Running   0          1m
```

### Step 3.2: Verify PersistentVolumeClaims

Check that PVCs are created and bound:

```bash
kubectl get pvc
```

Expected output:

```
NAME              STATUS   VOLUME         CAPACITY   ACCESS MODES   STORAGECLASS   AGE
data-mongodb-0    Bound    pv-mongodb-0   1Gi        RWO            mongodb-sc     3m
data-mongodb-1    Bound    pv-mongodb-1   1Gi        RWO            mongodb-sc     2m30s
data-mongodb-2    Bound    pv-mongodb-2   1Gi        RWO            mongodb-sc     2m
```

### Step 3.3: Test Data Persistence

Connect to the first MongoDB instance:

```bash
kubectl exec -it mongodb-0 -- mongo
```

Create some test data:

```javascript
use testdb
db.users.insert({name: "user1", email: "user1@example.com"})
db.users.insert({name: "user2", email: "user2@example.com"})
db.users.find()
exit
```

Delete the Pod to test persistence:

```bash
kubectl delete pod mongodb-0
```

Wait for the Pod to be recreated:

```bash
kubectl get pods -w
```

Connect to the recreated Pod and verify data persistence:

```bash
kubectl exec -it mongodb-0 -- mongo
```

```javascript
use testdb
db.users.find()
exit
```

You should see that your data is still there.

### Step 3.4: Scale the StatefulSet

Scale to 5 replicas:

```bash
kubectl scale statefulset mongodb --replicas=5
```

Verify the new Pods are created in order:

```bash
kubectl get pods -w
```

You should see:

```
NAME        READY   STATUS    RESTARTS   AGE
mongodb-0   1/1     Running   1          15m
mongodb-1   1/1     Running   0          14m
mongodb-2   1/1     Running   0          13m
mongodb-3   1/1     Running   0          2m
mongodb-4   1/1     Running   0          1m
```

Verify that PVCs for the new Pods are created and bound:

```bash
kubectl get pvc
```

Expected output should now include PVCs for all 5 Pods.

## 4. Clean Up

When you're done with the exercise, clean up all resources:

```bash
kubectl delete statefulset mongodb
kubectl delete svc mongodb-service
kubectl delete pvc --all
kubectl delete pv --all
kubectl delete sc mongodb-sc
```

