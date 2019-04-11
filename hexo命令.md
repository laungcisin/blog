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

su
npm install hexo-server
exit

git clone https://github.com/iissnan/hexo-theme-next themes/next

```

## 将Hexo博客 部署在Github Pages上
1. 创建Github账号
1. 创建Github仓库，Github仓库名为：`<Github账号名称>.github.io`
1. 将本地Hexo博客推送到GithubPages
    1. 安装`hexo-deployer-git`插件  
        `npm install hexo-deployer-git --save`
    1.  添加SSH key  
        - 创建一个 SSH key 。在命令行（即Git Bash）输入以下命令， 回车三下即可   
            `ssh-keygen -t rsa -C "邮箱地址"`
        - 添加到 github。 复制密钥文件内容（路径形如C:\Users\Administrator\.ssh\id_rsa.pub），粘贴到New SSH Key即可。
        - 测试是否添加成功。在命令行（即Git Bash）依次输入以下命令，返回“You’ve successfully authenticated”即成功：  
            ```
            ssh -T git@github.com
            yes
            ```
    1. 修改_config.yml  
        ```
        # 修改主题
        #theme: landscape
        theme: next
        
        # 添加github账号
        # Deployment
        ## Docs: https://hexo.io/docs/deployment.html
        deploy:
        type: git
        repository: git@github.com:<Github账号名称>/<Github账号名称 >.github.io.git
        branch: master
         ```
    2. 推送到GithubPages  
        `hexo clean && hexo g && hexo d`
           
```
#本地运行
hexo clean && hexo g && hexo s

#浏览器查看
http://192.168.33.61:4000/
```
---

```
#确保之前的步骤都能执行成功

#部署到github上
hexo clean && hexo g && hexo d

https://laungcisin.github.io/

yum install curl-devel expat-devel gettext-devel openssl-devel zlib-devel

git config --global user.email "laungcisin@qq.com"
git config --global user.name "laungcisin"
```