---
layout: post
title: flask error show (try except)
date: 2021-12-11
tags: flask try except
---

我的 telegram bot 呼叫 google app scripts 會一直不斷的重複跳出訊息

原來是 google app scripts 有呼叫我的 flask_uwsgi_app 去取資料

但是 flask_uwsgi_app 其中的 request 取不到資料原來是網站掛了

flask 掛了 telegram bot 就會一直不斷的重複跳出訊息... :(

改一下 加 errorhandler
```
from flask import Flask, request, abort, jsonify
from werkzeug.exceptions import HTTPException
import traceback
import sys
import requests
		
app = Flask(__name__)

@app.errorhandler(Exception)
def handle_error(e):
    code = 500
    if isinstance(e, HTTPException):
        code = e.code
	# call telegram bot show error
    return jsonify(error=str(e)), code

@app.route('/')
def index():
    #return (requests.get('https://jsonplaceholder.typicode.com/todos').text)
    return (requests.get('https://errorjsonplaceholder.typicode.com/todos').text)

app.run(port=1234)
```
