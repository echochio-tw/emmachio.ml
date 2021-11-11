---
layout: post
title: mysql 建立 slave
date: 2021-09-29
tags: mysql
---

master my.cnf
```
server_id = 1
log-basename=master
log-bin=/var/log/mysql/mariadb-bin #(開啟binlog，並指定存檔名稱為 mariadb-bin.xxxxxx)
expire_logs_days = 5
binlog-do-db=replica_db #(要同步的資料庫名稱，有指定多個db就重覆多行binlog-do-db=dbname，若這行沒有，就是全部同步)
```

master db 設定
```
mysql -uroot -p
mysql> show variables like 'log_bin';	 #查看binlog功能有無開啟
mysql> show variables like "server_id";  #查看server_id是不是剛剛設置好的
mysql> grant replication slave on *.* to 'repuser'@'10.170.0.7' identified by 'repl'; (10.170.0.7 為 Slave IP)
mysql> flush privileges;
mysql> FLUSH TABLES WITH READ LOCK;
mysql> show grants for repliuser1@'10.170.0.7'; 
mysql> show master status \G (查看目前索引位置)
```

master 倒資料
```
mysqldump --all-databases --user=root --password --master-data > masterdatabase.sql

scp masterdatabase.sql user@ip:~
```

master 要等 slave 做完後
```
mysql> UNLOCK TABLES; #要slave 好了才UNLOCK
```

slave my.cnf
```
server_id = 2 #(Slave設定2之後)
log-bin=/var/log/mysql/mariadb-bin #(開啟binlog，並指定存檔名稱為 mariadb-bin.xxxxxx)
relay-log = /var/log/mysql #(接收Master傳送來的binary內容) 若這行用 內定 /var/log/mysql
replicate-do-db=replica_db #(要同步的資料庫名稱，有指定多個db就重覆多行replicate-do-db=dbname，若這行沒有，就是全部同步)
```

slave 倒資料
```
# mysql -u root -p
mysql> source masterdatabase.sql
```

slave 重啟 MySQL
```
/usr/bin/mysqladmin -u root -S /var/lib/mysql/mysql.sock shutdown
/usr/bin/mysqld_safe --defaults-file=/etc/my.cnf 2>&1 > /dev/null &
```

檢查備份檔的 master_log_file & master_log_pos
```
cat masterdatabase.sql | grep MASTER_LOG_FILE
```

連 master
```
mysql -uroot -p
mysql> change master to master_host='MasterIP', master_port=3306, master_user='repuser', master_password='repl', master_log_file='mariadb-bin.000017', master_log_pos=0;
mysql> start slave;
mysql> show slave status \G;
# 檢查同步狀態 Slave_IO_Running 
               Slave_SQL_Running 需要為 Yes

```

```
# CHANGE MASTER TO
# -> MASTER_HOST='10.170.0.3',	         master IP
# -> MASTER_PORT=3306,		             DB port
# -> MASTER_USER='repuser',		         sync account
# -> MASTER_PASSWORD='password',   	     sync account password
# -> MASTER_LOG_FILE=' mariadb-bin.000017',  Binlog id (看 show master status 是多少?)
# -> MASTER_LOG_POS=0;		             Binlog 目前位置是多少(設0會自動)
```

在master DB 顯示 slave 資訊
```
SELECT * FROM information_schema.PROCESSLIST AS p WHERE p.COMMAND = 'Binlog Dump' \G
show slave hosts \G
```

slave 從 MySQL 實例中清除從屬信息的最快的方法 (沒測試過)
```
1 加 skip-slave-start 
/etc/my.cnf
[mysqld]
skip-slave-start 

2. service mysql stop

3.
rm -f /var/lib/mysql/master.info /var/lib/mysql/master-relay-bin.*
service mysql start

4.
移除 skip-slave-start from /etc/my.cnf
```		  
