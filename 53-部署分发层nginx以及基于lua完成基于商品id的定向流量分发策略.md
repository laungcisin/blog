---
title: 53-部署分发层nginx以及基于lua完成基于商品id的定向流量分发策略
date: 2018-12-27 13:24:41
tags:
categories: redis
---


用bigdata01和bigdata02作为应用层nginx服务器，用bigdata03作为分发层nginx。

在bigdata03，也就是分发层nginx中，编写lua脚本，完成基于商品id的流量分发策略。

流量分发策略：
>1. 获取请求参数，比如productI；
>1. 对productId进行hash；
>1. hash值对应用服务器数量取模，获取到一个应用服务器；
>1. 利用http发送请求到应用层nginx；
>1. 获取响应后返回；

这个就是基于商品id的定向流量分发的策略，

lua脚本来编写和实现

我们作为一个流量分发的nginx，会发送http请求到后端的应用nginx上面去，所以要先引入lua http lib包

```
cd /home/bigdata/ngnix-test/lualib/resty/  
wget https://raw.githubusercontent.com/pintsized/lua-resty-http/master/lib/resty/http_headers.lua
wget https://raw.githubusercontent.com/pintsized/lua-resty-http/master/lib/resty/http.lua
```


代码：
```
vi /home/bigdata/ngnix-test/lua/ngnix-test.lua

local uri_args = ngx.req.get_uri_args()
local productId = uri_args["productId"]

local host = {"192.168.33.61", "192.168.33.62"}
local hash = ngx.crc32_long(productId)
hash = (hash % 2) + 1  
backend = "http://"..host[hash]

local requestPath = uri_args["requestPath"]
requestPath = "/"..requestPath.."?productId="..productId


local http = require("resty.http")  
local httpc = http.new()  

local resp, err = httpc:request_uri(backend, {  
    method = "GET",  
	path = requestPath
})

if not resp then  
    ngx.say("request error :", err)  
    return  
end

ngx.say(resp.body)  
  
httpc:close() 
```
```
/export/servers/nginx/sbin/nginx -s reload
```

访问 [http://192.168.33.63/ngnix-test?requestPath=ngnix-test&productId=1](http://192.168.33.63/ngnix-test?requestPath=ngnix-test&productId=1)


基于商品id的定向流量分发策略的lua脚本就开发完了，而且也测试过了

我们就可以看到，如果你请求的是固定的某一个商品，那么就一定会将流量打到固定的一个应用nginx上面去

