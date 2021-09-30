---
layout: post
title: centos7 install zabbix server
date: 2018-09-28
tags: zabbix
---

ssh 換 port

```
sed -i 's/#Port 22/Port 50022/g' /etc/ssh/sshd_config
systemctl restart sshd
```

apache 換 port
```
yum -y install httpd
sudo sed -i 's/Listen 80/Listen 50080/g' /etc/httpd/conf/httpd.conf
systemctl start httpd
systemctl enable httpd
```

安裝 php 7.2
```
yum -y install epel-release
sudo rpm -Uvh https://mirror.webtatic.com/yum/el7/webtatic-release.rpm
yum -y install mod_php72w php72w-cli php72w-common php72w-devel php72w-pear php72w-gd php72w-mbstring php72w-mysql php72w-xml php72w-bcmath

rm -rf /etc/php.ini
cat << 'EOF' >> /etc/php.ini
max_execution_time = 600
max_input_time = 600
memory_limit = 256M
post_max_size = 32M
upload_max_filesize = 16M
date.timezone = Asia/Shanghai
EOF

cd /var/lib/php/
chown -R root:apache ./*
systemctl restart httpd
```

安裝 mysql
```
yum -y install mariadb-server
systemctl start mariadb
systemctl enable mariadb
mysql_secure_installation
mysql -u root -p
create database zabbix character set utf8 collate utf8_bin;
grant all privileges on zabbix.* to zabbix@'localhost' identified by 'mypassword'; ; 
flush privileges;
```

安裝 zabbix_server & zabbix-agent
```
yum -y install https://repo.zabbix.com/zabbix/5.0/rhel/7/x86_64/zabbix-release-5.0-1.el7.noarch.rpm
yum -y install zabbix-server-mysql zabbix-agent2 zabbix-get
yum-config-manager --enable zabbix-frontend
yum -y install centos-release-scl
yum -y install zabbix-web-mysql-scl zabbix-apache-conf-scl
cd /usr/share/doc/zabbix-server-mysql*
zcat /usr/share/doc/zabbix-server-mysql*/create.sql.gz | mysql -uzabbix -p mypassword
```

config zabbix_server
```
vi /etc/zabbix/zabbix_server.conf
DBHost=localhost
DBPassword=mypassword
```

啟動 zabbix_server
```
systemctl start zabbix-server
systemctl enable zabbix-server
systemctl status zabbix-server
```

設定 zabbix_agent
```
vi /etc/zabbix/zabbix_agentd2.conf
 Server=172.16.1.2
 ServerActive=172.16.1.2
 Hostname=dy-zabbix
```

啟動 zabbix-agent 
``` 
systemctl start zabbix-agent2
systemctl enable zabbix-agent2
systemctl status zabbix-agent2
```

設定自訂 agent
```
cd /etc/zabbix/zabbix_agentd2.d/

vi userparameter_mysql.conf
UserParameter=login-user,who|wc -l
UserParameter=mysqld-connect,netstat -natp |grep mysql |wc -l
```

```
systemctl restart zabbix-agent2.service
```

Server 端取資訊
```
zabbix_get -s 172.16.1.21 -p 10050 -k 'system.cpu.load[all,avg5]'
zabbix_get -s 172.16.1.21 -p 10050 -k "login-user"
zabbix_get -s 172.16.1.21 -p 10050 -k "mysqld-connect"
```



 
 
