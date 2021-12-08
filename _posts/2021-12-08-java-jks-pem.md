---
layout: post
title: 在 JKS 中使用 LetsEncrypt 證書
date: 2021-12-08
tags: ssl jks pem
---

在 Java 中處理證書總是很有趣。 Java 使用的密鑰庫與您在 Web 服務器或 node.js 中使用的證書文件不同。

要
```
能上網

OpenSSL 安裝

能夠運行 certbot LetsEncrypt 註冊

DNS 可以添加記錄

Java runtime 安裝

公共 IP 地址

```

這邊使用 demo.example.com 當範例


1. 從 LetsEncrypt 獲取 PEM 證書 (DNS 要添加 txt)
```
certbot certonly --manual --preferred-challenges dns -d demo.example.com dns-01 --server https://acme-v02.api.letsencrypt.org/directory 
```

2. 將 PEM 轉換為 PKCS12 格式

```
openssl pkcs12 -export -in fullchain1.pem -inkey privkey1.pem -out keystore.p12
```

3. 然後 PKCS12 轉換為 jks 格式。
```
keytool -importkeystore -srckeystore keystore.p12 -srcstoretype pkcs12  -destkeystore pkkpj.com.jks -deststoretype jks
```

缺點：每 90 天重複一次。
