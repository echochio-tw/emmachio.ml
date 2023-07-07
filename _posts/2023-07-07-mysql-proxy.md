---
layout: post
title: mysql-proxy
date: 2023-07-07
tags: mysql read write
---

目標：
```
proxy listening on port 127.0.0.1:3366
read/write backend: 127.0.0.1:3306
read-only backend: 127.0.0.1:3307
```

```
wget https://cdn.mysql.com/archives/mysql-proxy/mysql-proxy-0.8.5-linux-glibc2.3-x86-64bit.tar.gz
tar zxvf mysql-proxy-0.8.5-linux-glibc2.3-x86-64bit.tar.gz
```

```
cd mysql-proxy-0.8.5-linux-glibc2.3-x86-64bit
cp -ra . /usr/local/
mkdir /usr/local/mysql-proxy
mkdir conf
cd conf
```

請依照需求改 以下檔案 ..... 
```
cat <<EOF >>mysql-proxy.conf
[mysql-proxy]
daemon=true
user=root
keepalive=true
plugins=proxy,admin
log-level=debug
log-file=/var/log/mysql-proxy.log
proxy-address=127.0.0.1:3366
proxy-backend-addresses=127.0.0.1:3306
proxy-read-only-backend-addresses=127.0.0.1:3307
proxy-lua-script=/usr/local/share/doc/mysql-proxy/rw-splitting.lua
admin-address=127.0.0.1:4040
admin-username=root
admin-password=admin
admin-lua-script=/usr/local/lib/mysql-proxy/lua/admin.lua
EOF
```

```
chmod 0660 /usr/local/conf/mysql-proxy.conf
```

MySQL Proxy會檢測用戶端串連, 當串連沒有超過min_idle_connections預設值時, 

不會進行讀寫分離預設最小4個(最大8個)以上的用戶端串連才會實現讀寫分離, 現改為最小1個最大2個，

便於讀寫分離的測試，生產環境中，可以根據實際情況進行調整。

vi /usr/local/share/doc/mysql-proxy/rw-splitting.lua
```
 min_idle_connections = 1
 max_idle_connections = 2,
```

```
mkdir /usr/local/lib/mysql-proxy/lua/
/usr/local/bin/mysql-proxy -V
```

 ls -l /usr/local/lib/mysql-proxy/lua/
 ```
total 372
-rw-r--r-- 1 7161 wheel   2741 Aug 19  2014 admin.lua
-rwxr-xr-x 1 7161 wheel  30890 Aug 19  2014 chassis.so
-rwxr-xr-x 1 7161 wheel  14556 Aug 19  2014 glib2.so
-rwxr-xr-x 1 7161 wheel  35115 Aug 19  2014 lfs.so
-rwxr-xr-x 1 7161 wheel  98051 Aug 19  2014 lpeg.so
-rwxr-xr-x 1 7161 wheel 169365 Aug 19  2014 mysql.so
-rwxr-xr-x 1 7161 wheel  14492 Aug 19  2014 posix.so
drwxr-xr-x 2 7161 wheel   4096 Aug 19  2014 proxy
```
啟動
```
/usr/local/bin/mysql-proxy --defaults-file=/usr/local/conf/mysql-proxy.conf
```

log
```
tail -f /var/log/mysql-proxy.log
```

```
2023-07-07 14:42:13: (debug) chassis-unix-daemon.c:157: waiting for 7547
2023-07-07 14:42:13: (debug) chassis-unix-daemon.c:121: we are the child: 7547
2023-07-07 14:42:13: (critical) plugin proxy 0.8.5 started
2023-07-07 14:42:13: (critical) plugin admin 0.8.5 started
2023-07-07 14:42:13: (debug) max open file-descriptors = 65535
2023-07-07 14:42:13: (message) proxy listening on port 127.0.0.1:3366
2023-07-07 14:42:13: (message) added read/write backend: 127.0.0.1:3306
2023-07-07 14:42:13: (message) added read-only backend: 127.0.0.1:3307
2023-07-07 14:42:13: (message) admin-server listening on port 127.0.0.1:4040
2023-07-07 14:42:13: (debug) now running as user: root (0/0)

```



