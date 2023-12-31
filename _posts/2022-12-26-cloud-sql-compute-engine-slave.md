---
layout: post
title: MYSQL cloud sql with compute engine slave
date: 2022-12-26
tags: GCP
---
GCP Cloud SQL MYSQL 5.7.40 

Ubuntu 18.04 slave (GCP compute engine)

下載 https://dev.mysql.com/downloads/mysql/5.7.html
```
wget https://dev.mysql.com/get/Downloads/MySQL-5.7/mysql-server_5.7.40-1ubuntu18.04_amd64.deb-bundle.tar
```

安裝時出現 

```
couldn't be accessed by user '_apt'. - pkgAcquire::Run (13: Permission denied)
```
請 apt update 再裝一次

```
tar xvf mysql-server_5.7.40-1ubuntu18.04_amd64.deb-bundle.tar
apt-get install ./libmysql*
apt-get install libmecab2
apt-get install ./mysql-community-client_5.7.40-1ubuntu18.04_amd64.deb 
apt-get install ./mysql-client_5.7.40-1ubuntu18.04_amd64.deb 
apt-get install ./mysql-community-server_5.7.40-1ubuntu18.04_amd64.deb
apt-get install ./mysql-server_5.7.40-1ubuntu18.04_amd64.deb 
```

啟動
```
systemctl start mysql.service
systemctl enable mysql.service
```

MYSQL root 密碼是 localsqlpass

cloud SQL root 密碼是 cloudsqlpass

cloud SQL rep 密碼是 密碼是

cloud SQL 內網IP 10.3.7.29

才能只行下面 sell (不同自行修改 用 sed 換 10.3.7.29, cloudsqlpass, localsqlpass, loveme)

```
#!/bin/bash
USER="root"
PASSWORD="cloudsqlpass"
PASSWORD2="localsqlpass"

cat <<EOF > /etc/mysql/mysql.conf.d/mysqld.cnf
[mysqld]
pid-file        = /var/run/mysqld/mysqld.pid
socket          = /var/run/mysqld/mysqld.sock
datadir         = /var/lib/mysql
log-error       = /var/log/mysql/error.log
# By default we only accept connections from localhost
bind-address    = 127.0.0.1
server-id=2
gtid_mode=ON
enforce_gtid_consistency=ON
log_slave_updates=ON
replicate-ignore-db=mysql
binlog-format=ROW
log_bin=mysql-bin
expire_logs_days=1
read_only=ON
# Disabling symbolic-links is recommended to prevent assorted security risks
symbolic-links=0
EOF

systemctl stop mysql
systemctl start mysql

mysql -uroot -p$PASSWORD2 -e "show databases" | grep -v Database | grep -v mysql| grep -v information_schema| gawk '{print "drop database `" $1 "`;select sleep(0.1);"}' | mysql -uroot -p$PASSWORD2

# 下面兩行是刪除 rep user 與建立 rep user 如有了請自行註解或 自行改 rep 密碼
mysql -u $USER -p$PASSWORD -h 10.3.7.29 -e "DROP USER 'rep'@'%';"
mysql -u $USER -p$PASSWORD -h 10.3.7.29 -e "CREATE USER 'rep'@'%' IDENTIFIED BY 'loveme';GRANT REPLICATION SLAVE ON *.* TO 'rep'@'%';"

databases=`mysql -u $USER -p$PASSWORD -h 10.3.7.29 -e "SHOW DATABASES;" | tr -d "| " | grep -v Database`
mysql -u $USER -p$PASSWORD -h 10.3.7.29 -e "FLUSH TABLES WITH READ LOCK;show master status;"
for db in $databases; do
    if [[ "$db" != "information_schema" ]] && [[ "$db" != "performance_schema" ]] && [[ "$db" != "mysql" ]] && [[ "$db" != _* ]] ; then
        mysqldump -u $USER --set-gtid-purged=off -p$PASSWORD -h 10.3.7.29 --databases $db | mysql -u $USER -p$PASSWORD
    fi
done
lock=`mysql -u $USER -p$PASSWORD -h 10.3.7.29 -e "show master status;" | grep -v File |awk '{print $(NF)'}`
# 下面這行是請自行改 rep 密碼
mysql -u $USER -p$PASSWORD2 -e "CHANGE MASTER TO MASTER_HOST='10.3.7.29', MASTER_USER='rep',MASTER_PASSWORD='loveme', MASTER_AUTO_POSITION=1;"
mysql -u $USER -p$PASSWORD2 -e "stop slave;"
mysql -u $USER -p$PASSWORD2 -e "reset master;"
mysql -u $USER -p$PASSWORD2 -e "set global gtid_purged='$lock';"
mysql -u $USER -p$PASSWORD2 -e "start slave;"
mysql -u $USER -p$PASSWORD -h 10.3.7.29 -e "UNLOCK TABLES;"
mysql -u $USER -p$PASSWORD2 -e "show slave status\G"
```
用 sed 換 10.3.7.29, cloudsqlpass, localsqlpass, loveme
