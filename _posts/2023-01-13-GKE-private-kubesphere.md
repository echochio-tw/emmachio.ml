---
layout: post
title: GKE-private install kubesphere
date: 2023-01-13
tags: GCP
---
記錄一下 GKE-private 踩到的坑

kubesphere 安裝就依照官方網站安裝
```
kubesphere/ks-installer/releases/download/v3.3.1/kubesphere-installer.yaml
kubectl apply -f https://github.com/kubesphere/ks-installer/releases/download/v3.3.1/cluster-configuration.yaml
```
看安裝過程 log
```
kubectl logs -n kubesphere-system $(kubectl get pod -n kubesphere-system -l 'app in (ks-install, ks-installer)' -o jsonpath='{.items[0].metadata.name}') -f
```

最後可看到
```
Console: http://10.19.0.2:30880
Account: admin
Password: P@88w0rd
NOTES：
  1. After you log into the console, please check the
     monitoring status of service components in
     "Cluster Management". If any service is not
     ready, please wait patiently until all components
     are up and running.
  2. Please change the default password after login.
#####################################################
https://kubesphere.io             2023-01-12 04:25:27
#####################################################
```

改 LoadBalancer 馬上使用(當然也可用 ingress ... 請自行 google)
```
kubectl patch svc ks-console -n kubesphere-system -p '{"spec": {"type": "LoadBalancer"}}'
kubectl get service ks-console -n kubesphere-system
```
移除就用 kubesphere-delete.sh
```
wget https://raw.githubusercontent.com/kubesphere/ks-installer/master/scripts/kubesphere-delete.sh
chmod +x kubesphere-delete.sh
./kubesphere-delete.sh
```

改密碼 .....  GKE-private 環境在這邊會出錯 沒法改
```
kubectl patch users admin -p '{"spec":{"password":"eUXeP12345"}}' --type='merge' && kubectl annotate users admin iam.kubesphere.io/password-encrypted-
```
google 到是這個源因
```
https://github.com/open-telemetry/opentelemetry-operator/issues/1009
```

依照文章Firewall rule 將 gke-cluster-private-XXXXXXXX-master 的 Protocols and Ports 開這樣 就可改密碼了
```
tcp:10250
tcp:443
tcp:8443
tcp:9443
```

如果 nginx ingress 有問題也是這問題

调用webhook "validate.nginx.ingress.kubernetes.io "失败：在Kubernetes中应用ingress时出错。

我都是這樣處理沒去開 master PORT
```
kubectl delete -A ValidatingWebhookConfiguration nginx-ingress-ingress-nginx-admission
```

配合 Prometheus 有些問題 還沒試 應該可以成功
```
https://www.bookstack.cn/read/kubesphere-3.0-zh/17fb7f3300f73eea.md

https://kubesphere.io/zh/docs/v3.3/faq/observability/byop/

```
