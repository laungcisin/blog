---
title: 54-基于nginx-lua-java完成多级缓存架构的核心业务逻辑
date: 2018-12-27 19:38:12
tags:
categories: redis
---

分发层nginx，lua应用，会将商品id，商品店铺id，都转发到后端的应用nginx。

分发层ngnix配置
```
vi /home/bigdata/ngnix-test/ngnix-test.conf

	location /product {
		default_type 'text/html';
		content_by_lua_file /home/bigdata/ngnix-test/lua/distribute.lua;
	}
    
vi /home/bigdata/ngnix-test/lua/distribute.lua

local uri_args = ngx.req.get_uri_args()
local productId = uri_args["productId"]
local shopId = uri_args["shopId"]

local host = {"192.168.33.61", "192.168.33.62"}
local hash = ngx.crc32_long(productId)
hash = (hash % 2) + 1  
backend = "http://"..host[hash]

local requestPath = uri_args["requestPath"]
requestPath = "/"..requestPath.."?productId="..productId.."&shopId="..shopId


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

`/export/servers/nginx/sbin/nginx -s reload`

## 业务逻辑：
>1. 应用nginx的lua脚本接收到请求；
>1. 获取请求参数中的商品id，以及商品店铺id；
>1. 根据商品id和商品店铺id，在nginx本地缓存中尝试获取数据；
>1. 如果在nginx本地缓存中没有获取到数据，那么就到redis分布式缓存中获取数据，
	如果获取到了数据，还要设置到nginx本地缓存中；
>1. 如果缓存数据生产服务没有在redis分布式缓存中没有获取到数据，那么就在自己本地ehcache中获取数据，
	返回数据给nginx，也要设置到nginx本地缓存中；
>1. 如果ehcache本地缓存都没有数据，那么就需要去原始的服务中拉去数据，该服务会从mysql中查询，
	拉去到数据之后，返回给nginx，并重新设置到ehcache和redis中；
>1. nginx最终利用获取到的数据，动态渲染网页模板。
>1. 将渲染后的网页模板作为http响应，返回给分发层nginx
	
## 注意事项：
>建议不要用nginx+lua直接去获取redis数据；
因为openresty没有太好的redis cluster的支持包，所以建议是发送http请求到缓存数据生产服务，由该服务提供一个http接口；
缓存数生产服务可以基于redis cluster api从redis中直接获取数据，并返回给nginx。
要使用发送http请求，需下载http相关lua脚本：
```
cd /home/bigdata/ngnix-test/lualib/resty
wget https://raw.githubusercontent.com/pintsized/lua-resty-http/master/lib/resty/http_headers.lua
wget https://raw.githubusercontent.com/pintsized/lua-resty-http/master/lib/resty/http.lua
```


## 下载相关lua脚本:
```
cd /home/bigdata/ngnix-test/lualib/resty
wget https://raw.githubusercontent.com/bungle/lua-resty-template/master/lib/resty/template.lua

mkdir -p /home/bigdata/ngnix-test/lualib/resty/html
cd /home/bigdata/ngnix-test/lualib/resty/html

wget https://raw.githubusercontent.com/bungle/lua-resty-template/master/lib/resty/template/html.lua
```

## 配置模板
在 `/home/bigdata/ngnix-test/ngnix-test.conf` 的 `server` 中配置模板位置
```
vi /home/bigdata/ngnix-test/ngnix-test.conf

...
set $template_location "/templates";  
set $template_root "/home/bigdata/ngnix-test/templates";
...
```

```
mkdir -p /home/bigdata/ngnix-test/templates
vi /home/bigdata/ngnix-test/templates/product.html

<html>
	<head>
		<meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
		<title>商品详情页</title>
	</head>
<body>
商品id: {* productId *}<br/>
商品名称: {* productName *}<br/>
商品图片列表: {* productPictureList *}<br/>
商品规格: {* productSpecification *}<br/>
商品售后服务: {* productService *}<br/>
商品颜色: {* productColor *}<br/>
商品大小: {* productSize *}<br/>
店铺id: {* shopId *}<br/>
店铺名称: {* shopName *}<br/>
店铺评级: {* shopLevel *}<br/>
店铺好评率: {* shopGoodCommentRate *}<br/>
</body>
</html>
```

## 将渲染后的网页模板作为http响应，返回给分发层nginx

`/export/servers/nginx/conf/nginx.conf` 中添加设置：
```
vi /export/servers/nginx/conf/nginx.conf

lua_shared_dict my_cache 128m;
```

`/home/bigdata/ngnix-test/ngnix-test.conf` 中添加设置：
```
vi /home/bigdata/ngnix-test/ngnix-test.conf
...
	location /product {
		default_type 'text/html';
		content_by_lua_file /home/bigdata/ngnix-test/lua/product.lua;
	}
...
```

`/home/bigdata/ngnix-test/lua/product.lua` 脚本中：
```
vi /home/bigdata/ngnix-test/lua/product.lua

local uri_args = ngx.req.get_uri_args()
local productId = uri_args["productId"]
local shopId = uri_args["shopId"]

local cache_ngx = ngx.shared.my_cache

local productCacheKey = "product_info_"..productId
local shopCacheKey = "shop_info_"..shopId

local productCache = cache_ngx:get(productCacheKey)
local shopCache = cache_ngx:get(shopCacheKey)

if productCache == "" or productCache == nil then
	local http = require("resty.http")
	local httpc = http.new()

	local resp, err = httpc:request_uri("http://192.168.230.10:8080",{
  		method = "GET",
  		path = "/getProductInfo?productId="..productId
	})

	productCache = resp.body
	cache_ngx:set(productCacheKey, productCache, 10 * 60)
end

if shopCache == "" or shopCache == nil then
	local http = require("resty.http")
	local httpc = http.new()

	local resp, err = httpc:request_uri("http://192.168.230.10:8080",{
  		method = "GET",
  		path = "/getShopInfo?shopId="..shopId
	})

	shopCache = resp.body
	cache_ngx:set(shopCacheKey, shopCache, 10 * 60)
end

local cjson = require("cjson")
local productCacheJSON = cjson.decode(productCache)
local shopCacheJSON = cjson.decode(shopCache)

local context = {
	productId = productCacheJSON.id,
	productName = productCacheJSON.name,
	productPrice = productCacheJSON.price,
	productPictureList = productCacheJSON.pictureList,
	productSpecification = productCacheJSON.specification,
	productService = productCacheJSON.service,
	productColor = productCacheJSON.color,
	productSize = productCacheJSON.size,
	shopId = shopCacheJSON.id,
	shopName = shopCacheJSON.name,
	shopLevel = shopCacheJSON.level,
	shopGoodCommentRate = shopCacheJSON.goodCommentRate
}

local template = require("resty.template")
template.render("product.html", context)
```

-----------------------------------------------------------------------
在两台ngnix应用服务器，根据上面配置，重新部署。
```
scp /export/servers/nginx/conf/nginx.conf bigdata@bigdata02:/export/servers/nginx/conf/

scp -r /home/bigdata/ngnix-test/ bigdata@bigdata02:/home/bigdata/

/export/servers/nginx/sbin/nginx
/export/servers/nginx/sbin/nginx -s reload
```


第一次访问的时候，其实在nginx本地缓存中是取不到的，所以会发送http请求到后端的缓存服务里去获取，会从redis中获取

拿到数据以后，会放到nginx本地缓存里面去，过期时间是10分钟

然后将所有数据渲染到模板中，返回模板

以后再来访问的时候，就会直接从nginx本地缓存区获取数据了

缓存数据生产 -> 有数据变更 -> 主动更新两级缓存（ehcache+redis）-> 缓存维度化拆分

分发层nginx + 应用层nginx -> 自定义流量分发策略提高缓存命中率

nginx shared dict缓存 -> 缓存服务 -> redis -> ehcache -> 渲染html模板 -> 返回页面

还差最后一个很关键的要点，就是如果你的数据在nginx -> redis -> ehcache三级缓存都不在了，可能就是被LRU清理掉了

这个时候缓存服务会重新拉去数据，去更新到ehcache和redis中，这里我们还没讲解

分布式的缓存重建的并发问题