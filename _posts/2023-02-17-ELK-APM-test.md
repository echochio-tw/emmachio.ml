---
layout: post
title: ELK APM Server 連接測試
date: 2023-02-17
tags: ELK
---


```
pip install elastic-apm
pip install flask
```

如APM網址是: http://10.140.0.54:8200
 
```
from flask import Flask
from elasticapm.contrib.flask import ElasticAPM

app = Flask(__name__)

app.config['ELASTIC_APM'] = {
    'SERVICE_NAME': 'flask',
    'SECRET_TOKEN': '',
    'SERVER_URL': 'http://10.140.0.54:8200',
    'ENABLED': True,
    'ENVIRONMENT': 'production',
    'LOG_LEVEL': "debug",
}

apm = ElasticAPM(app)

@app.route('/')
def hello():
	return 'hello world!'

if __name__ == "__main__":
    app.run(host='0.0.0.0')
```
