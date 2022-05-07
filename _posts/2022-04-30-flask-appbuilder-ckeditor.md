---
layout: post
title:  flask-appbuilder 配合 ckeditor 發 api mail
date:   2022-04-30 19:51:18 +0800
categories: flask-appbuilder
---
需要把 登入的帳號去過濾 DB 這就紀錄一下片段.....
```
class SentmailModelView(BaseView):
    default_view = 'mailto'
    @expose('/mailto/')
    @has_access
    def mailto(self):
        return self.render_template('mail.html', base_template=appbuilder.base_template, appbuilder=appbuilder)
```
mail.html
```
<style>
        @import url(https://fonts.googleapis.com/css?family=Roboto:300,300i,400,400i,700|Montserrat:400,400i,500);
        @import url(https://fonts.googleapis.com/icon?family=Material+Icons);
        #work-collections .collection-item.avatar {
            height: auto;
            padding-top: 22px;
        }
        
        #content .container .row {
            margin-bottom: 0;
        }
    </style>
    {% extends "appbuilder/base.html" %} {# Inherit from base.html #}
 
{% block content %}
    <div id="breadcrumbs-wrapper">
        <!-- Search for small screen -->
        <div class="container">
            <!-- START searchBar -->
            <div class="row">
                <form id="telsent" novalidate="novalidate">
                    <div class="row">
                        <div class="input-field col s6 offset-s1">
                            <i class="material-icons prefix">mode_edit</i>
                            <label for="message" class="active">信件内容</label>
                            <textarea id="message" class="materialize-textarea" name="message" placeholder="ex: 549" maxlength="155"></textarea>
                        </div>
                    </div>
                    <script src="https://cdn.ckeditor.com/4.18.0/standard/ckeditor.js"></script>
                    <script>
                            CKEDITOR.replace( 'message');
                    </script>
                    <div class="row">
                        <div class="input-field col s6 offset-s1">
                            <i class="material-icons prefix">mode_edit</i>
                            <textarea id="subject" class="materialize-textarea" name="subject" placeholder="ex: test mail"></textarea>
                            <label for="message" class="active">邮件主旨</label>
                            <div id="card-alert" class="card gradient-45deg-light-blue-cyan">
                            </div>
                        </div>
                        <div class="input-field col s6 offset-s1">
                            <i class="material-icons prefix">mode_edit</i>
                            <textarea id="mailaddr" class="materialize-textarea" name="mailaddr" placeholder="ex: test@gmail.com"></textarea>
                            <label for="message" class="active">收信由箱</label>
                            <div id="card-alert" class="card gradient-45deg-light-blue-cyan">
                            </div>
                        </div>
                    </div>
                    <div class="row">
                        <div class="input-field col s6 offset-s1">
                            <input type="submit" value="发送" id="submitID">
                            <input type="reset" value="重置" id="ResetID">
                        </div>
                    </div>
                </form>
            </div>
            <hr>
        </div>
    </div>
    <p id="result"></p> <!-- 显示回传资料 -->
    <script src="https://ajax.googleapis.com/ajax/libs/jquery/3.3.1/jquery.min.js"></script> <!-- 引入 jQuery -->
    <script type="text/javascript">
    $SCRIPT_ROOT = {{ request.script_root|tojson|safe }};
    function CKupdate(){
        for(instance in CKEDITOR.instances)
            CKEDITOR.instances[instance].updateElement();
    };
    $(document).ready(function() {
        $("#submitID").click(function() { //ID 为 submitID 的按钮被点击时
            CKupdate();
            var sendData = $('#submitID').serialize();
            $.ajax({
                type: "POST", //传送方式
                url: $SCRIPT_ROOT + "/api/SendMAIL/", //传送目的地
                dataType: "json", //资料格式
                data: { //传送资料
                    subject:  $("#subject").val(), //表单字段 ID subject
                    mailaddr: $("#mailaddr").val(), //表单字段 ID mailaddr
                    message: $("#message").val() //表单字段 ID message
                },
                beforeSend: function () {
                    $("#submitID").val("处理中");
                    $("#submitID").css("background-color","aqua");
                    $("#submitID").attr({ disabled: "disabled" });
                },
                success: function(data) {
                    const myJSON = JSON.stringify(data);
                    $("#result").html( '<font color="#007500">回应 : <font color="#0000ff"><br />'+myJSON );
                },
                complete: function () {
                    $('#loading-image').remove();
                    $("#submitID").val("发送");
                    $("#submitID").css("background-color","red");
                    $("#ResetID").css("background-color","red");
                    $("#ResetID").attr({ disabled: "disabled" });
                },
                error: function(jqXHR) {
                    $("#datainput")[0].reset(); //重设 ID 为 datainput 的 form (表单)
                    $("#result").html('<font color="#ff0000">发生错误：' + jqXHR.status + '</font>');
                }
            })
        })
    });
    </script>
    {% endblock %}
```