---
layout: post
title:  kubernetes
date: 2022-07-02
tags: k8s
---

寫一下 Ubuntu 18.04 (客戶指定版本我也沒法 ......) 安裝 kubernetes (下面用 sudo su - 處理)

```
可參考 https://docs.docker.com/engine/install/ubuntu/
```

```
swapoff -a
sed -e '/swap/ s/^#*/#/' -i /etc/fstab

apt-get update
apt-get install apt-transport-https ca-certificates curl software-properties-common
curl -fsSLo /usr/share/keyrings/kubernetes-archive-keyring.gpg https://packages.cloud.google.com/apt/doc/apt-key.gpg
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | apt-key add -
add-apt-repository \
"deb [arch=amd64] https://download.docker.com/linux/ubuntu \
$(lsb_release -cs) \
stable"

apt-get update
(客戶指定版本我也沒法 ......)
apt-get install docker-ce=18.06.2~ce~3-0~ubuntu
cat > /etc/docker/daemon.json <<EOF
{
  "exec-opts": ["native.cgroupdriver=systemd"],
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "100m"
  },
  "storage-driver": "overlay2"
}
EOF

mkdir -p /etc/systemd/system/docker.service.d

systemctl daemon-reload
systemctl restart docker
```

```
可參考 https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/

```

```
sudo apt-get update
sudo apt-get install -y apt-transport-https curl
sudo curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -
suo cat <<EOF >/etc/apt/sources.list.d/kubernetes.list
deb https://apt.kubernetes.io/ kubernetes-xenial main
EOF
apt-cache madison kubeadm
(客戶指定版本我也沒法 ......)
K_VER="1.18.6-00"
apt install -y kubelet=${K_VER} kubectl=${K_VER} kubeadm=${K_VER}
apt install -y kubelet kubeadm kubectl
kubeadm version
```

```
可參考 https://projectcalico.docs.tigera.io/getting-started/kubernetes/quickstart
```

```
modprobe br_netfilter
echo 1 > /proc/sys/net/bridge/bridge-nf-call-iptables
echo 1 > /proc/sys/net/ipv4/ip_forward

kubeadm init --pod-network-cidr=192.168.0.0/16

mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

(紀錄 像是以下的資訊 要在 node 用)
kubeadm join 172.16.0.1:6443 \
--token 7o06nf.g1uuqnevq557z6cz \
--discovery-token-ca-cert-hash \
sha256:f891a74c5d0ab238eaab27d974f138a3fc3cca0a2ab15d0684570dd67cc66278

kubectl create -f https://projectcalico.docs.tigera.io/manifests/tigera-operator.yaml
kubectl create -f https://projectcalico.docs.tigera.io/manifests/custom-resources.yaml
kubectl apply -f https://docs.projectcalico.org/v3.9/manifests/calico.yaml
kubectl get nodes -o wide

```

```
node 安裝同master (docker kubernetes)只是不必 init
要 kubeadm join ......
如失敗可用 --ignore-preflight-errors=all 重新加入

kubeadm join 172.16.0.1:6443 \
--ignore-preflight-errors=all \
--token 7o06nf.g1uuqnevq557z6cz \
--discovery-token-ca-cert-hash \
sha256:f891a74c5d0ab238eaab27d974f138a3fc3cca0a2ab15d0684570dd67cc66278

mkdir -p $HOME/.kube
scp -i 172.16.0.1:/etc/kubernetes/admin.conf $HOME/.kube/config (要在 masker 複製過來)
chown $(id -u):$(id -g) $HOME/.kube/config
```

在 master
```
kubectl get node
```

安裝helm (master 與 node 都裝可以的)
```
curl https://baltocdn.com/helm/signing.asc | sudo apt-key add –
apt-get install apt-transport-https –yes
echo "deb https://baltocdn.com/helm/stable/debian/ all main" | sudo tee /etc/apt/sources.list.d/helm-stable-debian.list
apt-get update
apt-get install helm
helm repo add stable https://kubernetes-charts.storage.googleapis.com
helm repo add stable https://charts.helm.sh/stable
helm search repo
```

