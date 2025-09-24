# Multi-Node Kubernetes Cluster Setup Using Vagrant & Kubeadm

This guide provides step-by-step instructions to create a **multi-node Kubernetes cluster** using **Vagrant** for VMs and **Kubeadm** for Kubernetes setup.

---

## Table of Contents

1. [Prerequisites](#prerequisites)
2. [Vagrant VM Setup](#vagrant-vm-setup)
3. [Install Containerd](#install-containerd)
4. [Install Kubernetes Components](#install-kubernetes-components)
5. [Disable Swap & Enable Kernel Modules](#disable-swap--enable-kernel-modules)
6. [Initialize Master Node](#initialize-master-node)
7. [Install Flannel Network](#install-flannel-network)
8. [Join Worker Nodes](#join-worker-nodes)
9. [Verify Cluster](#verify-cluster)

---

## Prerequisites

- Ubuntu 18.04 or higher (for VM base box)
- At least **2GB RAM** and **2 CPU cores** per VM
- Network connectivity between nodes
- Root access to all VMs
- Installed packages: `vagrant`, `vagrant-libvirt`, `libvirt`, `qemu-kvm`

---

## Vagrant VM Setup

1. **Create a project directory:**

``` bash
mkdir ~/k8s-vagrant
cd ~/k8s-vagrant

```

2. **Create Vagrantfile:**

```ruby
Vagrant.configure("2") do |config|
  config.vm.box = "generic/ubuntu2204"

  # Master Node
  config.vm.define "master" do |master|
    master.vm.hostname = "master"
    master.vm.network "private_network", ip: "192.168.56.10"
    master.vm.provider :libvirt do |lv|
      lv.memory = 2048
      lv.cpus = 2
    end
  end

  # Worker Nodes
  (1..2).each do |i|
    config.vm.define "worker#{i}" do |worker|
      worker.vm.hostname = "worker#{i}"
      worker.vm.network "private_network", ip: "192.168.56.1#{i+0}"
      worker.vm.provider :libvirt do |lv|
        lv.memory = 1024
        lv.cpus = 1
      end
    end
  end
end
```

3. **Bring up the VMs:**

``` bash
vagrant up --provider=libvirt

 ```

4. **Access VMs via SSH:**

``` bash
vagrant ssh master
vagrant ssh worker1
vagrant ssh worker2

 ```

---


## Step 1: Update System and Install Dependencies

Update the system's package list and install the necessary dependencies:

```bash
sudo apt-get update
sudo apt install apt-transport-https curl -y
```


## Install Containerd

1. Add Docker repository and install containerd:

``` bash
sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
sudo apt-get install containerd.io -y
 ```

2. Create configuration file and enable SystemdCgroup:

``` bash
sudo mkdir -p /etc/containerd
sudo containerd config default | sudo tee /etc/containerd/config.toml
sudo sed -i -e 's/SystemdCgroup = false/SystemdCgroup = true/g' /etc/containerd/config.toml
sudo systemctl restart containerd
 ```

---

## Install Kubernetes Components

``` bash
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.30/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.30/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
sudo systemctl enable --now kubelet
 ```

---

## Disable Swap & Enable Kernel Modules

``` bash
sudo swapoff -a
sudo nano /etc/fstab      # Remove swap entries if present
sudo modprobe br_netfilter
sudo sysctl -w net.ipv4.ip_forward=1
 ```

---

## Initialize Master Node (Only in master)




 ```bash 
 sudo kubeadm init \
  --apiserver-advertise-address=192.168.56.10 \
  --pod-network-cidr=10.244.0.0/16
```

Set up kubeconfig:

``` bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
 ```

---

## Install Flannel Network

``` bash
kubectl apply -f https://github.com/flannel-io/flannel/releases/latest/download/kube-flannel.yml
 ```

---

## Join Worker Nodes

Run the command printed by `kubeadm init` on each worker node:

``` bash
sudo kubeadm join <MASTER_IP>:6443 --token <TOKEN>     --discovery-token-ca-cert-hash sha256:<HASH>
 ```

Example:

``` bash
sudo kubeadm join 192.168.56.10:6443 --token 0r08uj.ykoz34j1ik94x9vk     --discovery-token-ca-cert-hash sha256:2681b68d9a849df78086d325eccaa0b49f36b10a2491fc63bad31f9e105b317c
 ```

---

## Verify Cluster

``` bash
kubectl get nodes
kubectl get pods --all-namespaces
 ```

---

## Notes

- Run all `kubeadm` commands as **root** or use `sudo`.
- Ensure Vagrant user is part of `libvirt` group:

``` bash
sudo usermod -aG libvirt technonext
newgrp libvirt
 ```

-  warnings can be ignored.
- For troubleshooting:

``` bash
virsh list --all
vagrant status
```





# KVM Setup Guide for Vagrant + Libvirt

If you encounter the error:



follow these steps to fix it.

---

## 2. Check if KVM is available in Linux

After reboot, check if your CPU supports virtualization:

```bash
egrep -c '(vmx|svm)' /proc/cpuinfo

sudo modprobe kvm
sudo modprobe kvm_intel  
ls -alh /dev/kvm
sudo usermod -aG kvm $USER
newgrp kvm
vagrant up



