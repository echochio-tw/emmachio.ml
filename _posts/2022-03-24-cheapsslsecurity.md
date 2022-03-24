---
layout: post
title: cheapsslsecurity ssl 製作與安裝 for nginx
date: 2022-03-24
tags: cheapsslsecurity ssl
---

cheapsslsecurity ssl 製作與安裝 for nginx
cheapsslsecurity ssl 製作與安裝 for nginx

Linux 系統下生成處理 或 還是用網頁

https://www.cloudmax.com.tw/service/ssl-tools

```
#openssl req -new -newkey rsa:4096 -nodes -out star_mmxxll_net.csr -keyout star_mmxxll_net.key -subj "/C=HK/ST=hk/L=HK/O=Fig inc/CN=*.mmxxll.net"
```

```
# ls 
star_mmxxll_net.csr
star_mmxxll_net.key
```

.key 文檔需要保留(這是私鑰)

.csr 我們需要馬上使用到這文檔中的內容

進入Cheapsslsecurity 證書網站，後臺管理，

選擇訂單記錄，進行重新生成證書(Re-issue Certificate)

如新購選新購 

進入重新生成證書頁面

我們現在需要將，在之前生成 star_mmxxll_net.csr 文檔的內容，複製在這頁面中，進行提交


選 TXT 去 DNS 設定

等一下產生後下載證書(Download Certificate)

壓縮包中有多個文件，我們需要合併這多個文檔，生成 star_mmxxll_net_pub.crt
```
# cat STAR_mmxxll_net.crt DigiCertCA.crt USERTrustRSAAddTrustCA.crt TrustedRoot.crt > star_mmxxll_net_pub.crt
```

Nginx 配置
```
ssl on;
ssl_certificate /www/nginx/cert/star_mmxxll_net_pub.crt;
ssl_certificate_key /www/nginx/cert/star_mmxxll_net.key;
ssl_prefer_server_ciphers on;
```
