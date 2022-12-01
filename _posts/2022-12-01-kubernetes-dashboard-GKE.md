---
layout: post
title: GKE 安裝 kubernetes-dashboard 讓外部IP可訪問
date: 2022-12-01
tags: k8s GKE
---

GKE 安裝 kubernetes-dashboard 讓外部IP可訪問

```
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.7.0/aio/deploy/recommended.yaml
kubectl get service kubernetes-dashboard -n kubernetes-dashboard
kubectl patch svc kubernetes-dashboard -n kubernetes-dashboard -p '{"spec": {"type": "LoadBalancer"}}'
kubectl get service kubernetes-dashboard -n kubernetes-dashboard
```
等到顯示
```
NAME                   TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)         AGE
kubernetes-dashboard   LoadBalancer   10.16.130.180   34.81.100.122   443:30706/TCP   5
```

瀏覽 :
https://34.81.100.122

建立 TOKEN
```
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: ServiceAccount
metadata:
  name: admin-user
  namespace: kubernetes-dashboard 
EOF

```

```
cat <<EOF | kubectl apply -f -
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: admin-user
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- kind: ServiceAccount
  name: admin-user
  namespace: kubernetes-dashboard 
EOF
```

建立不過期 token (--duration=0)
```
kubectl -n kubernetes-dashboard create token admin-user --duration=0
```

```
eyJhbGciOiJSUzI1NiIsImtpZCI6ImNNMmJ1TmNZX2kwRUd5SXl1TmdYdnBuSkhCMG1kdXBsWGcteFljdUFTWHcifQ.eyJhdWQiOlsiaHR0cHM6Ly9jb250YWluZXIuZ29vZ2xlYXBpcy5jb20vdjEvcHJvamVjdHMvc3RhcnRsdW8tMzcwMjA1L2xvY2F0aW9ucy9hc2lhLWVhc3QxL2NsdXN0ZXJzL2F1dG9waWxvdC1jbHVzdGVyLXN0YXJsdW8iXSwiZXhwIjoxNjY5ODc0NTU3LCJpYXQiOjE2Njk4NzA5NTcsImlzcyI6Imh0dHBzOi8vY29udGFpbmVyLmdvb2dsZWFwaXMuY29tL3YxL3Byb2plY3RzL3N0YXJ0bHVvLTM3MDIwNS9sb2NhdGlvbnMvYXNpYS1lYXN0MS9jbHVzdGVycy9hdXRvcGlsb3QtY2x1c3Rlci1zdGFybHVvIiwia3ViZXJuZXRlcy5pbyI6eyJuYW1lc3BhY2UiOiJrdWJlcm5ldGVzLWRhc2hib2FyZCIsInNlcnZpY2VhY2NvdW50Ijp7Im5hbWUiOiJhZG1pbi11c2VyIiwidWlkIjoiYWU3MGRlZTktOWY5OS00MDQ1LWFlN2QtNWVmODdlNDBlMTcxIn19LCJuYmYiOjE2Njk4NzA5NTcsInN1YiI6InN5c3RlbTpzZXJ2aWNlYWNjb3VudDprdWJlcm5ldGVzLWRhc2hib2FyZDphZG1pbi11c2VyIn0.262YvSpGn08HF2JHTQOcxDsOnEXptudLr30kMhwJIkFXbxAdQbzKf4Yew9xcCbuZnqu1_QDhJyQnE5SJeYQ2yjeognmuZ2DgYIQbdZ3EYCd2Oem9MDMHOJQ0rmicQbd-qkAtOdOnzGhXfbbW6tPqDEUl3XRk8M1vtfcuqzvCFf0DXsEtfdZY_SwrKRZeRji_gVNIVTYoU_SbbmmKPXVKwVL95SN95CaD9oKd7srcmUDIpM-EvrXkQLCw2VqrwK4coHaNJXSItLsndhqmOE4JbgVNOCDsYLuBt72yJ4WYug98XdcONw5Feq7OrUNJU8tcXVBcDSYKGv_ymdiDtyxAvA
```

ingress 的方式

先裝 dashboard 與取 token 不設定 LoadBalancer

內定是 ClusterIP

```
NAME                   TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)   AGE
kubernetes-dashboard   ClusterIP   10.112.3.11   <none>        443/TCP   36m
```

裝 ingress-nginx
```
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm repo update
helm install nginx-ingress ingress-nginx/ingress-nginx
kubectl get deployment nginx-ingress-ingress-nginx-controller
kubectl get service nginx-ingress-ingress-nginx-controller
```

```
cat <<EOF | kubectl apply -f -
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: dashboard-ingress
  namespace: kubernetes-dashboard
  annotations:
    kubernetes.io/ingress.class: "nginx"
    nginx.ingress.kubernetes.io/ssl-redirect: "false"
    nginx.ingress.kubernetes.io/backend-protocol: "HTTPS"
spec:
  rules:
  - host: "dashboard.testme.com"
    http:
      paths:
      - pathType: ImplementationSpecific
        backend:
          service:
            name: kubernetes-dashboard
            port:
              number: 443
EOF
```

```
kubectl get ingress -n kubernetes-dashboard
NAME                CLASS    HOSTS                     ADDRESS         PORTS   AGE
dashboard-ingress   <none>   dashboard.testme.com   35.226.14.113   80      15
```

設定 DNS dashboard.testme.com ->  35.226.14.113   

訪問 http://dashboard.testme.com
