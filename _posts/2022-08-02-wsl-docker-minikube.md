---
layout: post
title: wsl + systemd + docker + minikube
date: 2022-08-02
tags: wsl
---


設定 wsl v2
wsl --set-default-version 2 

看已安裝的
wsl --list -v

打掉重練
wsl --unregister Distrod


設定記憶體及CPU核心與無SWAP (minikube 可有 SWAP)

notepad %UserProfile%\.wslconfig
```
[wsl2]
memory=4GB
processors=8
swap=2GB
```

用這個安裝 https://linuxcontainers.org/
下載
```
https://github.com/nullpo-head/wsl-distrod/releases/latest/download/distrod_wsl_launcher-x86_64.zip
```

解壓安裝 .... 用內定的 ubuntu focal 

進去OS後 (這是記錄不必做)
```
sudo su -
apt install curl -y
curl -L -O "https://raw.githubusercontent.com/nullpo-head/wsl-distrod/main/install.sh"
chmod +x install.sh
sudo ./install.sh install
/opt/distrod/bin/distrod enable
```

如要windows 啟動就跑 systemd (放入排程... 需windows 密碼)  (這是記錄不必做)
``` 
/opt/distrod/bin/distrod enable --start-on-windows-boot

Distrod] Distrod has been enabled. Now your shell will start under systemd.
[Distrod] Enabling atuomatic startup of Distrod. UAC dialog will appear because scheduling
a task requires the admin privilege. Please hit enter to proceed.

Enabling autostart has succeeded.
[Distrod] Distrod will now start automatically on Windows startup.
```

掛入登入就是root
及環境設定
```
sudo su -

cat >> /root/.profile <<EOF
[ -f /etc/resolv.conf ]  || echo nameserver 1.1.1.1 > /etc/resolv.conf
EOF

apt update
apt upgrade

printf "\n[user]\ndefault = root\n" | sudo tee -a /etc/wsl.conf
echo -e "[network]\ngenerateResolvConf = false" | sudo tee -a /etc/wsl.conf
cat /etc/wsl.conf

sudo apt install --no-install-recommends apt-transport-https ca-certificates curl gnupg2 -y
exit
logout
```
在CMD
```
wsl --shutdown
wsl
```

```
apt-get install -y containerd.io
mkdir -p /etc/containerd
containerd config default > /etc/containerd/config.toml
systemctl restart containerd
systemctl enable containerd
```

用官方方法安裝 docker
```
apt-get remove docker docker-engine docker.io containerd runc
apt-get -y install ca-certificates curl gnupg lsb-release
mkdir -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
apt-get update
apt-get install  containerd.io docker-ce docker-ce-cli docker-compose-plugin
systemctl restart containerd
systemctl enable containerd
systemctl restart docker
systemctl enable docker
```

確認無 swap (minikube 可有 SWAP)
```
free -m
              total        used        free      shared  buff/cache   available
Mem:           7954         349        6504           0        1101        7365
Swap:             0           0           0
```

安装 kubectl
```
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
kubectl version --client
```

安装 Minikube
```
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
install minikube-linux-amd64 /usr/local/bin/minikube
minikube start --force
```
編輯 minikube.service
```
systemctl edit --force --full minikube.service
```
```
[Unit]
Description=minikube
Wants=network-online.target
After=network.target local-fs.target containerd.service docker.service
Requires=containerd.service docker.service

[Service]
Type=oneshot
RemainAfterExit=yes
WorkingDirectory=/root
ExecStartPre=/usr/local/bin/minikube stop
ExecStart=/usr/local/bin/minikube start --force
ExecStop=/usr/local/bin/minikube stop
User=root
Group=root

[Install]
WantedBy=multi-user.target
```

設定 啟動minikube 
```
systemctl enable minikube.service
```

```
minikube node list
minikube dashboard --url &
# 让其它 IP 可以访问
kubectl proxy --port=8888 --address='0.0.0.0' --accept-hosts='^.*'
```

會出現向這樣的
```
http://127.0.0.1:41789/api/v1/namespaces/kubernetes-dashboard/services/http:kubernetes-dashboard:/proxy/
```

```
kubectl create deployment nginx --image=nginx
kubectl expose deployment nginx --port=80 --type=NodePort
# 获取访问地址
minikube service --url nginx
```

```
也可以通过 kubectl proxy 拼接 url 访问
https://kubernetes.io/zh/docs/tasks/access-application-cluster/access-cluster/#manually-constructing-apiserver-proxy-urls
http://10.74.2.71:8888/api/v1/namespaces/default/services/nginx:80/proxy/
```

负载均衡访问，Minikube 网络
```
minikube tunnel --cleanup=true &

# 重新部署
kubectl delete deployment nginx
kubectl delete service nginx
kubectl create deployment nginx --image=nginx
kubectl expose deployment nginx --port=80 --type=LoadBalancer
# 查看外部地址

kubectl port-forward svc/nginx 8080:80 --address='0.0.0.0'
```


一些指令
```
kubectl get node
kubectl get pods -A
kubectl get svc -A
kubectl describe pod coredns-6d4b75cb6d-4p946 -n kube-system

```

```
 curl -O https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3
 chmod +x get-helm-3
 ./get-helm-3
 helm repo add projectcalico https://projectcalico.docs.tigera.io/charts
```
```
https://raw.githubusercontent.com/karthequian/docker-helloworld/master/deployment.yml
minikube service --url helloworld

```
自己的 docker file
```
git clone https://github.com/echochio-tw/flask-appbuilder-docker.git
docker build -t flask-appbuilder .
docker images
```

自建 registry
```
minikube addons enable registry
docker run --rm -it --network=host alpine ash -c "apk add socat && socat TCP-LISTEN:5000,reuseaddr,fork TCP:$(minikube ip):5000" &
```

放入 registry
```
docker tag flask-appbuilder localhost:5000/flask-appbuilder
docker push localhost:5000/flask-appbuilder
```

deployment 出去
```
kubectl create deployment flask-appbuilder --image=localhost:5000/flask-appbuilder
kubectl expose deployment flask-appbuilder --type=NodePort --port=80
kubectl get service flask-appbuilder
```

minikube service lask-appbuilder --url

輸出類似於：
```
http://127.0.0.1:31637
```

瀏覽
```
http://127.0.0.1:31637
```
