---
layout: post
title: 用 haproxy 轉發 XMPP 
date: 2023-03-26
tags: swift XMPP openfire
---

/etc/haproxy/haproxy.cfg
```
defaults
    timeout connect 5s
    timeout client 24h
    timeout server 60m
    log 127.0.0.1 local0

frontend m1-access
        bind *:6222
        default_backend m1-access
        mode tcp
        option tcplog
        option tcpka
        option clitcpka

backend m1-access
        option independent-streams
        server worker0 127.0.0.1:5222 check
```

設定完要測試與啟定
```
/usr/sbin/haproxy -c -V -f /etc/haproxy/haproxy.cfg
systemctl restart haproxy
```

安裝openfire
```
mkdir -p /srv/docker/openfire
docker run --name openfire -d --restart=always --publish 9090:9090 --publish 5222:5222 --publish 7777:7777 --volume /srv/docker/openfire:/var/lib/openfire sameersbn/openfire:3.10.3-19
docker exec -it openfire tail -f /var/log/openfire/info.log
```

安裝後開網頁 http://127.0.0.1:8080 看圖說故事....

下載安裝 XMPP client https://swift.im/

測試可以client 登入 127.0.0.1:6222

openfire 管理介面 可以 廣播 到 swift
