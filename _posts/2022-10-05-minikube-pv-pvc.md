---
layout: post
title: minikube pv pvc 
date: 2022-10-05
tags: 
---

用 kubernetes-dashboard 將yaml 加一加
有 MOUNT 到那空間就對了

<img src="/images/posts/minikube_PVC/1.png">
<img src="/images/posts/minikube_PVC/2.png">
<img src="/images/posts/minikube_PVC/3.png">

```
kind: StorageClass
apiVersion: storage.k8s.io/v1
metadata:
  name: standard
  uid: 675655e9-f48e-42f9-ac6f-964b491a2d4f
  resourceVersion: '285'
  creationTimestamp: '2022-09-14T03:21:34Z'
  labels:
    addonmanager.kubernetes.io/mode: EnsureExists
  annotations:
    kubectl.kubernetes.io/last-applied-configuration: >
      {"apiVersion":"storage.k8s.io/v1","kind":"StorageClass","metadata":{"annotations":{"storageclass.kubernetes.io/is-default-class":"true"},"labels":{"addonmanager.kubernetes.io/mode":"EnsureExists"},"name":"standard"},"provisioner":"k8s.io/minikube-hostpath"}
    storageclass.kubernetes.io/is-default-class: 'true'
  managedFields:
    - manager: kubectl-client-side-apply
      operation: Update
      apiVersion: storage.k8s.io/v1
      time: '2022-09-14T03:21:34Z'
      fieldsType: FieldsV1
      fieldsV1:
        f:metadata:
          f:annotations:
            .: {}
            f:kubectl.kubernetes.io/last-applied-configuration: {}
            f:storageclass.kubernetes.io/is-default-class: {}
          f:labels:
            .: {}
            f:addonmanager.kubernetes.io/mode: {}
        f:provisioner: {}
        f:reclaimPolicy: {}
        f:volumeBindingMode: {}
provisioner: k8s.io/minikube-hostpath
reclaimPolicy: Delete
volumeBindingMode: Immediate
```

```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-localfile
  namespace: default
spec:
  accessModes:
    - ReadWriteMany
  storageClassName: standard
  resources:
    requests:
      storage: 100Gi
  dnsPolicy: Default
```

```
apiVersion: v1
kind: Pod
metadata:
  name: mypod
  namespace: default
spec:
  containers:
  - name: mypod
    image: nginx:1.15.5-alpine
    volumeMounts:
    - mountPath: "/mnt/my-localfile"
      name: volume
  volumes:
      - name: volume
        persistentVolumeClaim:
          claimName: my-localfile
  hostNetwork: true
  dnsPolicy: Default
```

## 下面用 NFS當PV 記錄一下

ip a |grep eth0 取IP
```
root@chio:/data# ip a |grep eth0
6: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000
    inet 172.30.226.133/20 brd 172.30.239.255 scope global eth0
```

安裝NFS Server
```
apt update
apt install nfs-kernel-server
mkdir -p /data
chown -R nobody:nogroup /data
chmod 777 /data
```
設定分享目錄 編輯 /etc/exports
```
/data *(fsid=0,rw,sync,no_subtree_check)
```

檢查是否可分享 
```
exportfs -a
```

啟動 NFS server
```
systemctl restart nfs-kernel-server
```

PersistentVolume
```
apiVersion: v1
kind: PersistentVolume
metadata:
  name: my-pv
spec:
  storageClassName: standard
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteOnce
  nfs:
    path: /data
    server: 172.30.226.133
```

PersistentVolumeClaim
```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-pvc
  namespace: default
spec:
  storageClassName: standard
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
  dnsPolicy: Default
```
  
  pod
```
 apiVersion: v1
kind: Pod
metadata:
  name: mypod
  namespace: default
spec:
  containers:
  - name: mypod
    image: nginx:1.15.5-alpine
    volumeMounts:
    - mountPath: "/mnt/my-pvc"
      name: volume
  volumes:
      - name: volume
        persistentVolumeClaim:
          claimName: my-pvc
  hostNetwork: true
  dnsPolicy: Default
```

shell to pod
```
kubectl exec --stdin --tty mypod -- /bin/sh
```

check nfs mount
```
df -h |grep my-pvc
```

正常是 :
```
172.30.226.133:/data    251.0G      8.0G    230.2G   3% /mnt/my-pv
```
 
 用 RAM DISK ... 當NFS 這邊就不重寫了
```
mkdir /mnt/ramdisk 
mount -t tmpfs -o rw,size=1G tmpfs /mnt/ramdisk
```

