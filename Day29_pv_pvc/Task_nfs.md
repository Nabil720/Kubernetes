# Kubernetes NFS Persistent Volume (PV) and PVC Guide

This guide walks you through setting up an NFS server using Vagrant and using it in a Kubernetes cluster for dynamic PV/PVC management.

---

## Part 1: Create an NFS Server with Vagrant

## tep 1: Vagrantfile for NFS Server

Create a folder:

```bash

mkdir -p ~/Documents/Kubernetes/ClusterSetup/nfs-server
cd ~/Documents/Kubernetes/ClusterSetup/nfs-server
nano Vagrantfile

# Vagrantfile

Vagrant.configure("2") do |config|
  config.vm.box = "ubuntu/focal64"
  config.vm.hostname = "nfs-server"
  config.vm.network "private_network", ip: "192.168.56.50"

  config.vm.provider "virtualbox" do |vb|
    vb.memory = "1024"
  end

  config.vm.provision "shell", inline: <<-SHELL
    apt-get update
    apt-get install -y nfs-kernel-server
    mkdir -p /srv/nfs/kubedata
    chown nobody:nogroup /srv/nfs/kubedata
    chmod 777 /srv/nfs/kubedata
    echo "/srv/nfs/kubedata *(rw,sync,no_subtree_check,no_root_squash)" >> /etc/exports
    exportfs -ra
    systemctl restart nfs-kernel-server
  SHELL
end

# Start the NFS VM

vagrant up

```

## ⚠️ Install nfs-common on All Worker Nodes

```bash
vagrant ssh worker1
sudo apt install -y nfs-common
exit

vagrant ssh worker2
sudo apt install -y nfs-common
exit

```

## Create NFS-Based PV, PVC, and Pod

```bash

# pv
apiVersion: v1
kind: PersistentVolume
metadata:
  name: nfs-pv
spec:
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteMany
  nfs:
    server: 192.168.56.50
    path: "/srv/nfs/kubedata"
  persistentVolumeReclaimPolicy: Retain

# pvc 
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: nfs-pvc
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 500Mi

# pod
apiVersion: v1
kind: Pod
metadata:
  name: nfs-pod
spec:
  containers:
  - name: nfs-container
    image: busybox
    command: ["sleep", "3600"]
    volumeMounts:
    - mountPath: "/mnt"
      name: nfs-storage
  volumes:
  - name: nfs-storage
    persistentVolumeClaim:
      claimName: nfs-pvc

```
## Apply Resources

```bash
kubectl get pod nfs-pod
kubectl describe pod nfs-pod

# Enter Pod and Write Data

kubectl exec -it nfs-pod -- sh
cd /mnt
echo "Hello from NFS" > test.txt
cat test.txt
exit

```
Now we can also see the data in the nfs server