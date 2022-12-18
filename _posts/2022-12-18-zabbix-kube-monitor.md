---
layout: post
title: zabbix 監控 GKE 自動偵測
date: 2022-12-18
tags: k8s GKE
---
這是用最傳統的方式也是最簡單方式

root 環境必須要確認可以執行 kubectl top node 取值再往下做

先加一些檔案 (請改程式password)

/usr/lib/zabbix/externalscripts/zbx_k8s_lld.py
```
#!/usr/bin/env python3
import json
import paramiko
cmd = "kubectl get pod -o=custom-columns=NAME:.metadata.name,STATUS:.status.phase,NODE:.spec.nodeName  --all-namespaces -o json"
client = paramiko.SSHClient()
client.set_missing_host_key_policy(paramiko.MissingHostKeyPolicy())
client.connect("127.0.0.1", port=22, username="root", password="password")
stdin, stdout, stderr = client.exec_command(cmd)
opt = stdout.readlines()
opt = "".join(opt)
client.close()
del client, stdin, stdout, stderr
data = json.loads(opt)
containers = []
if data:
    for i in data["items"]:
        for c in i["spec"]["containers"]:
            e = {"{#NAME}": i["metadata"]["name"], "{#NAMESPACE}":
                    i["metadata"]["namespace"], "{#CONTAINER}": c["name"],
                    "{#NODE}": i["spec"]["nodeName"]}
            containers.append(e)
    print("{\"data\":"+json.dumps(containers)+"}")
else:
    print(err, file=sys.stderr)
```

/usr/lib/zabbix/externalscripts/zbx_k8s_node.py
```
#!/usr/bin/env python3
import sys
import paramiko
import sys
if len(sys.argv)>1:
    input = str(sys.argv[1])
else:
    input = ''
cmd = 'kubectl top node '+input+' --no-headers=true'
#print (cmd)
client = paramiko.SSHClient()
client.set_missing_host_key_policy(paramiko.MissingHostKeyPolicy())
client.connect("127.0.0.1", port=22, username="root", password="password")
stdin, stdout, stderr = client.exec_command(cmd)
opt = stdout.readlines()
opt = "".join(opt)
client.close()
del client, stdin, stdout, stderr
print (opt)
```

/usr/lib/zabbix/externalscripts/zbx_k8s_npods.py 
```
#!/usr/bin/env python3
import sys
import paramiko
import sys
if len(sys.argv)>2:
    p1= str(sys.argv[1])
    p2= str(sys.argv[2])
    cmd ='kubectl top pods '+p1+' -n '+p2+' --no-headers=true'
else:
    cmd ='kubectl top pods --no-headers=true'
#print (cmd)
client = paramiko.SSHClient()
client.set_missing_host_key_policy(paramiko.MissingHostKeyPolicy())
client.connect("127.0.0.1", port=22, username="root", password="password")
stdin, stdout, stderr = client.exec_command(cmd)
opt = stdout.readlines()
opt = "".join(opt)
client.close()
del client, stdin, stdout, stderr
print (opt)
```

/etc/zabbix/zabbix_agentd.conf 後面加
```
UserParameter=discovery.k8s.cluster,/usr/lib/zabbix/externalscripts/zbx_k8s_lld.py
UserParameter=discovery.k8s.node.pcpu[*],/usr/lib/zabbix/externalscripts/zbx_k8s_node.py $1 | tr -s ' ' | cut -d' ' -f3 | grep -Eo '[0-9]*'
UserParameter=discovery.k8s.node.pmem[*],/usr/lib/zabbix/externalscripts/zbx_k8s_node.py $1 |tr -s ' '|cut -d' ' -f5|grep -Eo '[0-9]*'
UserParameter=discovery.k8s.node.cpu[*],/usr/lib/zabbix/externalscripts/zbx_k8s_node.py $1 | tr -s ' ' | cut -d' ' -f2 | grep -Eo '[0-9]*'
UserParameter=discovery.k8s.node.mem[*],/usr/lib/zabbix/externalscripts/zbx_k8s_node.py $1 | tr -s ' ' | cut -d' ' -f4 | grep -Eo '[0-9]*'
UserParameter=discovery.k8s.npods.mem[*],/usr/lib/zabbix/externalscripts/zbx_k8s_npods.py $1 $2 | tr -s ' ' | cut -d' ' -f3 | grep -Eo '[0-9]*'
UserParameter=discovery.k8s.npods.cpu[*],/usr/lib/zabbix/externalscripts/zbx_k8s_npods.py $1 $2 | tr -s ' ' | cut -d' ' -f2 | grep -Eo '[0-9]*'
```

重啟 agent
```
systemctl restart zabbix-agent.service
```

自動偵測 測試
```
zabbix_agentd -t discovery.k8s.cluster
zabbix_get -s 127.0.0.1 -p 10050 -k 'discovery.k8s.cluster'
```
其他取值 測試(自行換節點名 或 POD 名)
```
zabbix_agentd -t discovery.k8s.node.pcpu[gke-happy-pool-03c06af7-rr1p]
zabbix_get -s 127.0.0.1 -p 10050 -k 'discovery.k8s.node.pcpu[gke-happy-pool-03c06af7-rr1p]'
zabbix_agentd -t discovery.k8s.node.pmem[gke-happy-pool-03c06af7-rr1p]
zabbix_get -s 127.0.0.1 -p 10050 -k 'discovery.k8s.node.pmem[gke-happy-pool-03c06af7-rr1p]'
zabbix_agentd -t discovery.k8s.node.cpu[gke-happy-pool-03c06af7-rr1p]
zabbix_get -s 127.0.0.1 -p 10050 -k 'discovery.k8s.node.cpu[gke-happy-pool-03c06af7-rr1p]'
zabbix_agentd -t discovery.k8s.node.mem[gke-happy-pool-03c06af7-rr1p]
zabbix_get -s 127.0.0.1 -p 10050 -k 'discovery.k8s.node.mem[gke-happy-pool-03c06af7-rr1p]'
zabbix_agentd -t discovery.k8s.npods.mem[redis-master-0,major]
zabbix_get -s 127.0.0.1 -p 10050 -k 'discovery.k8s.npods.mem[redis-master-0,major]'
zabbix_agentd -t discovery.k8s.npods.cpu[redis-master-0,major]
zabbix_get -s 127.0.0.1 -p 10050 -k 'discovery.k8s.npods.cpu[redis-master-0,major]'
```

zabbix 內建立一個主機叫 GKE 用 Template: Kubernetes Top Resources by kubectl (匯入)

kubernetes-top-resources.xml [kubernetes-top-resources.xml](../images/gke/kubernetes-top-resources.xml)

<img src="/images/gke/1.png">

在 configuration 內 hosts 的 GKE 裡面選 Discovery 建立 Discovery cluster

<img src="/images/gke/2.png">

```
Name: Discovery cluster
key: discovery.k8s.cluster
Host interface: 127.0.0.1:10050
Update interval: 10m
```
<img src="/images/gke/3.png">

過一下子可出現 Item prototypes 有東西

Trigger prototypes 要自建立

過一下 GKE 這個 host 就有圖了

<img src="/images/gke/4.png">
這邊是參考
https://faun.dev/c/stories/filipisaci/httpsmediumcomfilipisacicreating-a-zabbix-low-level-discovery-rule-step-by-step-1e30f7b516d7/
