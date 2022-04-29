---
layout: post
title: flask-appbuilder user 資訊去過濾 DB表單
date: 2022-04-29
tags: flask-appbuilder
---


需要把 登入的帳號去過濾 DB 這就紀錄一下片段.....
```
from flask_appbuilder.models.sqla.interface import SQLAInterface
from flask_appbuilder.models.sqla.filters import FilterEqualFunction
from .models import Member,Message_list
from flask import g
.....
.....
def get_user():
    return g.user.username
......
......
class MessagelistcustomerModelView(ModelView):
    datamodel = SQLAInterface(Message_list)
    base_permissions = ['can_list','can_show']
    column_default_sort = ('returnlog', True)
    base_filters = [['user', FilterEqualFunction, get_user]]
    base_order = ('id','desc')

```

由登入號去DB找資訊配合 TWIG 顯示出來
```
class SelfReceiverModelView(BaseView):
    default_view = 'SelfReceiver'
    @expose('/SelfReceiver/')
    @has_access
    def SelfReceiver(self):
        u = get_user()
        a = db.session.query(Member).filter_by(user = u).all()
        self.update_redirect()
		if(a is None):
            return self.render_template('nouser.html', base_template=appbuilder.base_template, appbuilder=appbuilder)
        usrinfo=[]#有可能很多筆所以用 list
        for ainfo in a:
            usrinfo.append(ainfo.userinfo)
        return self.render_template('show-user.html', base_template=appbuilder.base_template, appbuilder=appbuilder, userinfo=userinfo)
```
