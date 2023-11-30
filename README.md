# INSTALLATION KUBERNETES MULTIMASTER
How to instal kubernetes multimaster Ubuntu 20.04 LTS

## ARCHITECTURE (TOPOLOGI)

![Alt text](image.png)

# Do in all Node

## Login to root
```bash
- sudo -i
```

## set preverse Hostname (Change Hostname VM)
**preserve_hostname** adalah opsi atau pengaturan yang digunakan dalam konfigurasi beberapa distribusi Linux, terutama dalam file konfigurasi **/etc/cloud/cloud.cfg**. Pengaturan ini dapat memiliki nilai true atau false dan mempengaruhi cara hostname sistem dihandle selama proses boot atau provisioning.

Ketika **preserve_hostname** diatur sebagai true, itu berarti sistem akan mempertahankan hostname yang sudah ditetapkan secara manual atau yang diberikan selama proses konfigurasi. Dengan kata lain, meskipun ada konfigurasi atau otomatisasi yang mencoba mengubah hostname selama proses boot, sistem akan mempertahankan nilai hostname yang sudah ada.

Jika preserve_hostname diatur sebagai false, maka sistem dapat mengganti hostname selama proses boot jika ada konfigurasi atau skrip yang mencoba mengatur hostname baru.

- nano /etc/cloud/cloud.cfg<br/>
  preserve_hostname : true

## Setup Hostname 
- hostnamectl set-hostname {hostname}

| Hostname       | Ip            | Description                                |
| :--------   | :-------      | :----------------------------------------- |
| `Lb-master` | `10.10.90.51` | Load balance for kube api-server port 6443 |
| `master-01` | `10.10.90.52` | Controle plane                             |
| `master-02` | `10.10.90.53` | Controle plane                             |
| `master-03` | `10.10.90.54` | Controle plane                             |
| `worker-01` | `10.10.90.55` | Worker                                     |
| `worker-02` | `10.10.90.56` | Worker                                     |
| `worker-03` | `10.10.90.57` | Worker                                     |

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
## comment swap in fstab
- nano /etc/fstab<br/>
  swap bla bla
## set off swap
- swapoff -a 
- mount -a

  
## set layer file system
- modprobe overlay

## set network bridge
- modprobe br_netfilter

## set config layer file system
- tee /etc/modules-load.d/k8s.conf <<EOF<br/>
  overlay<br/>
  br_netfilter<br/>
  EOF

## set config bridge network k8s
- tee /etc/sysctl.d/kubernetes.conf<<EOF<br/>
  net.bridge.bridge-nf-call-ip6tables = 1<br/>
  net.bridge.bridge-nf-call-iptables = 1<br/>
  net.ipv4.ip_forward = 1<br/>
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














