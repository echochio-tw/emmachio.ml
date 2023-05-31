---
layout: post
title: create nfs pvc for redis mysql minio
date: 2023-05-31
tags: k8s
---

建立一个 nfs 要安装 设定 anongid anonuid for mysql

```
/data 192.168.99.0/24(rw,no_root_squash,no_subtree_check,sync,anongid=999,anonuid=999)
```

k8s 安装 NFS client for pv
```
helm repo add nfs-subdir-external-provisioner https://kubernetes-sigs.github.io/nfs-subdir-external-provisioner/

helm install nfs-subdir-external-provisioner \
nfs-subdir-external-provisioner/nfs-subdir-external-provisioner \
-n major --set nfs.server=192.168.99.7  \
--set nfs.path=/data

```

建立 redis pvc
```
cat <<EOF | kubectl apply -f -
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: redis-data-redis-master-0
  namespace: major
spec:
  accessModes:
    - ReadWriteMany
  storageClassName: nfs-client
  resources:
    requests:
      storage: 8Gi
EOF
```

安装 redis
```
helm repo add bitnami https://charts.bitnami.com/bitnami
 
helm install redis -n major \
--set architecture=standalone \
--set auth.enabled=false bitnami/redis
```

建立 minio pvc
```
cat <<EOF | kubectl apply -f -
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: minio
  namespace: major
spec:
  accessModes:
    - ReadWriteMany
  storageClassName: nfs-client
  resources:
    requests:
      storage: 2Gi
EOF
```

安装 minio
```
helm repo add minio https://helm.min.io/

helm install --namespace major \
--generate-name \
--set persistence.existingClaim=minio minio/minio
```

安装 mysql pvc
```
cat <<EOF | kubectl apply -f -
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: mysql
  namespace: major
spec:
  accessModes:
    - ReadWriteMany
  storageClassName: nfs-client
  resources:
    requests:
      storage: 10Gi
EOF
```

安装 mysql
```
helm install  mysql-server  \
-n major --set persistence.existingClaim=mysql stable/mysql
```
