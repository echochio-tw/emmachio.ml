---
layout: post
title: 在 JKS 中使用 LetsEncrypt 證書
date: 2021-12-08
tags: ssl jks pem
---

在 Java 中處理證書總是很有趣。 Java 使用的密鑰庫與您在 Web 服務器或 node.js 中使用的證書文件不同。

Salesforce 是基於 Java 構建的，因此我們必須與 Keystore 和平相處。本文概述了在密鑰庫中使用 LetsEncrypt 證書的步驟。你會需要：

能上網

OpenSSL 安裝

能夠運行 certbot LetsEncrypt 註冊

DNS 可以添加記錄

Java runtime 安裝

公共 IP 地址

這邊使用 demo.example.com 當範例


1. 從 LetsEncrypt 獲取 PEM 證書 (DNS 要添加 txt)
```
certbot certonly --manual --preferred-challenges dns -d demo.example.com dns-01 --server https://acme-v02.api.letsencrypt.org/directory 
```

2. 將 PEM 轉換為 PKCS12 格式 首先將所有 PEM 文件連接成一個

```
cat /etc/letsencrypt/life/demo.example.com/*.pem > fullcert.pem
```

3. 然後使用 OpenSSL 將其轉換為 PKCS12 格式。

```
openssl pkcs12 -export -out fullchain.pkcs12 -in fullchain.pem
```

4. 準備一個 Java JSK 密鑰庫 您不能只創建一個空的密鑰庫，因此創建一個新的臨時密鑰並指定一個新的密鑰庫，然後刪除該密鑰。這為您提供了空的密鑰庫：

```
keytool -genkey -keyalg RSA -alias sfdcsec -keystore sfdcsec.ks
keytool -delete -alias sfdcsec -keystore sfdcsec.ks
```

將 pkcs12 導入 JKS (最後的步驟。不要忘記你的密碼)
```
keytool -v -importkeystore -srckeystore fullchain.pkcs12 -destkeystore sfdcsec.ks -deststoretype JKS
```

調整 Salesforce 導入的別名 Salesforce 導入實用程序對別名很挑剔。 之前的導入創建了 別名：1 需要更新:
```
keytool -keystore sfdcsec.ks -changealias -alias 1 -destalias demo_example_com
```

Salesforce 實例擁有正確簽名的證書。 缺點：每 90 天重複一次。
