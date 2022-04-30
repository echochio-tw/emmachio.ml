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

客人說要在 godaddy 10個域名還要SSL

godaddy 就亂碼產生器去買 ....

NS 然後設定到 cloudflare (可以一次選那十個然後轉到 cloudflare NS)

建立 cloudflare 域名管理 當然用 API 建立
```
zone_names = ["fnnfra.com","bebsya.com","eonfij.com","vzfqoi.com","chyese.com","azxgaq.com","mopoie.com","odyauu.com","yleoyn.com","hvppah.com"]
for zone_name in zone_names:
    print(zone_name)
    cf = CloudFlare.CloudFlare(email='abcde@gmail.com', token='12a933133045909646ca34aa59737f76d999')
    zone_info = cf.zones.post(data={'jump_start':False, 'name': zone_name})
    zone_id = zone_info['id']
    print(zone_id)
```

檢查 NS
```
import dns.resolver
zone_names = ["fnnfra.com","bebsya.com","eonfij.com","vzfqoi.com","chyese.com","azxgaq.com","mopoie.com","odyauu.com","yleoyn.com","hvppah.com"]
for zone_name in zone_names:
    answers = dns.resolver.query(zone_name,'NS')
    for server in answers:
        print(server.target)
```

建立 SSL
```
import subprocess
import shlex
zone_names = ["fnnfra.com","bebsya.com","eonfij.com","vzfqoi.com","chyese.com","azxgaq.com","mopoie.com","odyauu.com","yleoyn.com","hvppah.com"]
for zone_name in zone_names:
    do = 'certbot certonly --dns-cloudflare --dns-cloudflare-credentials /root/.secrets/certbot/cloudflare.ini -d *.'+ zone_name +' -d '+ zone_name
    #print (shlex.split(do))
    subprocess.call(shlex.split(do))
```

察到期日
```
CERT=/etc/letsencrypt/live/chyese.com/fullchain1.pem
date --date="$(openssl x509 -enddate -noout -in $CERT | cut -d= -f 2)" --iso-8601
```
轉 key 與 crt
```
openssl rsa -in /etc/letsencrypt/archive/chyese.com/privkey1.pem -out /etc/letsencrypt/archive/chyese.com/privkey1.key
openssl x509 -in /etc/letsencrypt/archive/chyese.com/fullchain1.pem -out /etc/letsencrypt/archive/chyese.com/fullchain1.crt
```
網路上找的 https://codebox.net/pages/https-certificate-expiry-checker 還不錯用
```
import sys
import multiprocessing
import ssl
import socket
import datetime
import concurrent.futures
import math


DEFAULT_HTTPS_PORT = 443
WORKER_THREAD_COUNT = multiprocessing.cpu_count()
SOCKET_CONNECTION_TIMEOUT_SECONDS = 10
WARN_IF_DAYS_LESS_THAN = 7
EXIT_SUCCESS = 0
EXIT_EXPIRING_SOON = 1
EXIT_ERROR = 2
EXIT_NO_HOST_LIST = 9

def make_host_port_pair(endpoint):
    host, _, specified_port = endpoint.partition(':')
    port = int(specified_port or DEFAULT_HTTPS_PORT)

    return host, port

def pluralise(singular, count):
    return '{} {}{}'.format(count, singular, '' if count == 1 else 's')

def get_certificate_expiry_date_time(context, host, port):
    with socket.create_connection((host, port), SOCKET_CONNECTION_TIMEOUT_SECONDS) as tcp_socket:
        with context.wrap_socket(tcp_socket, server_hostname=host) as ssl_socket:
            # certificate_info is a dict with lots of information about the certificate
            certificate_info = ssl_socket.getpeercert()
            exp_date_text = certificate_info['notAfter']
            # date format is like 'Sep  9 12:00:00 2016 GMT'
            return datetime.datetime.strptime(exp_date_text, r'%b %d %H:%M:%S %Y %Z')


def format_time_remaining(time_remaining):
    day_count = time_remaining.days

    if day_count >= WARN_IF_DAYS_LESS_THAN:
        return pluralise('day', day_count)

    else:
        seconds_per_minute = 60
        seconds_per_hour = seconds_per_minute * 60
        seconds_unaccounted_for = time_remaining.seconds

        hours = int(seconds_unaccounted_for / seconds_per_hour)
        seconds_unaccounted_for -= hours * seconds_per_hour

        minutes = int(seconds_unaccounted_for / seconds_per_minute)

        return '{} {} {}'.format(
            pluralise('day', day_count),
            pluralise('hour', hours),
            pluralise('min', minutes)
        )

def get_exit_code(err_count, min_days):
    code = EXIT_SUCCESS

    if err_count:
        code += EXIT_ERROR

    if min_days < WARN_IF_DAYS_LESS_THAN:
        code += EXIT_EXPIRING_SOON

    return code

def format_host_port(host, port):
    return host + ('' if port == DEFAULT_HTTPS_PORT else ':{}'.format(port))

def check_certificates(endpoints):
    context = ssl.create_default_context()
    host_port_pairs = [make_host_port_pair(endpoint) for endpoint in endpoints]

    with concurrent.futures.ThreadPoolExecutor(max_workers=WORKER_THREAD_COUNT) as executor:
        futures = {
            executor.submit(get_certificate_expiry_date_time, context, host, port):
            (host, port) for host, port in host_port_pairs
        }

        endpoint_count = len(endpoints)
        err_count = 0
        min_days = math.inf
        max_host_port_len = max([len(format_host_port(host, port)) for host, port in host_port_pairs])
        print('Checking {}...'.format(pluralise('endpoint', endpoint_count)))
        for future in concurrent.futures.as_completed(futures):
            host, port = futures[future]
            try:
                expiry_time = future.result()
            except Exception as ex:
                err_count += 1
                print('{} ERROR {}'.format(format_host_port(host, port).ljust(max_host_port_len), ex))
            else:
                time_remaining = expiry_time - datetime.datetime.utcnow()
                time_remaining_txt = format_time_remaining(time_remaining)
                days_remaining = time_remaining.days
                min_days = min(min_days, days_remaining)

                print('{} {:<5} expires in {}'.format(
                    format_host_port(host, port).ljust(max_host_port_len),
                    'WARN' if days_remaining < WARN_IF_DAYS_LESS_THAN else 'OK',
                    time_remaining_txt))

    exit_code = get_exit_code(err_count, min_days)
    sys.exit(exit_code)

if __name__ == '__main__':
    endpoints = sys.argv[1:]

    if len(endpoints):
        check_certificates(endpoints)
    else:
        print('Usage: {} <list of endpoints>'.format(sys.argv[0]))
        sys.exit(EXIT_NO_HOST_LIST)
```

