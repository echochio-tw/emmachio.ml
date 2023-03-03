---
layout: post
title: zabbix wb 監控
date: 2023-03-03
tags: zabbix web
---

https://XXX.XXXX.XXX/api/status


回應的是
```
"nodeStatus":"Node Healthy"
```

Preprocessing　設定
```
Name --> Parameters
JSONPath --> $.nodeStatus
JavaScript --> return JSON.stringify(value == 'Node Healthy' ? 1:0);
```

Trigger 
```
Problem expression
avg(/Api-status/nodeStatus,5m)=0

Recovery expression
avg(/Api-status/nodeStatus,5m)=1
```


這邊還有其他的運用如抓 API 看餘額

<img src="/images/posts/zabbix/1.png">

<img src="/images/posts/zabbix/2.png">

<img src="/images/posts/zabbix/3.png">
