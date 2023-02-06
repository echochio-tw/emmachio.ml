---
layout: post
title: GKE + Prometheus + Grafana
date: 2023-02-06
tags: k8s
---

```
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
```

```
wget https://echochio.ml/images/values-prometheus.yaml
```

```
helm install -f values-prometheus.yaml prometheus-stack prometheus-community/kube-prometheus-stack -n prometheus-stack --version 40.3.1 --create-namespace
```

```
cat <<EOF | kubectl apply -f -
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: prometheus-stack
  namespace: prometheus-stack
  annotations:
    kubernetes.io/ingress.class: "nginx"
    nginx.ingress.kubernetes.io/ssl-redirect: "false"
spec:
  rules:
  - host: "grafana.echochio.ml"
    http:
      paths:
      - pathType: ImplementationSpecific
        backend:
          service:
            name: prometheus-stack-grafana
            port:
              number: 80
EOF
```

```
https://grafana.echochio.ml
admin
changemechangeme
```
