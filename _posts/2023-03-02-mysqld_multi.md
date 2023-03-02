---
layout: post
title: mysql mysqld_multi
date: 2023-03-02
tags: 單機多 mysql 
---

列表一下看裝了哪些

apt list --installed |grep mysql

```
libmysqlclient-dev/bionic-updates,bionic-security,now 5.7.41-0ubuntu0.18.04.1 amd64 [installed]
libmysqlclient20/bionic-updates,bionic-security,now 5.7.41-0ubuntu0.18.04.1 amd64 [installed]
libmysqld-dev/now 5.7.40-1ubuntu18.04 amd64 [installed,upgradable to: 5.7.41-0ubuntu0.18.04.1]
mysql-client/now 5.7.40-1ubuntu18.04 amd64 [installed,upgradable to: 5.7.41-0ubuntu0.18.04.1]
mysql-common/bionic,now 5.8+1.0.4 all [installed,automatic]
mysql-community-client/now 5.7.40-1ubuntu18.04 amd64 [installed,local]
mysql-community-server/now 5.7.40-1ubuntu18.04 amd64 [installed,local]
mysql-server/now 5.7.40-1ubuntu18.04 amd64 [installed,upgradable to: 5.7.41-0ubuntu0.18.04.1]
``` 

首先確認有 mysqld_multi 指令
```
find / -name mysqld_multi
```

確認 mysqld_multi 可執行
```
/usr/bin/mysqld_multi -version
mysqld_multi version 2.16 by Jani Tolonen
```

/etc/mysql/mysql.conf.d/mysqld.cnf
```
[mysqld_multi]
mysqld     = /usr/bin/mysqld_safe
mysqladmin = /usr/bin/mysqladmin
log        = /var/log/mysql/mysqld_multi.log

[mysqld]
#20230301
server-id=1
expire_logs_days= 7
binlog-format=row
log-bin=mysql-bin
#20230301
port=3306
pid-file        = /var/run/mysqld/mysqld.pid
socket          = /var/run/mysqld/mysqld.sock
datadir         = /var/lib/mysql
tmpdir          = /tmp
#log-error       = /var/log/mysql/error.log
# By default we only accept connections from localhost
#bind-address   = 127.0.0.1
# Disabling symbolic-links is recommended to prevent assorted security risks
symbolic-links=0

[mysqld2]
server-id=5
user            = mysql
pid-file        = /var/run/mysqld/mysqld2.pid
socket          = /var/run/mysqld/mysqld2.sock
port            = 3307
basedir         = /usr
datadir         = /var/lib/mysql2
tmpdir          = /tmp
skip-external-locking
#log_error = /var/log/mysql/error1.log
```

有 apparmor 去停用或編輯 apparmor conf 把目錄權限設定好
```
service apparmor stop
systemctl disable apparmor
mkdir /var/lib/mysql2
mysqld --initialize --datadir=/var/lib/mysql2
```

mysqld --initialize 會顯示root 密碼
```
2023-03-01T08:08:54.943150Z 0 [Warning] TIMESTAMP with implicit DEFAULT value is deprecated. Please use --explicit_defaults_for_timestamp server option (see documentation for more details).
2023-03-01T08:08:55.482674Z 0 [Warning] InnoDB: New log files created, LSN=45790
2023-03-01T08:08:55.559693Z 0 [Warning] InnoDB: Creating foreign key constraint system tables.
2023-03-01T08:08:55.574906Z 0 [Warning] No existing UUID has been found, so we assume that this is the first time that this server has been started. Generating a new UUID: 4fc27e5f-b808-11ed-b233-42010a8c003d.
2023-03-01T08:08:55.577908Z 0 [Warning] Gtid table is not ready to be used. Table 'mysql.gtid_executed' cannot be opened.
2023-03-01T08:08:55.959724Z 0 [Warning] A deprecated TLS version TLSv1 is enabled. Please use TLSv1.2 or higher.
2023-03-01T08:08:55.959767Z 0 [Warning] A deprecated TLS version TLSv1.1 is enabled. Please use TLSv1.2 or higher.
2023-03-01T08:08:55.960407Z 0 [Warning] CA certificate ca.pem is self signed.
2023-03-01T08:08:56.057088Z 1 [Note] A temporary password is generated for root@localhost: TtbmP&g%h0n(
```

改權限
```
chown -R mysql:mysql /var/lib/mysql2
```

啟動(先刪除 lock file ... 如果有)
```
rm -rf /var/run/mysqld/mysqld2.*
/usr/bin/mysqld_multi --defaults-file=/etc/mysql/mysql.conf.d/mysqld.cnf start 2
```

連進去
```
mysql -h 127.0.0.1 -u root -p -P3307
```

改密碼之類的SQL操作
```
ALTER USER 'root'@'localhost' IDENTIFIED BY 'changeme'; flush privileges;
ALTER USER 'root'@'%' IDENTIFIED BY 'changeme'; flush privileges;
```
