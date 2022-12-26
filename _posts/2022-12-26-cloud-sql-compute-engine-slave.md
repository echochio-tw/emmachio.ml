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

安裝
```
tar xvf mysql-server_5.7.40-1ubuntu18.04_amd64.deb-bundle.tar
apt-get install ./libmysql*
apt-get install libmecab2
apt-get install ./mysql-community-client_5.7.40-1ubuntu18.04_amd64.deb 
apt-get install ./mysql-client_5.7.40-1ubuntu18.04_amd64.deb 
apt-get install ./mysql-community-server_5.7.40-1ubuntu18.04_amd64.deb
apt update
apt-get install ./mysql-server_5.7.40-1ubuntu18.04_amd64.deb 
```

啟動
```
systemctl start mysql.service
systemctl enable mysql.service
```

MYSQL root 密碼設定 loveme

cloud SQL root 也是 loveme

cloud SQL 內網IP 10.3.7.29 

才能只行下面 sell (不同自行修改)
```
#!/bin/bash
USER="root"
PASSWORD="loveme"

databases=`mysql -u $USER -p$PASSWORD -e "CREATE USER 'rep'@'%' IDENTIFIED BY 'loveme';GRANT REPLICATION SLAVE ON *.* TO 'rep'@'%';"

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

databases=`mysql -u $USER -p$PASSWORD -h 10.3.7.29 -e "SHOW DATABASES;" | tr -d "| " | grep -v Database`
mysql -u $USER -p$PASSWORD -h 10.3.7.29 -e "FLUSH TABLES WITH READ LOCK;show master status;"
for db in $databases; do
    if [[ "$db" != "information_schema" ]] && [[ "$db" != "performance_schema" ]] && [[ "$db" != "mysql" ]] && [[ "$db" != _* ]] ; then
        mysqldump -u $USER -p$PASSWORD -h 10.3.7.29 --databases $db | mysql -u $USER -p$PASSWORD
    fi
done
lock=`mysql -u $USER -p$PASSWORD -h 10.3.7.29 -e "show master status;" | grep -v File |awk '{print $(NF)'}`
mysql -u $USER -p$PASSWORD -e "CHANGE MASTER TO MASTER_HOST='10.3.7.29', MASTER_USER='rep',MASTER_PASSWORD='loveme', MASTER_AUTO_POSITION=1;"
mysql -u $USER -p$PASSWORD -e "stop slave;"
mysql -u $USER -p$PASSWORD -e "reset master;"
mysql -u $USER -p$PASSWORD -e "set global gtid_purged='$lock';"
mysql -u $USER -p$PASSWORD -e "start slave;"
mysql -u $USER -p$PASSWORD -h 10.3.7.29 -e "UNLOCK TABLES;"
mysql -u $USER -p$PASSWORD -e "show slave status\G"
```
