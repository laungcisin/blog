---
title: SpringBoot-Mybatis关于开启驼峰映射的设置
date: 2018-12-18 11:09:03
categories:
tags:
---


1. 在application.properties文件中添加：
```
mybatis.configuration.map-underscore-to-camel-case=true
```

1. 在mybatis的配置文件，如mybatis-config.xml中进行配置
```
<configuration>
    <!-- 开启驼峰映射 ，为自定义的SQL语句服务-->
    <!--设置启用数据库字段下划线映射到java对象的驼峰式命名属性，默认为false-->  
    <settings>
      <setting name="mapUnderscoreToCamelCase" value="true"/>
    </settings> 
</configuration>
```