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

建立不過期 Secret
```
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: Secret
metadata:
  name: admin-user
  annotations:
    kubernetes.io/service-account.name: "admin-user"
type: kubernetes.io/service-account-token
EOF

export DASHBOARD=$(kubectl -n kubernetes-dashboard get secret |awk '{print $1}'|grep admin-user)
echo $DASHBOARD
kubectl -n kubernetes-dashboard  describe secret $DASHBOARD
```

```
Name:         admin-user-token-qqv65
Namespace:    kubernetes-dashboard
Labels:       <none>
Annotations:  kubernetes.io/service-account.name: admin-user
              kubernetes.io/service-account.uid: b46b15e8-636a-4014-b6d4-a63c3fb3b209

Type:  kubernetes.io/service-account-token

Data
====
token:      eyJhbGciOiJSUzI1NiIsImtpZCI6IklPbXpYU2NYSWJ3WDJ2X0dCbC1WWEZsS0VjaFozTG5UVHREc2YySjE3clUifQ.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJrdWJlcm5ldGVzLWRhc2hib2FyZCIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VjcmV0Lm5hbWUiOiJhZG1pbi11c2VyLXRva2VuLXFxdjY1Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZXJ2aWNlLWFjY291bnQubmFtZSI6ImFkbWluLXVzZXIiLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlcnZpY2UtYWNjb3VudC51aWQiOiJiNDZiMTVlOC02MzZhLTQwMTQtYjZkNC1hNjNjM2ZiM2IyMDkiLCJzdWIiOiJzeXN0ZW06c2VydmljZWFjY291bnQ6a3ViZXJuZXRlcy1kYXNoYm9hcmQ6YWRtaW4tdXNlciJ9.SmBDDHg4c57RhACJCtGRstmhQpa6r1seu-vTnsuwVuywjjcR2FSu4Yu0W3_EXfo18U6220F72-BQGCvVbIWpUaNFLwHF79dxe967pKpJgnhtzk96H5VI9yJ0BJCdhVHX2m8Az2JlOmrLY5fH7fHcdsPZpc_bwFuz4YzjvV2CnjuQfAMMKaYYQ7kNhM7YsbwTQeEMUbK0cJknizeghXkdyMag5gB6OoIbkmtBoio-MINau70Hp4M-sHAGQt4AW6zX3iJNYWHycX3jG_uSR08PhxDpENozOQF--a5Rldo9QkcU_9uB3DFOVxTr2eHTdSzGUArosYVZHGvjnXu8ddDuWQ
ca.crt:     1509 bytes
namespace:  20 bytes
```
ReadOnly user
```
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: ServiceAccount
metadata:
  name: read-user
  namespace: kubernetes-dashboard
EOF
```

```
cat <<EOF | kubectl apply -f -
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: read-user
rules:
- apiGroups: ["*"]
  resources: ["*"]
  verbs: ["get", "watch", "list"]        
EOF
```

```
cat <<EOF | kubectl apply -f -
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: read-user
roleRef:
  apiGroup: rbac.authorization.k8s.io
  name: read-user
  kind: ClusterRole
subjects:
  - kind: ServiceAccount
    name: read-user
    namespace: kubernetes-dashboard
EOF
```

```
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: Secret
metadata:
  name: read-user
  annotations:
    kubernetes.io/service-account.name: "read-user"
type: kubernetes.io/service-account-token
EOF
```

```
export READBOARD=$(kubectl -n kubernetes-dashboard get secret |awk '{print $1}'|grep read-user)
echo $READBOARD
kubectl -n kubernetes-dashboard  describe secret $READBOARD
```

# ingress 的方式

這邊是參考 https://cloud.google.com/community/tutorials/nginx-ingress-gke 去改的

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

如果是 Private clusters GKE

發現 CPU 及 記憶體 沒資訊

LOG 看是
```
Metric client health check failed: the server is currently unable to handle the request (get services dashboard-metrics-scraper). Retrying in 30 seconds.
Internal error occurred: No metric client provided. Skipping metrics.
```

直接用 kubernetes-dashboard 線上修改 kubernetes-dashboard deployments 的 yaml

找到原本是
```
          args:
            - '--auto-generate-certificates'
            - '--namespace=kubernetes-dashboard'
```

變成
```
          args:
            - '--auto-generate-certificates'
            - '--namespace=kubernetes-dashboard'
            - '--sidecar-host=http://dashboard-metrics-scraper.kubernetes-dashboard.svc.cluster.local:8000'
```


dashboard-read-only.yaml
```
apiVersion: v1
kind: ServiceAccount
metadata:
  name: read-only-user
  namespace: kubernetes-dashboard
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  annotations:
    rbac.authorization.kubernetes.io/autoupdate: "true"
  labels:
  name: read-only-clusterrole
  namespace: default
rules:
- apiGroups:
  - ""
  resources: ["*"]
  verbs:
  - get
  - list
  - watch
- apiGroups:
  - extensions
  resources: ["*"]
  verbs:
  - get
  - list
  - watch
- apiGroups:
  - apps
  resources: ["*"]
  verbs:
  - get
  - list
  - watch
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: read-only-binding
roleRef:
  kind: ClusterRole
  name: read-only-clusterrole
  apiGroup: rbac.authorization.k8s.io
subjects:
- kind: ServiceAccount
  name: read-only-user
  namespace: kubernetes-dashboard
```

```
kubectl get secret -n kubernetes-dashboard $(kubectl get serviceaccount read-only-user -n kubernetes-dashboard -o jsonpath="{.secrets[0].name}") -o jsonpath="{.data.token}" | base64 --decode
```
