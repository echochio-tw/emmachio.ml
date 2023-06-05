---
layout: post
title: GKE minio 做 clone 过程
date: 2023-06-05
tags: k8s
---

minio 做 clone 过程

先停 public GKE 的 minio
```
kubectl scale -n major deployment minio-master --replicas=0
```

再去GCE 那边 找到 public GKE minio PVC 的 HDD 做 clone 叫 clone-minio

再启动 public GKE 的 minio
```
kubectl scale -n major deployment minio-master --replicas=1
```

以下到新的 private GKE 操作

建立 PV (HDD 大小自己改成正确的)
```
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: PersistentVolume
metadata:
  name: minio
spec:
  storageClassName: standard-rwo
  persistentVolumeReclaimPolicy: Delete
  capacity:
    storage: 2Gi
  accessModes:
    - ReadWriteOnce
  claimRef:
    namespace: major
    name: minio
  gcePersistentDisk:
    pdName: clone-minio
    fsType: ext4
EOF
```

建立 PVC (HDD 大小自己改成正确的)
```
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  namespace: major
  name: minio
spec:
  storageClassName: standard-rwo
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 2Gi
EOF
```

建立 minio 去挂刚刚建立的 PVC
```
helm install minio stable/minio -f minio.yaml -n major --set persistence.existingClaim=minio
```

https://raw.githubusercontent.com/echochio-tw/echochio.ml/master/images/minio.ymal

