---
layout: post
title:  flask-appbuilder 配合 ajax 顯示 HTML <table> 表格
date:   2022-05-07
categories: flask-appbuilder
---
紀錄一下 javascript tr append 方法
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
		<!--Start breadcrumbs -->
		<div class="row">
			<div class="col s10 m6 l6">
				<h4 class="breadcrumbs-title">请输入 id 找资讯</h4>
			</div>
		</div>
		<!--End breadcrumbs -->
		<!-- START searchBar -->
		<div class="row">
			<form id="telsent" novalidate="novalidate">
				<div class="row">
					<div class="input-field col s6 offset-s1">
						<i class="material-icons prefix">mode_edit</i>
						<textarea id="message" class="materialize-textarea" name="msgid" placeholder="" cols="50" rows="1" ></textarea>
						<label for="message" class="active">请输入 id </label>
						<div id="card-alert" class="card gradient-45deg-light-blue-cyan">
							<div class="card-content white-text">
								<p>
									<i class="material-icons">info_outline</i> &nbsp;&nbsp; ID请自行记录下来
									<p></p>
									<i class="material-icons">info_outline</i> &nbsp;&nbsp; 要5-10分后再做MID查询
									<p></p>
							</div>
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
<table id="result" class="table table-striped table-bordered">
	<thead>
		<tr>
			<th>机号</th>
			<th>平台返回的id</th>
			<th>返回的时间</th>
			<th>状态码</th>
			<th>接口服务器返回的时间</th>
			<th>次号</th>
		</tr>
	</thead>
	<tbody>
			<div class="editview-product-box"> </div>
	</tbody>
</table>
<script src="https://ajax.googleapis.com/ajax/libs/jquery/3.3.1/jquery.min.js"></script> <!-- 引入 jQuery -->
<script type="text/javascript">
$SCRIPT_ROOT = {{ request.script_root|tojson|safe }};
$(document).ready(function() {
	$("#submitID").click(function() { //ID 为 submitID 的按钮被点击时
		$.ajax({
			type: "POST", //传送方式
			url: $SCRIPT_ROOT + "/api/Getid/", //传送目的地
			dataType: "json", //资料格式
			data: { //传送资料
				message: $("#message").val() //表单字段 ID message
			},
			beforeSend: function () {
				$("#submitID").val("处理中");
				$("#submitID").css("background-color","aqua");
				$("#submitID").attr({ disabled: "disabled" });
			},
			success: function(data) {
				onSuccess(data);
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

function onSuccess(data) {
	let html = "";
	$.each(data, function(k, v){
	  var mobile = $("<td></td>").append(v.data);
	  var msgid = $("<td></td>").append(v.msgid);
	  var reportTime = $("<td></td>").append(v.reportTime);
	  var status = $("<td></td>").append(v.status);
	  var notifyTime = $("<td></td>").append(v.notifyTime);
	  var batchSeq = $("<td></td>").append(v.batchSeq);
	  $("<tr></tr>").append(data).append(msgid).append(reportTime).append(status).append(notifyTime).append(batchSeq).appendTo($("table tbody"));
	})
	$('.editview-radio-box').html(html);
  }
</script>
{% endblock %}

```