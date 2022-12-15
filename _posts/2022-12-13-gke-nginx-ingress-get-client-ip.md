---
layout: post
title: GKE nginx ingress 取 client IP
date: 2022-12-13
tags: k8s GKE
---

我的專案是 line-370201

域名是 line.net

用 cloudflare proxy

以下是在 google cloud shell 做的

連GKE 然後將 image 推到 GKE
```
gcloud container clusters get-credentials line --zone asia-east1-a --project line-370201
docker pull containous/whoami
docker tag containous/whoami gcr.io/line-370201/whomai
docker push gcr.io/line-370201/whomai
```

將image 放到 deployment 與 Service
```
kubectl create deployment whoami --image=gcr.io/line-370201/whomai:latest
kubectl expose deployment whoami --port=80 --target-port=80
```

設定 域名到 ingress 
```
cat <<EOF | kubectl apply -f -
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ingress-whoami
  annotations:
    kubernetes.io/ingress.class: "nginx"
    nginx.ingress.kubernetes.io/ssl-redirect: "false"
spec:
  rules:
  - host: "k8s-ingress.line.net"
    http:
      paths:
      - pathType: Prefix
        path: "/whoami"
        backend:
          service:
            name: whoami
            port:
              number: 80
EOF
```

取 k8s-ingress.line.net IP
```
kubectl get ingress ingress-whoami
```

顯示:
```
NAME             CLASS    HOSTS                      ADDRESS          PORTS   AGE
ingress-whoami   <none>   k8s-ingress.line.net   35.187.141.16   80      25m
```

設定 cloudflare DNS k8s-ingress.line.net 對應 35.187.141.16 開 proxy

cutl 一下
```
curl https://k8s-ingress.line.net/whoami
```
顯示:
```
Hostname: whoami-78fbc58646-t7pq8
IP: 127.0.0.1
IP: 10.12.5.141
RemoteAddr: 10.12.2.3:55058
GET /whoami HTTP/1.1
Host: k8s-ingress.line.net
User-Agent: curl/7.68.0
Accept: */*
Accept-Encoding: gzip
Cdn-Loop: cloudflare
Cf-Connecting-Ip: 118.231.144.191
Cf-Ipcountry: TW
Cf-Ray: 778ec91ada0980ad-NRT
Cf-Visitor: {"scheme":"https"}
X-Forwarded-For: 10.140.15.192
X-Forwarded-Host: k8s-ingress.line.net
X-Forwarded-Port: 80
X-Forwarded-Proto: http
X-Forwarded-Scheme: http
X-Original-Forwarded-For: 118.231.144.191
X-Real-Ip: 10.140.15.192
X-Request-Id: c04b8b9104e4985e10d801ee88ab33db
X-Scheme: http
```
那個 X-Original-Forwarded-For 就是我的IP

如果源沒有 443 可用 ingress compute-full-forwarded-for 在 control 設定：
```
apiVersion: v1
kind: ConfigMap
......
data:
  compute-full-forwarded-for: "true"
  use-forwarded-headers: "true"
```

參考：
https://cloud.google.com/container-registry/docs/pushing-and-pulling

https://cloud.google.com/community/tutorials/nginx-ingress-gke

https://blog.horus-k.com/2021/06/17/k8s%E6%B5%81%E9%87%8F%E7%AD%96%E7%95%A5%E4%B8%8Eingress%E8%8E%B7%E5%8F%96%E7%9C%9F%E5%AE%9Eip/

