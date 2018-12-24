---
title: 38-CentOS7安装部署MySQL数据库
date: 2018-12-03 23:26:10
categories: mysql
tags:
---

[Centos7 Yum方式安装Mysql7](https://www.cnblogs.com/golerun/archive/2018/04/04/8710235.html)

安装命令：
```
wget http://repo.mysql.com/mysql-community-release-el7-5.noarch.rpm
rpm -ivh mysql-community-release-el7-5.noarch.rpm

yum install -y mysql-server
service mysqld start
chkconfig mysqld on
yum install -y mysql-connector-java
```

