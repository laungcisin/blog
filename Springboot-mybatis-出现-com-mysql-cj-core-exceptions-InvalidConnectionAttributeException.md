---
title: >-
  Springboot-mybatis-出现
  com.mysql.cj.core.exceptions.InvalidConnectionAttributeException...
date: 2018-10-24 21:31:32
tags:
---
```
其实这个不是spring boot + mybatis的问题， 其实是为了使MySQL JDBC驱动程序的5.1.33版本与UTC时区配合使用，必须在连接字符串中明确指定serverTimezone。
```

```
jdbc:mysql://localhost:3306/lovewhf?useUnicode=true&useJDBCCompliantTimezoneShift=true&useLegacyDatetimeCode=false&serverTimezone=UTC
```
```
在这里边明确指定serverTimezone
```