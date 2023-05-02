---
layout: post
title: python logging
date: 2019-10-28
tags: python
---

寫了一大堆 python 常用 log + wintail 去看執行的資訊 紀錄一下

```
import logging
logging.basicConfig(filename='D:/run.log',level=logging.INFO,format='%(asctime)s - %(name)s - %(levelname)s - %(message)s')
logging.info('finish multi sim bix1 close')
```

log 不斷變大需要 Rotating
```
import logging
from logging.handlers import RotatingFileHandler
logging.basicConfig(format='%(asctime)s - %(lineno)d - %(funcName)s - %(levelname)s - %(message)s', level=logging.DEBUG)
logger = logging.getLogger("mylog")
formatter = logging.Formatter('%(asctime)s - %(lineno)d - %(funcName)s - %(levelname)s - %(message)s')
handler = RotatingFileHandler('mylog.log', maxBytes=20000, backupCount=5)
handler.setFormatter(formatter)
handler.setLevel(logging.DEBUG)
logger.addHandler(handler)
for _ in range(10000):
    logger.debug('Hello, world!')
```
