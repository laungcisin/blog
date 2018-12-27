---
title: 52-基于OpenResty部署应用层nginx以及nginx-lua开发helloworld
date: 2018-12-26 19:38:25
tags:
categories: redis
---


1、部署第一个`nginx`，作为应用层`nginx`（192.168.332.61那个机器上）

## 部署`OpenResty`
```

#使用root用户安装
yum install -y readline-devel pcre-devel openssl-devel gcc

#使用bigdata用户安装
cd /home/bigdata/software
wget http://openresty.org/download/openresty-1.13.6.1.tar.gz
tar -zxvf /home/bigdata/software/openresty-1.13.6.1.tar.gz -C /export/servers/

cd /export/servers/openresty-1.13.6.1/bundle/LuaJIT-2.1-20171103/
make clean && make && make install
ln -sf luajit-2.1.0-alpha /usr/local/bin/luajit

cd /export/servers/openresty-1.13.6.1/bundle 
wget https://github.com/FRiCKLE/ngx_cache_purge/archive/2.3.tar.gz
tar -xvf 2.3.tar.gz

cd /export/servers/openresty-1.13.6.1/bundle
wget https://github.com/yaoweibin/nginx_upstream_check_module/archive/v0.3.0.tar.gz
tar -xvf v0.3.0.tar.gz

cd /export/servers/openresty-1.13.6.1
./configure --prefix=/export/servers --with-http_realip_module  --with-pcre  --with-luajit --add-module=./bundle/ngx_cache_purge-2.3/ --add-module=./bundle/nginx_upstream_check_module-0.3.0/ -j2

make && make install

cd /export/servers/
ll
#显示有以下结果
/export/servers/luajit
/export/servers/lualib
/export/servers/nginx
/export/servers/nginx/sbin/nginx -V

#使用root启动nginx，否则会报错
#原因：the socket API bind() to a port less than 1024, such as 80 as your title mentioned, need root access.
/export/servers/nginx/sbin/nginx

```

## `nginx+lua`开发的`hello world`
```
vi /export/servers/nginx/conf/nginx.conf
```
在`http`部分添加：

```
...
lua_package_path "/export/servers/lualib/?.lua;;";
lua_package_cpath "/export/servers/lualib/?.so;;";
...
```
`/export/servers/nginx/conf`下，创建一个`lua.conf`
```
cd /export/servers/nginx/conf
vi lua.conf

server {
	listen 80;
	server_name _;
}
```
在`nginx.conf`的`http`部分添加：
```
vi /export/servers/nginx/conf/nginx.conf
include lua.conf;
```
验证配置是否正确：
```
#使用root用户执行
/export/servers/nginx/sbin/nginx -t
```

在`lua.conf`的`server`部分添加：
```
vi /export/servers/nginx/conf/lua.conf
	location /lua {
	default_type 'text/html';
	content_by_lua 'ngx.say("hello world")';
}

#使用root用户执行
/export/servers/nginx/sbin/nginx -t
```

重新`nginx`加载配置
```
/export/servers/nginx/sbin/nginx -s reload
```

访问 [http://192.168.33.61/lua](http://192.168.33.61/lua)
```
cd /export/servers/nginx/conf/lua/
vi test.lua

ngx.say("hello world"); 
```
修改`lua.conf`
```
vi /export/servers/nginx/conf/lua.conf
	location /lua {
		default_type 'text/html';
		content_by_lua_file conf/lua/test.lua; 
	}
```
查看异常日志
```
tail -f /export/servers/nginx/logs/error.log
```

## 工程化的`nginx+lua`项目结构
```
mkdir -p /home/bigdata/ngnix-test
```

```
#修改nginx.conf 把user 改成有权限的用户（当前用户）
#否则会报nginx_bug（1）：failed to load external Lua file "xxxx.lua": cannot open xxxx.lua: Permission denied 错误
vi /export/servers/nginx/conf/nginx.conf

#user nobody;
# 改成 --> user [用户名].根据实际情况修改
user bigdata


```

项目工程结构

```
ngnix-test
    ngnix-test.conf     
    lua              
		ngnix-test.lua
    lualib            
		*.lua
		*.so
```

放在`/home/bigdata/ngnix-test`目录下
```
vi /export/servers/nginx/conf/nginx.conf

http {
    include       mime.types;
    default_type  application/octet-stream;

	lua_package_path "/home/bigdata/ngnix-test/lualib/?.lua;;";
	lua_package_cpath "/home/bigdata/ngnix-test/lualib/?.so;;";
	include /home/bigdata/ngnix-test/ngnix-test.conf;
}

cd /home/bigdata/ngnix-test/
vi ngnix-test.conf

server {
    listen       80;  
    server_name  _;

	location /ngnix-test {
		default_type 'text/html';
		content_by_lua_file /home/bigdata/ngnix-test/lua/ngnix-test.lua;
	}
}

mkdir -p /home/bigdata/ngnix-test/lua
cd /home/bigdata/ngnix-test/lua
vi ngnix-test.lua

ngx.say("hello world");

mkdir -p /home/bigdata/ngnix-test/lualib
cp -r /export/servers/lualib/ /home/bigdata/ngnix-test/

#使用root用户执行
/export/servers/nginx/sbin/nginx -s reload
```

访问 [http://192.168.33.61/ngnix-test](http://192.168.33.61/ngnix-test)

## 如法炮制，在其余机器上，也用`OpenResty`部署一个`nginx`
```
scp -r /home/bigdata/ngnix-test/ bigdata@192.168.33.63:/home/bigdata/
scp -r /export/servers/nginx/conf/nginx.conf bigdata@192.168.33.63:/export/servers/nginx/conf/nginx.conf
```
