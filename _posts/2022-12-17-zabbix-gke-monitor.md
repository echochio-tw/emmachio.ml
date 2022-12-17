---
layout: post
title: zabbix 監控 GKE 內 K8S
date: 2022-12-17
tags: k8s GKE
---

我這邊環境 Zabbix Server 是 Compute Engine 的獨立主機 

先设定 GCP
防火牆規則詳細資料
加 --> 

目标标记 zabbix
来源范围 10.16.0.0/14 (看 GKE Pod IP 位址範圍 10.16.0.0/14)

TCP 10050-10051
zabbix 加網路標記 zabbix
GKE 叢集pool 加網路標記 zabbix

所有 pod svc 都要能出去到 zabbix 要去所有node下(要去看 pod 內網IP)

```
iptables -A POSTROUTING -d 10.140.0.0/20 -m addrtype ! --dst-type LOCAL -j MASQUERADE -t nat
```

```
helm repo add zabbix-chart-6.2 https://cdn.zabbix.com/zabbix/integrations/kubernetes-helm/6.2/
helm repo list
helm pull zabbix-chart-6.2/zabbix-helm-chrt
tar -xf zabbix-helm-chrt-1.1.1.tgz
cd zabbix-helm-chrt
```
我只更改 values.yaml
```
zabbixProxy.env.ZBX_SERVER_HOST	10.140.0.28
zabbixProxy.env.ZBX_HOSTNAME zabbix-proxy-k8s
```
ZabbixServer地址 指向 GCP 的VM 内网 IP 


```
kubectl create ns zabbix
helm install zabbix . --dependency-update -n zabbix
```
查看K8S Zabbix Pod
```
kubectl get pods -n zabbix
```

获取API接口访问Token,后面配置Zabbix需要使用到
```
kubectl get secret zabbix-service-account -n zabbix -o jsonpath={.data.token} | base64 -d
```

登录Zabbix页面创建Zabbix Proxy
```
Administrator --> Proxies --> create proxy
Proxy name：zabbix-proxy-k8s
Proxy mode： Active
其他不填
```

察看log
```
kubectl logs -n zabbix $(kubectl get pod -n zabbix |grep zabbix-proxy|awk {'print $1'})
```
一直到
```
159:20221217:034628.189 received configuration data from server at "10.140.0.28", datalen 4689
```

如果不行就重启 zabbix-proxy 如不会好就要 debug 网路了
```
kubectl rollout restart deployment zabbix-proxy -n zabbix
``` 


创建k8s-node主机，用于自动发现K8S节点主机
```
Host name:k8s-nodes
Templates: 选择Template group 中 Templates 下的 Kubernetes nodes by HTTP 模板，用于自动发现K8S节点主机
Host groups: K8S Server (如是新的会出现 new 字眼）
Monitored by proxy: zabbix-proxy-k8s

Macros 要加
{$KUBE.API.ENDPOINT.URL} : https://35.189.122.11/api [去GKE 看那 -> 叢集 端點 ]
{$KUBE.API.TOKEN}: XXXXXXXX [上面获取到的token]
{$KUBE.NODES.ENDPOINT.NAME}： zabbix-zabbix-helm-chrt-agent [通过kubectl get ep -n zabbix 获取到]
```

创建k8s-cluster主机，用于自动发现服务组件
```
Host name: k8s-cluster
Templates: 选择Template group 中 Templates 下的Kubernetes cluster state by HTTP 模板，用于自动发现K8S节点主机
Host groups： K8S Server
Monitored by proxy: zabbix-proxy-k8s

Macros 要加
{$KUBE.API.HOST}: 35.189.122.11 [去GKE 看那 -> 叢集 端點 ]
{$KUBE.API.PORT}：443
{$KUBE.API.TOKEN}： XXXXX [上面获取到的token]
{$KUBE.API.URL} : https://35.189.122.11
{$KUBE.API_SERVER.PORT}：443
{$KUBE.API_SERVER.SCHEME}：https
{$KUBE.CONTROLLER_MANAGER.PORT}：10252
{$KUBE.CONTROLLER_MANAGER.SCHEME}：http
{$KUBE.KUBELET.PORT}：10250
{$KUBE.KUBELET.SCHEME}：https
{$KUBE.SCHEDULER.PORT}：10251
{$KUBE.SCHEDULER.SCHEME}：http
{$KUBE.STATE.ENDPOINT.NAME}：zabbix-kube-state-metrics 【通过kubectl get ep -n zabbix 获取到】
```
主要參考 : https://www.fxkjnj.com/3969/

雖然裝起來還不如自己寫 shell 如ssh到 root 執行 kubectl top nodes 抓資料到 zabbix 寫成 python 也可

當然那個 root shell 要裝 gcloud
```
sshpass -p 'password' ssh root@127.0.0.1 -oStrictHostKeyChecking=no 'kubectl top nodes'
sshpass -p 'password' ssh root@127.0.0.1 -oStrictHostKeyChecking=no 'kubectl top pod -n zabbix'
```
