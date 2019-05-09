---
title: jQuery-Validate-layer表单校验
date: 2088-01-01 08:00:01
tags: jQuery
categories: 前端
---

***用jQuery Validate layer插件实现好看的表单校验***

![jQuery-Validate-layer表单校验效果](jQuery-Validate-layer表单校验效果.png 'jQuery-Validate-layer表单校验效果')


### 完整代码实现
```javascript
<html>
	<head>
	    <title>valdate</title>
		<meta charset="UTF-8">
		<link rel="stylesheet" type="text/css" href="http://layui.hcwl520.com.cn/layui-v2.4.5/css/layui.css?v=201811010202"/>
		
	</head>
	
	<body style="padding-top: 20px">
	<form class="layui-form" id="signupForm" action="">
		<div class="layui-form-item">
			<label class="layui-form-label">名字</label>
			<div class="layui-input-inline">
				<input type="text" id="firstname" name="firstname" autocomplete="off" placeholder="请输入名字" class="layui-input">
			</div>
		</div>
		<div class="layui-form-item">
			<label class="layui-form-label">姓氏</label>
			<div class="layui-input-inline">
				<input type="text" id="lastname" name="lastname" autocomplete="off" placeholder="请输入姓氏" class="layui-input">
			</div>
		</div>
		<div class="layui-form-item">
			<label class="layui-form-label">用户名</label>
			<div class="layui-input-inline">
				<input type="text" id="username" name="username" autocomplete="off" placeholder="请输入用户名" class="layui-input">
			</div>
		</div>
		<div class="layui-form-item">
			<label class="layui-form-label">密码</label>
			<div class="layui-input-inline">
				<input type="password" id="password" name="password" autocomplete="off" placeholder="请输入密码" class="layui-input">
			</div>
		</div>
		<div class="layui-form-item">
			<label class="layui-form-label">验证密码</label>
			<div class="layui-input-inline">
				<input type="password" id="confirm_password" name="confirm_password" autocomplete="off" placeholder="请输入验证密码" class="layui-input">
			</div>
		</div>
		<div class="layui-form-item">
			<label class="layui-form-label">Email</label>
			<div class="layui-input-inline">
				<input type="text" id="email" name="email" autocomplete="off" placeholder="请输入Email" class="layui-input">
			</div>
		</div>
		<div class="layui-form-item">
			<label class="layui-form-label"></label>
			<div class="layui-input-inline">
				<button class="layui-btn" lay-submit="" lay-filter="demo1">立即提交</button>
				<button type="reset" class="layui-btn layui-btn-primary">重置</button>
			</div>
		</div>

    </form>
	
	<script src="https://cdn.bootcss.com/jquery/3.3.1/jquery.min.js"></script>
	<script src="https://cdn.bootcss.com/jquery-validate/1.19.0/jquery.validate.min.js"></script>
	<script src="https://cdn.bootcss.com/jquery-validate/1.19.0/localization/messages_zh.min.js"></script>
	<script src="http://layui.hcwl520.com.cn/layui-v2.4.5/layui.js?v=201811010202"></script>
	<style>
	.layui-form-label {
		width: 150px;
	}
	.layui-input {
		width: 200px;
	}
	.layui-form-checkbox span {
		height: 38px;
		vertical-align: middle;
	}
	.error {
		color: blue;
	}
    </style>
	<script>
	    layui.use(['layer', 'form'], function () {
			var layer = layui.layer,
				form = layui.form;
		});
	
        $.validator.setDefaults({
            submitHandler: function () {
                layer.alert("提交事件!");
            }
        });
		
        $(function(){
            // 在键盘按下并释放及提交后验证提交表单
            $("#signupForm").validate({
				onfocusin: function (element) {
					$(element).valid();
				},
				onfocusout: function (element) {
					$(element).valid();
				},
				onclick: function (element) {
					$(element).valid();
				},
				/*
				onkeyup: function (element) {
					$(element).valid();
				},
				*/
                rules: {
                    firstname: "required",
                    lastname: "required",
                    username: {
                        required: true,
                        minlength: 2
                    },
                    password: {
                        required: true,
                        minlength: 5
                    },
                    confirm_password: {
                        required: true,
                        minlength: 5,
                        equalTo: "#password"
                    },
                    email: {
                        required: true,
                        email: true
                    }
                },
                messages: {
                    firstname: "请输入您的名字",
                    lastname: "请输入您的姓氏",
                    username: {
                        required: "请输入用户名",
                        minlength: "用户名不能少于两个字母"
                    },
                    password: {
                        required: "请输入密码",
                        minlength: "密码长度不能小于 5 个字母"
                    },
                    confirm_password: {
                        required: "请输入密码",
                        minlength: "密码长度不能小于 5 个字母",
                        equalTo: "两次密码输入不一致"
                    },
                    email: "请输入一个正确的邮箱"
                },
                //重写showErrors
                showErrors: function (errorMap, errorList) {
					$.each(errorList, function (i, v) {
						//layer显示
						layer.tips(v.message, v.element, {tips: [2, '#ee1212'], time: 1500});
						return false;
					});
                },
                /* 失去焦点时不验证 */
                onfocusout: false
            });
        });
    </script>

	</body>
</html>
```





