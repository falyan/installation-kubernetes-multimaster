# INSTALASI KUBERNETES MULTI MASTER
instal kubernetes multimaster Ubuntu 20.04 LTS

## ARSITEKTUR (TOPOLOGI)

![Alt text](image.png)

## set preverse hostname (do all node)
**preserve_hostname** adalah opsi atau pengaturan yang digunakan dalam konfigurasi beberapa distribusi Linux, terutama dalam file konfigurasi **/etc/cloud/cloud.cfg**. Pengaturan ini dapat memiliki nilai true atau false dan mempengaruhi cara hostname sistem dihandle selama proses boot atau provisioning.

Ketika **preserve_hostname** diatur sebagai true, itu berarti sistem akan mempertahankan hostname yang sudah ditetapkan secara manual atau yang diberikan selama proses konfigurasi. Dengan kata lain, meskipun ada konfigurasi atau otomatisasi yang mencoba mengubah hostname selama proses boot, sistem akan mempertahankan nilai hostname yang sudah ada.

Jika preserve_hostname diatur sebagai false, maka sistem dapat mengganti hostname selama proses boot jika ada konfigurasi atau skrip yang mencoba mengatur hostname baru.
```bash
nano /etc/cloud/cloud.cfg
```
preserve_hostname : true

## Set the hostname and adjust it to the required hostname (do all node)
```bash
hostnamectl set-hostname {hostname}
```
| HOSTNAME    | IP            | KETERANGAN                               |
| :--------   | :-------      | :----------------------------------------- |
| `lb-master` | `10.10.90.51` | Load balance for kube api-server port 6443 |
| `master-01` | `10.10.90.52` | Controle plane                             |
| `master-02` | `10.10.90.53` | Controle plane                             |
| `master-03` | `10.10.90.54` | Controle plane                             |
| `worker-01` | `10.10.90.55` | Worker                                     |
| `worker-02` | `10.10.90.56` | Worker                                     |
| `worker-03` | `10.10.90.57` | Worker                                     |

## Set time (do all node)
```bash
timedatectl set-timezone Asia/Jakarta
```
## set Domain Local (do all node)
```bash
nano /etc/hosts
```
  10.10.90.51 lb-master<br/>
  10.10.90.52 master-01<br/>
  10.10.90.53 master-02<br/>
  10.10.90.54 master-03<br/>
  10.10.90.55 worker-01<br/>
  10.10.90.56 worker-02<br/>
  10.10.90.57 worker-03<br/>

# K8s Installation (Do all Master and worker except Load Balancer)

## update repository
```bash
apt update
```
## install https Transport
```bash
apt install curl apt-transport-https -y
```
## add api-key official k8s from google
```bash
curl -fsSL  https://packages.cloud.google.com/apt/doc/apt-key.gpg|sudo gpg --dearmor -o /etc/apt/trusted.gpg.d/k8s.gpg
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
```
## add repository k8s to source list ubuntu
```bash
echo "deb https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list
```
## update repository just added
```bash
apt update
```
## instal kubectl kubeadm kubelet spceific version (v1.25)
```bash
apt install -y kubeadm=1.25.0-00 kubelet=1.25.0-00 kubectl=1.25.0-00 
```
## Hold all service to keep version 
```bash
apt-mark hold kubelet kubeadm kubectl
```
### System linux configuration ###
## comment swap in fstab
```bash
nano /etc/fstab
``` 
comment swap.img

```bash
# /etc/fstab: static file system information.
#
# Use 'blkid' to print the universally unique identifier for a
# device; this may be used with UUID= as a more robust way to name devices
# that works even if disks are added and removed. See fstab(5).
#
# <file system> <mount point>   <type>  <options>       <dump>  <pass>
# / was on /dev/ubuntu-vg/ubuntu-lv during curtin installation
/dev/disk/by-id/dm-uuid-LVM-97dj8w1JiawUrjPxBSMmOPPQAcxCp0Unxd76ijKb4JTNISA31NTokVogkj9nr9uZ / ext4 defaults 0 1
# /boot was on /dev/sda2 during curtin installation
/dev/disk/by-uuid/20b8e0ca-52b3-44dd-bcec-26609d5746d1 /boot ext4 defaults 0 1
#/swap.img      none    swap    sw      0       0

## set off swap
```
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














