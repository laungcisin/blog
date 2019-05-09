---
title: mybatis问题解决集合
date: 2066-01-01 00:00:01
tags: mybatis
categories: mybatis
---

## **mybatis-foreach使用**

**前言**：
>后台传给mapper的字符串：`"b, c"`,
>mybatis需生成`select * from table where ids in ('b', 'e')`语句，来执行查询。
    
**解决方法**：
>mybatis只需使用foreach语句，即可实现功能。

**mybatis-xml配置**：
>```xml
<select id="getSimilarity" parameterType="java.lang.String" resultType="java.util.HashMap">
    select * from table
	where ids in 
	<foreach item="item" index="index" collection="ids.split(’,’)"  open="(" separator="," close=")">
		#{item}
	</foreach>
</select>
```

**控制台打印**：

>```
Preparing: select * from table where ids in ( ? , ?  ) 
Parameters: b(String), e(String)
```
1. 
1. 
1. 

