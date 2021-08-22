---
layout: post
title: Let's Encrypt 免費SSL 憑證 (certbot + cloudflare)
date: 2021-08-22
tags: certbot cloudflare ssl
---

Let's Encrypt 免費SSL 憑證

certbot + cloudflare 免費 SSL 申請

也不知裝有沒有錯反正可以用
```
yum update
yum install certbot
yum install certbot python-certbot-nginx
yum install python3-certbot-dns-cloudflare
yum list available | grep cloudflare
yum install python2-certbot-dns-cloudflare.noarch
```

設定
```
cat /etc/letsencrypt/cli.ini
rsa-key-size = 4096
email = webmaster@gamil.com
```

傳統做法
```
[root@localhost ~]# certbot certonly -d "*.abcde.com" --manual --preferred-challenges \
>    dns-01 --server https://acme-v02.api.letsencrypt.org/directory 

Saving debug log to /var/log/letsencrypt/letsencrypt.log
Plugins selected: Authenticator manual, Installer None
Starting new HTTPS connection (1): acme-v02.api.letsencrypt.org
Requesting a certificate for *.abcde.com
Performing the following challenges:
dns-01 challenge for abcde.com

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Please deploy a DNS TXT record under the name
_acme-challenge.abcde.com with the following value:

0IHPpgkjEBg6Wmi7vte43zDrd_B1PqVNuFJqUsnsDwE

Before continuing, verify the record is deployed.
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Press Enter to Continue
```

cloudflare API 設定
```
[root@localhost ~]# cat /root//.secrets/certbot/cloudflare.ini
dns_cloudflare_email = abcde@gmail.com
dns_cloudflare_api_key = 12a933133045909646ca34aa59737f76d999
```

cloudflare.ini 權限要對
```
[root@localhost ~]# chmod 600 ~/.secrets/certbot/cloudflare.ini
```

測試可以的
```
[root@localhost ~]# certbot certonly \
>   --dns-cloudflare \
>   --dns-cloudflare-credentials ~/.secrets/certbot/cloudflare.ini \
>   -d jzsticy.com \
>   -d *.jzsticy.com \
>

Saving debug log to /var/log/letsencrypt/letsencrypt.log
Plugins selected: Authenticator dns-cloudflare, Installer None
Starting new HTTPS connection (1): acme-v02.api.letsencrypt.org
Requesting a certificate for jzsticy.com and *.jzsticy.com
Performing the following challenges:
dns-01 challenge for jzsticy.com
dns-01 challenge for jzsticy.com
Starting new HTTPS connection (1): api.cloudflare.com
Starting new HTTPS connection (1): api.cloudflare.com
Waiting 10 seconds for DNS changes to propagate
Waiting for verification...
Cleaning up challenges
Starting new HTTPS connection (1): api.cloudflare.com
Starting new HTTPS connection (1): api.cloudflare.com
Subscribe to the EFF mailing list (email: antillean@gamil.com).
Starting new HTTPS connection (1): supporters.eff.org

IMPORTANT NOTES:
 - Congratulations! Your certificate and chain have been saved at:
   /etc/letsencrypt/live/jzsticy.com/fullchain.pem
   Your key file has been saved at:
   /etc/letsencrypt/live/jzsticy.com/privkey.pem
   Your certificate will expire on 2021-11-20. To obtain a new or
   tweaked version of this certificate in the future, simply run
   certbot again. To non-interactively renew *all* of your
   certificates, run "certbot renew"
 - If you like Certbot, please consider supporting our work by:

   Donating to ISRG / Let's Encrypt:   https://letsencrypt.org/donate
   Donating to EFF:                    https://eff.org/donate-le
```
