---
layout: post
title:  crontab 用 一行 if 去跑
date: 2022-06-16
tags: grep
---

寫個 crontab 去檢查程式是否有執行

沒有就去執行

就用 at now 放背景....

用 一行 if 去跑(發現 Ubuntu 不能執行, 在 crontab 第一行加 SHELL=/bin/bash 就可)

```
*/5 * * * * ps -ef | grep mail-return-check.py|grep -v grep;[ $? == 1 ] &&  at now  <<< "/usr/bin/python3 /root/mail-return-check.py &"
```

其中 

==1 是沒有找到

==0 是有找到

用 || 不行在 crontab
```
[ "$(ps -ef | grep 'mail-return-check.py'|grep -v grep)" ] && echo 'is running' || at now  <<< "/usr/bin/python3/root/ mail-return-check.py &"
```

混著用(關掉 warning ....)
```
[ "$(ps -ef | grep 'kubectl proxy --port=8080'|grep -v grep)" ] || ([ "$(ps -ef | grep 'containerd.sock'|grep -v grep)" ] && at now <<< "sleep 10;kubectl proxy --port=8080 &" 2>&1 | fgrep -v 'warning: commands will be executed using /bin/sh')
```
