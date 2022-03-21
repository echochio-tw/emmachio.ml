---
layout: post
title: flask jsonify 
date: 2022-03-21
tags: python
---


json 在 flask 用 jsonify 顯示中文
```
app.config['JSON_AS_ASCII'] = false #顯示中文
```

pymysql 用 pymysql.cursors.DictCursor 把 MYSQL 轉成 Dic
```
cursor = connection.cursor(pymysql.cursors.DictCursor) #设定 Dic 指标
```

測試的code 

```
from flask import Flask, jsonify
import pymysql

class Book(object):
    def __init__(self):
        self.conn = pymysql.connect(db="test") #连DB
        cursors = self.connection.cursor(pymysql.cursors.DictCursor) #设定 Dic 指标
    def __del__(self):
        self.cursors.close()
        self.conn.close()
    def get_info(self):
        sql = 'select * from .....'
        self.cursor.execute(sql)
        data=[]
        for t in self.cursor.fetchall():
            data.append(t)
        return(data)

app = Flask(__name__)
app.config['JSON_AS_ASCII'] = False #显示中文
@app.route('/',methods=['GET','POST']) #路由
def hello():
    return(jsonify(Book.get_info()))

```
