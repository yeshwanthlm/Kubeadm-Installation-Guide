# Kubeadm Installation Guide

This guide outlines the steps needed to set up a Kubernetes cluster using kubeadm.

## Pre-requisites:
* Ubuntu OS (Xenial or later)
* sudo privileges
* AWS Account
* Passion to Learn

### Master & Worker Node: 
Run the following commands on both the master and worker nodes to prepare them for kubeadm.

```bash
 curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"

 curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl.sha256"

 echo "$(cat kubectl.sha256)  kubectl" | sha256sum --check

 sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl

 chmod +x kubectl
 mkdir -p ~/.local/bin
 mv ./kubectl ~/.local/bin/kubectl
 # and then append (or prepend) ~/.local/bin to $PATH

 kubectl version --client

# disable swap
sudo swapoff -a

# Create the .conf file to load the modules at bootup
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF

sudo modprobe overlay
sudo modprobe br_netfilter

# sysctl params required by setup, params persist across reboots
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables  = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward                 = 1
EOF

# Apply sysctl params without reboot
sudo sysctl --system

## Install CRIO Runtime
sudo apt-get update -y
sudo apt-get install -y software-properties-common curl apt-transport-https ca-certificates gpg

sudo curl -fsSL https://pkgs.k8s.io/addons:/cri-o:/prerelease:/main/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/cri-o-apt-keyring.gpg
echo "deb [signed-by=/etc/apt/keyrings/cri-o-apt-keyring.gpg] https://pkgs.k8s.io/addons:/cri-o:/prerelease:/main/deb/ /" | sudo tee /etc/apt/sources.list.d/cri-o.list

sudo apt-get update -y
sudo apt-get install -y cri-o

sudo systemctl daemon-reload
sudo systemctl enable crio --now
sudo systemctl start crio.service

echo "CRI runtime installed successfully"

# Add Kubernetes APT repository and install required packages
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.29/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.29/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list

sudo apt-get update -y
sudo apt-get install -y kubelet="1.29.0-*" kubectl="1.29.0-*" kubeadm="1.29.0-*"
sudo apt-get update -y
sudo apt-get install -y jq

sudo systemctl enable --now kubelet
sudo systemctl start kubelet

```
### Master Node (Only):
a) Initialize the Kubernetes master node.

```bash
 sudo kubeadm config images pull

 sudo kubeadm init

 mkdir -p "$HOME"/.kube
 sudo cp -i /etc/kubernetes/admin.conf "$HOME"/.kube/config
 sudo chown "$(id -u)":"$(id -g)" "$HOME"/.kube/config

 # Network Plugin = calico
 kubectl apply -f https://raw.githubusercontent.com/projectcalico/calico/v3.26.0/manifests/calico.yaml
```
After succesfully running, your Kubernetes control plane will be initialized successfully.

b) Generate a token for worker nodes to join:

```bash
 kubeadm token create --print-join-command
```

c) Expose port 6443 in the Security group for the Worker to connect to Master Node

### Worker Node (Only):

a) Run the following commands on the worker node.

```bash
sudo kubeadm reset pre-flight checks
```

b) Paste the join command you got from the master node and append --v=5 at the end. Make sure either you are working as sudo user or usesudo before the command

Verify if it is working as expected!

```bash
kubectl get nodes
```

Link to the video tutorial: 

Follow our tutorials here: https://www.youtube.com/@amonkincloud/videos \
Follow my personal blog here: https://dev.to/yeshwanthlm/ \
Follow us on Instagram: https://www.instagram.com/amonkincloud/ \
For queries write to us at: amonkincloud@gmail.com 
