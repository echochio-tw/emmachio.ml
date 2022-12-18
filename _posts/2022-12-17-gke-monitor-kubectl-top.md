---
layout: post
title: 用 kubectl top nodes 顯示負載量
date: 2022-12-1
tags: k8s GKE
---

kubectl top nodes
```
NAME                          CPU(cores)   CPU%   MEMORY(bytes)   MEMORY%
gke-happy-pool-03c06af7-rr1p   221m         5%     2018Mi          15%
gke-happy-pool-6ec0146d-pdk6   294m         7%     2358Mi          17%
gke-happy-pool-f6d3a1c3-lxg1   258m         6%     2661Mi          20%
```
如果用 ssh 到原本的 user 那環境都會存在例如 gcloud ......

這邊用 python 將資料撈出來變成 list 之後如何用自行自行考量 (當然那 password 要換你的)
```
import paramiko
import re
client = paramiko.SSHClient()
client.set_missing_host_key_policy(paramiko.MissingHostKeyPolicy())
client.connect("127.0.0.1", port=22, username="root", password="password")
stdin, stdout, stderr = client.exec_command("kubectl top nodes")
opt = stdout.readlines()
opt = "".join(opt)
data = [(' '.join(i.split())).split(' ') for i in opt.split('\n')]
data.pop(-1)
data.pop(0)
print (data)
```

```
[['gke-happy-pool-03c06af7-rr1p', '182m', '4%', '2015Mi', '15%'], ['gke-happy-pool-6ec0146d-pdk6', '284m', '7%', '2354Mi', '17%'], ['gke-happy-pool-f6d3a1c3-lxg1', '258m', '6%', '2584Mi', '19%']]
```

也可換成 kubectl top pods -A 之後如何用就自行考量
```
import paramiko
import re
client = paramiko.SSHClient()
client.set_missing_host_key_policy(paramiko.MissingHostKeyPolicy())
client.connect("127.0.0.1", port=22, username="root", password="password")
stdin, stdout, stderr = client.exec_command("kubectl top pods -A")
opt = stdout.readlines()
opt = "".join(opt)
data = [(' '.join(i.split())).split(' ') for i in opt.split('\n')]
data.pop(-1)
data.pop(0)
print (data)
```
