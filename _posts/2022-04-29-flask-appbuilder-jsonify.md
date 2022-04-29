---
layout: post
title: flask-appbuilder jsonify 送中文字
date: 2022-04-29
tags: flask-appbuilder
---

如果 config.py 設定
```
JSON_AS_ASCII = False
```
jsonify 送中文字 就沒問題了
但是發現DB撈出中文字會
```
  File "/home/flask/app/message/env/lib/python3.6/site-packages/urllib3/connectionpool.py", line 392, in _make_request
    conn.request(method, url, **httplib_request_kw)
  File "/usr/lib64/python3.6/http/client.py", line 1254, in request
    self._send_request(method, url, body, headers, encode_chunked)
  File "/usr/lib64/python3.6/http/client.py", line 1299, in _send_request
    body = _encode(body, 'body')
  File "/usr/lib64/python3.6/http/client.py", line 171, in _encode
    (name.title(), data[err.start:err.end], name)) from None
UnicodeEncodeError: 'latin-1' codec can't encode characters in position 72-92: Body ('您的優惠已送出，請重登錄) is not valid Latin-1. Use body.encode('utf-8') if you want to send it encoded in UTF-8.
```

要改成 json.dumps 再用 response json_string 去回應
```
from flask import json,Response 
@app.route("/")
def hello():
    my_list = []
    my_list.append(u'中文字')
    data = { "result" : my_list}
    json_string = json.dumps(data,ensure_ascii = False)
    #creating a Response object to set the content type and the encoding
    response = Response(json_string,content_type="application/json; charset=utf-8" )
    return response
```
