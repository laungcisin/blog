---
title: hexo命令
date: 2018-01-01 00:00:01
tags:
---

```
#安装环境
hexo init laungcisin.github.io
cd laungcisin.github.io
npm install

npm audit fix

npm install hexo-cli -g
npm install hexo --save
npm install --save hexo-deployer-git
npm install hexo-server –save
git clone https://github.com/iissnan/hexo-theme-next themes/next


#本地运行
hexo clean && hexo g && hexo s

#浏览器查看
http://192.168.33.61:4000/
```
---

```
#部署到github上
hexo clean && hexo g && hexo d

https://laungcisin.github.io/

yum install curl-devel expat-devel gettext-devel openssl-devel zlib-devel

git config --global user.email "laungcisin@qq.com"
git config --global user.name "laungcisin"
```