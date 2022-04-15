# Cluster Setup
Reference: https://www.cloudsigma.com/how-to-install-and-use-kubernetes-on-ubuntu-20-04/
- If using a VM, create the VM with 2CPU/2GB

## Digital Ocean Example
- Create a droplet 20.04 (LTS) Basic Regular 2 CPU/60 GB/3 TB tx
- Access via the console
- perform the following:

```
apt-get update
apt -y upgrade
apt -y autoremove
apt-get install -y docker.io
docker version
docker images
apt install apt-transport-https curl
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add
echo "deb https://apt.kubernetes.io/ kubernetes-xenial main" >> ~/kubernetes.list
mv ~/kubernetes.list /etc/apt/sources.list.d
apt update
apt -y install kubelet kubeadm kubectl kubernetes-cni
swapoff -a
hostnamectl set-hostname kubernetes-master
lsmod | grep br_netfilter
modprobe br_netfilter
sysctl net.bridge.bridge-nf-call-iptables=1
cat <<EOF | sudo tee /etc/docker/daemon.json
{ "exec-opts": ["native.cgroupdriver=systemd"],
"log-driver": "json-file",
"log-opts":
{ "max-size": "100m" },
"storage-driver": "overlay2"
}
EOF
systemctl enable docker
systemctl daemon-reload
systemctl restart docker
ufw allow 6443
ufw allow 6443/tcp
kubeadm init --pod-network-cidr=10.244.0.0/16

mkdir -p $HOME/.kube
cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
chown $(id -u):$(id -g) $HOME/.kube/config

kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/k8s-manifests/kube-flannel-rbac.yml

kubectl get pods --all-namespaces
kubectl get componentstatus
```
Note: You must repeat the steps above on all of the nodes except for the kubeadm init part

## Test Drive with a Deployment
```
kubectl create deployment nginx --image=nginx
kubectl describe deployment nginx
kubectl create service nodeport nginx --tcp=80:80
kubectl get svc
kubectl get po
curl 137.184.155.28:31348
```
## Remote kubectl
- copy the .kube/config file to the host and install kubectl


