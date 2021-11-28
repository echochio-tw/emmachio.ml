---
layout: post
title: nginx config server_name 正規
date: 2021-11-28
tags: nginx 正規 regular Regular Expression
---

nginx server_name 正規測試

https://nginx.viraptor.info/

任何 域名 點com 的
```
~^(www)\.(?<domain>[^.]+)\.com$;
~^(?<domain>[^.]+)\.com$;
```

例如 http://pma.ne2011.com/images/ 可用
```
server {
 listen       80;
 server_name ~^(pma)\.([a-z][a-z][0-9][0-9][0-9][0-9])\.com$;
 access_log   logs/domain2.access.log  main;
 
 location ~ ^/(images|javascript|js|css|flash|media|static)/
 {
  root    /var/www/virtual/big.server.com/htdocs;
  expires 30d;
 }

 location / {
  proxy_pass      http://127.0.0.1:8080;
 }
}
```

客人的是 ww1.as3211.com ww2.as3211.com ww3.as3211.com
```
~^(ww[1-3])\.([a-z][a-z][0-9][0-9][0-9][0-9])\.com$;
```

as3211.com
```
~^([a-z][a-z][0-9][0-9][0-9][0-9])\.com$;
```

大概就這樣
