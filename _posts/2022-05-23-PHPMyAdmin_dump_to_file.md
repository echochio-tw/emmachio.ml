---
layout: post
title:  PHPMyAdmin dump to file
date: 2022-05-23
tags: PHPMyAdmin mysql linux
---

紀錄一下 PHPMyAdmin to file 
```
SELECT * INTO OUTFILE '/tmp/202204result.txt' FROM `list` WHERE `message` LIKE 'CMD: SendVerificationCode , user: test%' and `send_request` > '2022-04-01 00:00:00'
```

