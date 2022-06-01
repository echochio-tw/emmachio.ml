---
layout: post
title:  flask-appbuilder 打包成 docker
date: 2022-06-01
tags: grep
---
幾個重要檔案
```
.
├── Dockerfile
├── app
│   ├── __init__.py
│   ├── models.py
│   ├── templates
│   │   └── 404.html
│   ├── translations
│   │   └── pt
│   │       └── LC_MESSAGES
│   │           ├── messages.mo
│   │           └── messages.po
│   └── views.py
├── app.db
├── babel
│   ├── babel.cfg
│   └── messages.pot
├── config.py
├── main.py
├── requirements.txt
├── run.py
└── uwsgi.ini
```

建立 用正常方法
```
python -m pip install flask-appbuilder
# flask fab create-app
Your new app name: app
Your engine type, SQLAlchemy or MongoEngine (SQLAlchemy, MongoEngine) [SQLAlchemy]:
Something went wrong [Errno 13] Permission denied: 'Flask-AppBuilder-Skeleton-master' -> 'app'
Try downloading from https://github.com/dpgaspar/Flask-AppBuilder-Skeleton/archive/master.zip
# cd Flask-AppBuilder-Skeleton-master/
# flask fab create-admin
Username [admin]:
User first name [admin]:
User last name [user]:
Email [admin@fab.org]:
Password:
Repeat for confirmation:
/usr/local/lib/python3.8/dist-packages/flask_sqlalchemy/__init__.py:872: FSADeprecationWarning: SQLALCHEMY_TRACK_MODIFICATIONS adds significant overhead and will be disabled by default in the future.  Set it to True or False to suppress this warning.
................................
```

在 Flask-AppBuilder-Skeleton-master 目錄下有
```
.
├── Dockerfile
├── app
│   ├── __init__.py
│   ├── models.py
│   ├── templates
│   │   └── 404.html
│   ├── translations
│   │   └── pt
│   │       └── LC_MESSAGES
│   │           ├── messages.mo
│   │           └── messages.po
│   └── views.py
├── app.db
├── babel
│   ├── babel.cfg
│   └── messages.pot
├── config.py
├── main.py
├── requirements.txt
├── run.py
└── uwsgi.ini
```

Dockerfile
```
FROM tiangolo/uwsgi-nginx-flask:python3.8

RUN apt update
RUN yes | apt install vim

COPY ./ /app
WORKDIR /app

ENV STATIC_URL /app/static
ENV STATIC_PATH /var/www/app/static

RUN pip install -r /app/requirements.txt
```

requirements.txt
```
apispec==3.3.2
attrs==21.4.0
Babel==2.10.1
click==8.1.3
colorama==0.4.4
dnspython==2.2.1
docopt==0.6.2
email-validator==1.2.1
Flask==2.1.2
Flask-AppBuilder==4.1.0
Flask-Babel==2.0.0
Flask-JWT-Extended==4.4.0
Flask-Login==0.6.1
Flask-SQLAlchemy==2.5.1
Flask-WTF==0.15.1
greenlet==1.1.2
gunicorn==19.3.0
idna==3.3
importlib-metadata==4.11.3
importlib-resources==5.7.1
itsdangerous==2.1.2
Jinja2==3.1.2
jsonschema==4.4.0
MarkupSafe==2.1.1
marshmallow==3.15.0
marshmallow-enum==1.5.1
marshmallow-sqlalchemy==0.26.1
packaging==21.3
prison==0.2.1
PyJWT==2.3.0
pyparsing==3.0.8
pyrsistent==0.18.1
python-dateutil==2.8.2
pytz==2022.1
PyYAML==6.0
six==1.16.0
SQLAlchemy==1.4.36
SQLAlchemy-Utils==0.38.2
Werkzeug==2.1.2
WTForms==2.3.3
yarg==0.1.9
zipp==3.8.0
```

main.py
```
from flask import Flask
from app import app

if __name__ == "__main__":
    app.run(debug=True, port=80)
```

uwsgi.ini
```
[uwsgi]
module = main
callable = app
master = true
```
建立 image
```
docker build -t myweb .
```

docker run 把服務起起來
```
docker run -d -p 8000:80 --name=myweb myweb
```

打上你的 http://你的IP:8000/ 就完成囉

可以直接 git 抓下來打包 Docker

https://github.com/echochio-tw/flask-appbuilder-docker.git
