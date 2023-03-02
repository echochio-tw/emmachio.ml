---
layout: post
title: zabbix 監控 mysql
date: 2007-06-28
tags: zabbix mysql
---
zabbix 監控 mysql

安裝 Zabbix agent 2 選
```
MySQL by Zabbix agent 2
```

Macro設定
```
Macro ==> Value
{$MYSQL.DSN}  ==>  tcp://127.0.0.1:3306
{$MYSQL.PASSWORD}  ==>   lovetoplay
{$MYSQL.USER}  ==>  root
```
測試不在 mysql 本機 也能監測 用遠端 如  
```
tcp://10.140.0.99:3307 
```

k8s 內的 mysql 用 ssh tunnel

以下是將 mysql 3306 導到本地 9000
```
#!/bin/bash
in1=`netstat -natp|grep 127.0.0.1:9000|wc -l`
if [ "$in1" != "1" ];then
  /usr/local/bin/kubectl exec -ti $(kubectl get pod -l app=mysql -o name -n project) -n project -- sh -c "rm -rf  /etc/apt/sources.list.d/mysql.list;apt update;apt install openssh-client sshpass procps -y;pkil
l ssh"
  /usr/local/bin/kubectl exec -ti $(kubectl get pod -l app=mysql -o name -n project) -n project -- sh -c "sshpass -p 'psword' ssh -NfR 9000:127.0.0.1:3306 root@10.140.0.11 -o StrictHostKeyChecking=no"
fi
```



2023-03-01-zabbix-monitor-mysql.md

```
SHELL=/bin/bash
 ps -ef | grep pods_check.py|grep -v grep;[ $? == 1 ] &&  at now  <<< "/usr/bin/python3 /root/pods_check.py &"
