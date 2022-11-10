---
layout: post
title: mail server SPF &DKIM
date: 2022-11-07
tags: postfix mail server
---

在GCP 安裝了一台 mail server (POSTFIX) 設定資訊

centos 或 ubuntu 就用套件安裝 postfix 再來設定...

/var/spool/postfix/etc/resolv.conf
```
nameserver 8.8.8.8
nameserver 8.8.4.4
nameserver 1.1.1.1
```

yum install opendkim opendkim-tools openssl

依照 https://askubuntu.com/questions/438756/using-dkim-in-my-server-for-multiple-domains-websites

/etc/opendkim.conf
```
# This is a basic configuration that can easily be adapted to suit a standard
# installation. For more advanced options, see opendkim.conf(5) and/or
# /usr/share/doc/opendkim/examples/opendkim.conf.sample.
#
#Domain                  example.com
#KeyFile                 /etc/opendkim/201205.private
#Selector                201205
#
# Commonly-used options
Canonicalization        relaxed/simple
Mode                    sv
SubDomains              yes
# Log to syslog
Syslog                  yes
LogWhy                  yes
# Required to use local socket with MTAs that access the socket as a non-
# privileged user (e.g. Postfix)
UMask                   022
UserID                  opendkim:opendkim
#
KeyTable                /etc/opendkim/KeyTable
SigningTable            /etc/opendkim/SigningTable
ExternalIgnoreList      /etc/opendkim/TrustedHosts
InternalHosts           /etc/opendkim/TrustedHosts
#
Socket                  inet:8891@localhost
#EOF
```

```
mkdir /etc/opendkim
```

/etc/opendkim/TrustedHosts (裡面 35.121.21.112 是GCP IP 這邊我的域名叫 ms10.otherdomain.com)
```
127.0.0.1
localhost
35.121.21.112
ms10.otherdomain.com
```

/etc/default/opendkim
```
SOCKET="inet:8891@localhost"
```

/etc/postfix/main.cf 最後面加
```
milter_default_action = accept
milter_protocol = 6
smtpd_milters = inet:localhost:8891
non_smtpd_milters = inet:localhost:8891
```

```
systemctl restart opendkim
systemctl reload postfix
systemctl restart postfix
```

這邊我的域名叫 ms10.otherdomain.com
```
mkdir -p /etc/opendkim/keys/ms10.otherdomain.com
cd /etc/opendkim/keys/ms10.otherdomain.com
opendkim-genkey -r -d ms10.otherdomain.com
chown opendkim:opendkim default.private
```

/etc/opendkim/KeyTable
```
default._domainkey.ms10.otherdomain.com ms10.otherdomain.com:default:/etc/opendkim/keys/ms10.otherdomain.com/default.private
```

/etc/opendkim/SigningTable
```
otherdomain.com default._domainkey.ms10.otherdomain.com
```

/etc/opendkim/TrustedHosts
```
ms10.otherdomain.com
```

cat /etc/opendkim/keys/ms10.otherdomain.com/default.txt
```
default._domainkey      IN      TXT     ( "v=DKIM1; k=rsa; s=email; "
          "p=MIGfMA0GCSqGSIb3DQEBAQUAA4GNADCBiQKBgQCrl95KgJkJ7vjSHdeNvSr6yNRDtqc9pWGdYl8fSjQKIdSgsshLj7EbGuIKKYh2WKHrGiDdZi4ORhhnYMKyzrUTRIjEq33SEWuYiWTHVZMMNuUiYQ4viBIDbhiPO+FPsi5rR/Veuqiet4mAoeklpDS9nVFor5wupmZW8q0p0YHHZQIDAQAB" )  ; ----- DKIM key default for ms10.otherdomain.com

```

SPF 設定 (裡面 35.121.21.112 是GCP IP 這邊我的域名叫 ms10.otherdomain.com)
```
ms10.otherdomain.com  TXT v=spf1 a:ms10.otherdomain.com ip4:35.121.21.112
```

DKIM 設定 是在 
default._domainkey.ms10.otherdomain.com 的 TXT
```
v=DKIM1; k=rsa; s=email;
p=MIGfMA0GCSqGSIb3DQEBAQUAA4GNADCBiQKBgQCrl95KgJkJ7vjSHdeNvSr6yNRDtqc9pWGdYl8fSjQKIdSgsshLj7EbGuIKKYh2WKHrGiDdZi4ORhhnYMKyzrUTRIjEq33SEWuYiWTHVZMMNuUiYQ4viBIDbhiPO+FPsi5rR/Veuqiet4mAoeklpDS9nVFor5wupmZW8q0p0YHHZQIDAQAB
```

DMARC 設定 是在 _dmarc.ms10.otherdomain.com 的 TXT
```
v=DMARC1; p=quarantine; rua=mailto:info@ms10.otherdomain.com;
```

```
systemctl restart opendkim
systemctl reload postfix
systemctl restart postfix
```

<img src="/images/posts/mailsrver/DKIM_SPF1.png">
<img src="/images/posts/mailsrver/DKIM_SPF3.png">
<img src="/images/posts/mailsrver/DKIM_SPF2.png">
已設定 SPF 與 DKIM 基本收到信已不太會認為異常信件拒絕

但SPAM 會有單一IP發太多還是被擋所以不能狂發信 .... 哈
