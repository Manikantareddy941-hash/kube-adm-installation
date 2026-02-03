# Kubernetes Cluster Setup Using kubeadm on EC2

This project documents how to set up a multi-node Kubernetes cluster using kubeadm on AWS EC2 instances.

## Tech Stack

- Cloud: Amazon EC2
- Container Runtime: containerd
- Orchestration: Kubernetes (kubeadm)
- Network Plugin: Calico
- OS: Ubuntu 20.04 / 22.04

---

## Architecture

Master Node (Control Plane)
- Initializes cluster
- Manages worker nodes

Worker Nodes (2)
- Run application workloads

Total Nodes: 3

---

## Prerequisites

- 3 EC2 instances with Ubuntu
- SSH access to all nodes
- Open security group ports:
  - 6443 (Kubernetes API)
  - 10250, 10251, 10252
  - All node-to-node traffic allowed

---

## Installation Steps

### On All Nodes

Disable swap:

sudo swapoff -a

Install containerd:

sudo apt update  
sudo apt install -y containerd  

Configure:

sudo mkdir -p /etc/containerd  
containerd config default | sudo tee /etc/containerd/config.toml  

Set:

SystemdCgroup = true  

Restart:

sudo systemctl restart containerd  
sudo systemctl enable containerd  

---

### Install Kubernetes Tools (All Nodes)

sudo apt update  
sudo apt install -y apt-transport-https ca-certificates curl  

curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.29/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg  

echo "deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.29/deb/ /" | sudo tee /etc/apt/sources.list.d/kubernetes.list  

sudo apt update  
sudo apt install -y kubelet kubeadm kubectl  
sudo apt-mark hold kubelet kubeadm kubectl  

---

### On Master Node

Initialize cluster:

sudo kubeadm init --pod-network-cidr=192.168.0.0/16  

Configure kubectl:

mkdir -p $HOME/.kube  
sudo cp /etc/kubernetes/admin.conf $HOME/.kube/config  
sudo chown $(id -u):$(id -g) $HOME/.kube/config  

Install Calico network:

kubectl apply -f https://raw.githubusercontent.com/projectcalico/calico/v3.27.0/manifests/calico.yaml  

---

### On Worker Nodes

Join cluster using command shown after kubeadm init:

sudo kubeadm join <MASTER-IP>:6443 --token <token> --discovery-token-ca-cert-hash sha256:<hash>

---

## Verification

On master:

kubectl get nodes

All nodes should show:

Ready

---

## Result

A fully working multi-node Kubernetes cluster on AWS EC2 using kubeadm.

---

## Author

Mani  
DevOps & Cloud Engineer in training
