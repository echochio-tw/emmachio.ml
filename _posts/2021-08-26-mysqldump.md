---
layout: post
title: mysqldump show process
date: 2021-08-26
tags: mysql mysqldump
---

做一個 ramdesk 來用
```
mkdir /tmp/ramdisk
chmod 777 /tmp/ramdisk
mount -t tmpfs -o size=5G tmpfs /tmp/ramdisk
```

先到要rerestore 的查一下
```
mysql>show VARIABLES like '%max_allowed_packet%';
mysql>show VARIABLES like '%net_buffer_length';
```

dump 
```
mysqldump -uroot -ph6?aszaws6SwAws db_chio168 -e --max_allowed_packet=16777216 --net_buffer_length=16384 | pv | gzip -9 > db_chio168.sql.gz
```

restore
```
pv db_chio168.sql.gz | gunzip | mysql -h 172.31.20.11 -uroot -pvisgdob2cwd db_chio168
```

在匯出匯入時看sql 效能
```
watch -n 5 'echo "show processlist;" |  mysql -uroot -ph6?+WZ+zs#6SwtFd';
```

加快程序 renice (gzip , mysqld, mysqldump)
```
renice -10 1725
```


