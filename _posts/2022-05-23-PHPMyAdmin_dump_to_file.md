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

簡單的導入
```
LOAD DATA INFILE '/tmp/mydata.txt' INTO TABLE PerformanceReport;
```

cmd導入
```
mysqlimport -u root -ptmppassword --local test employee.txt
test.employee: Records: 3  Deleted: 0  Skipped: 0  Warnings: 0
```

csv 忽略第一行 
```
LOAD DATA INFILE 'c:/tmp/discounts.csv' 
INTO TABLE discounts 
FIELDS TERMINATED BY ',' 
ENCLOSED BY '"'
LINES TERMINATED BY '\n'
IGNORE 1 ROWS 
(title,@expired_date,amount)
SET expired_date = STR_TO_DATE(@expired_date, '%m/%d/%Y');
```

這要試試應該沒問題
```
LOAD DATA INFILE '/tmp/filename.csv' replace INTO TABLE  (field1,field2,field3);
```