---
layout: post
title: fail2ban + phpMyAdmin
date: 2021-09-02
tags: fail2ban, phpMyAdmin
---

php 套件列表
```
php-fpm-7.4.23-1.el7.remi.x86_64
php-tidy-7.4.23-1.el7.remi.x86_64
oniguruma5php-6.9.7.1-1.el7.remi.x86_64
php-xml-7.4.23-1.el7.remi.x86_64
php-pecl-mcrypt-1.0.4-1.el7.remi.7.4.x86_64
php-json-7.4.23-1.el7.remi.x86_64
php-mysqlnd-7.4.23-1.el7.remi.x86_64
php-bcmath-7.4.23-1.el7.remi.x86_64
php-common-7.4.23-1.el7.remi.x86_64
php-gd-7.4.23-1.el7.remi.x86_64
php-opcache-7.4.23-1.el7.remi.x86_64
php-process-7.4.23-1.el7.remi.x86_64
php-php-gettext-1.0.12-1.el7.noarch
php-tcpdf-dejavu-sans-fonts-6.2.26-1.el7.noarch
php-cli-7.4.23-1.el7.remi.x86_64
php-fedora-autoloader-1.0.1-2.el7.noarch
php-mbstring-7.4.23-1.el7.remi.x86_64
php-tcpdf-6.2.26-1.el7.noarch
php-pecl-zip-1.19.3-2.el7.remi.7.4.x86_64
php-pdo-7.4.23-1.el7.remi.x86_64
```

安裝 fail2ban

/etc/fail2ban/jail.conf
```
[DEFAULT]
ignoreip = 127.0.0.1/8 192.168.0.0/24
bantime = 600
findtime = 600
maxretry = 3

[ssh-iptables]
enabled = true
filter = sshd
action = iptables[name=SSH, port=ssh, protocol=tcp]
logpath = /var/log/secure
maxretry = 5
bantime = 86400

[phpmyadmin]
enabled = true
port = http
filter = phpmyadmin
action = iptables[name=phpmyadmin, port=http, protocol=tcp]
logpath = /var/log/phpmyadmin.log
maxretry = 3
bantime = 86400
```

抓與裝 phpmyadmin
```
wget https://files.phpmyadmin.net/phpMyAdmin/5.1.1/phpMyAdmin-5.1.1-all-languages.zip
mv phpMyAdmin-5.1.1-all-languages phpMyAdmin
mv phpMyAdmin /usr/share
```

nginx 設定
```
# cat /etc/nginx/conf.d/tttest.haha.com.conf
server {
    server_name tttest.haha.com;
    include snippets/phpMyAdmin.conf;

}
# cat /etc/nginx/conf.d/defailt.conf
server {
  # other code

  location ~ \.php$ {
    try_files $uri =404;
    fastcgi_pass unix:/run/php-fpm/www.sock;
    fastcgi_index index.php;
    fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
    include fastcgi_params;
  }
}

# cat /etc/nginx/snippets/phpMyAdmin.conf
location /phpMyAdmin {
       root /usr/share;
       index index.php index.html index.htm;
       location ~ ^/phpMyAdmin/(.+\.php)$ {
               try_files $uri =404;
               root /usr/share/;
               fastcgi_pass unix:/run/php-fpm/www.sock;
               fastcgi_index index.php;
               fastcgi_send_timeout 900;
               fastcgi_read_timeout 900;
               fastcgi_buffer_size 128K;
               fastcgi_buffers 4 128k;
               fastcgi_busy_buffers_size 256k;
               fastcgi_temp_file_write_size 256k;
               fastcgi_connect_timeout 900;
               fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
               include /etc/nginx/fastcgi_params;
       }
       location ~* ^/phpMyAdmin/(.+\.(jpg|jpeg|gif|css|png|js|ico|html|xml|txt))$ {
               root /usr/share/;
       }
}
location /phpmyadmin {
    rewrite ^/* /phpMyAdmin last;
}
```

phpMyAdmin/libraries/config.default.php 差異
```
45c45
< $cfg['AuthLog'] = 'auto';
---
> $cfg['AuthLog'] = '/var/log/phpmyadmin.log';
533c533
< $cfg['Servers'][$i]['AllowRoot'] = true;
---
> $cfg['Servers'][$i]['AllowRoot'] = false;
842c842
< $cfg['AllowArbitraryServer'] = false;
---
> $cfg['AllowArbitraryServer'] = true;
```

```
touch /var/log/phpmyadmin.log
chown nginx /var/log/phpmyadmin.log
```

/etc/php-fpm.d/www.conf 要設定
