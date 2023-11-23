##### INSTALLATION KUBERNETES MULTIMASTER #####
How to instal kubernetes multimaster Ubuntu 20.04 LTS

## Login to root
- sudo -i

## set preverse Hostname
- nano /etc/cloud/cloud.cfg<br/>
  preserve_hostname : true

## Setup Hostname 
- hostnamectl set-hostname {hostname}

## logout and login again
- exit
- ssh 

## Set time
- timedatectl set-timezone Asia/Jakarta

## set Domain Local
- nano /etc/hosts/<br/>
  10.1.x.x {hostname-local}<br/>
  10.1.x.x {hostname-local}<br/>
  10.1.x.x {hostname-local}<br/>
  10.1.x.x {hostname-local}<br/>

## update repository
- apt update

## install https Transport
- apt install curl apt-transport-https -y

## add api-key official k8s from google
- curl -fsSL  https://packages.cloud.google.com/apt/doc/apt-key.gpg|sudo gpg --dearmor -o /etc/apt/trusted.gpg.d/k8s.gpg
- curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -

## add repository k8s to source llist ubuntu
- echo "deb https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list

## update repository
- apt update

## instal kubectl kubeadm kubelet spceific version (v1.25)
- apt install -y kubeadm=1.25.0-00 kubelet=1.25.0-00 kubectl=1.25.0-00 

## Hold all service 
- apt-mark hold kubelet kubeadm kubectl

### System linux configuration ###
## set off swap
- swapoff -a 
- mount -a
## comment swap in fstab
- nano /etc/fstab
  swap bla bla
  
## set layer file system
- modprobe overlay

## set network bridge
- modprobe br_netfilter

## set config layer file system
- tee /etc/modules-load.d/k8s.conf <<EOF
  overlay
  br_netfilter
  EOF

## set config bridge network k8s
- tee /etc/sysctl.d/kubernetes.conf<<EOF
  net.bridge.bridge-nf-call-ip6tables = 1
  net.bridge.bridge-nf-call-iptables = 1
  net.ipv4.ip_forward = 1
  EOF

## apply system bridge
- sysctl --system

### Install Containerd ###
## add denpedencies and install ca certificate
- apt install -y gnupg2 software-properties-common ca-certificates

## add official repository
- curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
- add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"

## install containerd
- apt install -y containerd.io

## make directory config
- mkdir -p /etc/containerd

## add config default containerd
- containerd config default > /etc/containerd/config.toml

## restart and enable service containerd
- systemctl restart containerd
- systemctl enable containerd
- systemctl status containerd


### Initial Master ####

# pull kubernetes image
- kubeadm config images pull

# check preflight
- kubeadm init phase preflight

# Init Multi Master 
- kubeadm init --control-plane-endpoint="{ip LB master}:6443" --upload-certs --pod-network-cidr=192.168.0.0/16 --service-cidr=172.16.0.0/16
# add kubeconfig admin 
- mkdir -p $HOME/.kube
- sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
- sudo chown $(id -u):$(id -g) $HOME/.kube/config














