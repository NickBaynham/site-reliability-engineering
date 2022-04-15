# Cluster Setup
Reference: https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/
- If using a VM, create the VM with 2CPU/2GB

## Digital Ocean Example
- Create a droplet 20.04 (LTS) Basic Regular 2 CPU/60 GB/3 TB tx
- Access via the console
- perform the following:

```
apt-get update
apt -y upgrade
apt-get install -y docker.io
docker version
free -m # check if swap is enabled
ufw disable
sudo apt-get update
sudo apt-get install -y apt-transport-https ca-certificates curl
sudo curl -fsSLo /usr/share/keyrings/kubernetes-archive-keyring.gpg https://packages.cloud.google.com/apt/doc/apt-key.gpg
echo "deb [signed-by=/usr/share/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list
sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
kubectl version --client && kubeadm version
sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab
swapoff -a
```
Note: You must repeat the steps above on all of the nodes

## Setting up the control node
```
kubeadm init
mkdir -p $HOME/kube
sudo cp -I /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
kubectl cluster-info
kubectl get nodes
```
