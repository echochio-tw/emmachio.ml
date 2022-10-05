---
layout: post
title: minikube pv pvc 
date: 2022-10-05
tags: 
---

愛斬人 蓮花 ...A90/2.5 已賣出有點後悔說

用 kubernetes-dashboard 將yaml 加一加
有 MOUNT 到那空間就對了

<img src="./images/posts/minikube_PVC/1.jpg">
<img src="./images/posts/minikube_PVC/2.jpg">
<img src="./images/posts/minikube_PVC/3.jpg">

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
