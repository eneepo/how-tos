---
title: "Kubernetes on-premise setup"
linkTitle: "On-premise"
description: "Setting Up an On-premise Kubernetes Cluster from Scratch"
date: 2023-12-10T13:55:29+01:00
categories: [CKA]
tags: [kubeadm, containerd, calico]
---

## Container Runtimes
### Installing containerd

To facilitate the installation of containerd, start by configuring the necessary kernel modules. Create a new file at `/etc/modules-load.d/containerd.conf` and include the following lines:

```conf
# /etc/modules-load.d/containerd.conf
overlay
br_netfilter
```

Activate the kernel modules immediately without rebooting the system using the following commands:

```bash
sudo modprobe overlay
sudo modprobe br_netfilter
```

Set specific sysctl settings by creating a configuration file at `/etc/sysctl.d/99-kubernetes-cri.conf` with the following content:

```conf
# /etc/sysctl.d/99-kubernetes-cri.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
```

Save and close the file, then reload sysctl:

```bash
sudo sysctl --system
```

Proceed with installing containerd:

```bash
sudo apt update && sudo apt install -y containerd
```

Create a dedicated directory for containerd:

```bash
sudo mkdir -p /etc/containerd
```

Generate the configuration file:

```bash
sudo containerd config default > /etc/containerd/config.toml
```

Restart the containerd service:

```bash
sudo systemctl restart containerd
```

Enable containerd to start automatically upon system boot:

```bash
sudo systemctl enable containerd
```

## Installing Kubernetes

### Prerequisites
Before installing Kubernetes, ensure that swap is disabled. Open the fstab file for editing:

```bash
sudo vi /etc/fstab
```

Comment out the swap line:

```conf
# /swap.img      none    swap    sw      0       0
```

Save and close the file. Disable swap without rebooting:

```bash
sudo swapoff -a
```

Update the package manager and install necessary prerequisites:

```bash
sudo apt update && sudo apt install -y apt-transport-https curl
```

Add the GPG key for the Kubernetes repository:

```bash
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
```

Include the Kubernetes repository:

```bash
echo "deb https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list
```

Update apt and install the required Kubernetes components:

```bash
apt update && sudo apt -y install kubelet=1.27.6 kubeadm=1.27.6 kubectl=1.27.6
```

Hold the installed versions to prevent unintended upgrades:

```bash
apt-mark hold kubelet kubeadm kubectl
```

```bash
kubeadm init --pod-network-cidr 192.168.0.0/16 --kubernetes-version=1.27.6

Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

Alternatively, if you are the root user, you can run:

  export KUBECONFIG=/etc/kubernetes/admin.conf

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join 10.0.1.101:6443 --token 693re4.sb1ihswxbsrn0avl \
        --discovery-token-ca-cert-hash sha256:14add48dd42627c31fad8e21ef9eadbb50c3b692a67af8c936dd642a8ac2eefa
```

## Networking

### Calico
```bash
kubectl apply -f https://docs.tigera.io/calico/latest/manifests/calico.yaml
```

```bash
watch kubectl get nodes
```

```bash
kubeadm token create --print-join-command
kubeadm join 10.0.1.101:6443 --token 6da0pn.n4zb9emytmevftoc --discovery-token-ca-cert-hash sha256:14add48dd42627c31fad8e21ef9eadbb50c3b692a67af8c936dd642a8ac2eefa
```

## Ansible
```bash
ansible -i inventory.ini kubernetes -m ping
ansible-playbook -i inventory.ini install_kubernetes.yaml -kK
```