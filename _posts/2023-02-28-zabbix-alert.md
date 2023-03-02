---
layout: post
title: zabbix More than 100 items having missing data
date: 2023-02-28
tags: zabbix 告警 
---

zabbix告警：
```
More than 100 items having missing data for more than 10 minutes
```

原因：
```
server端與proxy端時間不同步
server端分配的緩存不夠
server端分配的線程不夠
server端負載比較大{CPU,IO,MEM}
```

zabbix_server有沒有出現 告警
```
Zabbix poller processes more than 75% busy
```

修改配置文件增大線程數和緩存

zabbix_server.conf

```
StartPollers=500
StartPollersUnreachable=50
StartTrappers=30
StartDiscoverers=6
CacheSize=1G
CacheUpdateFrequency=300
StartDBSyncers=20
HistoryCacheSize=512M
TrendCacheSize=256M
# HistoryTextCacheSize=80M
ValueCacheSize=1G
```
HistoryTextCacheSize 新版的沒有這設定
