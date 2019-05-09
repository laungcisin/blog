---
title: Spring读取properties文件内容
date: 2019-04-28 09:04:50
tags: Spring
categories: Spring
---

https://www.cnblogs.com/zxf330301/p/6184139.html

1. 通过context:property-placeholder加载配置文件内容
```xml
<context:property-placeholder location="classpath:jdbc.properties" ignore-unresolvable="true"/>
```
此种配置方式与以下配置等价：
```xml
    <bean id="propertyConfigurer" class="org.springframework.beans.factory.config.PropertyPlaceholderConfigurer">
        <property name="ignoreUnresolvablePlaceholders" value="true"/>
        <property name="locations">
            <list>
                <value>classpath:jdbc.properties</value>
            </list>
        </property>
    </bean>
```
2. 代码中读取properties文件的内容

```xml
    <!-- 使用注解注入properties中的值 -->
    <bean id="propertiesConfig" class="org.springframework.beans.factory.config.PropertiesFactoryBean">
        <property name="locations">
            <list>
                <value>classpath:jdbc.properties</value>
            </list>
        </property>
        <!-- 设置编码格式 -->
        <property name="fileEncoding" value="UTF-8"/>
    </bean>
```
3. ddd
4. d
5. dddd
6. 
