---
title: 安装hexo并部署到github-page上
date: 2018-01-01 00:00:01
tags: hexo
---

```
#安装必须的插件
yum install curl-devel expat-devel gettext-devel openssl-devel zlib-devel

#配置git账号
git config --global user.email "laungcisin@qq.com"
git config --global user.name "laungcisin"

#安装之前，保证已经装好nodejs
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

```
#如果要本地运行
hexo clean && hexo g && hexo s

#浏览器查看
http://192.168.33.61:4000/
```

## 将Hexo博客 部署在Github Pages上
1. 创建Github账号
1. 创建Github仓库，Github仓库名为：`<Github账号名称>.github.io`
1. 将本地Hexo博客推送到GithubPages
    1. 安装`hexo-deployer-git`插件  
        `npm install hexo-deployer-git --save`
    1.  添加SSH key  
        - 创建一个 SSH key 。在命令行（即Git Bash）输入以下命令， 回车三下即可   
            `cd ~/.ssh`  
            `ssh-keygen -t rsa -C "laungcisin@qq.com"`
        - 添加到 github。   
        复制密钥文件内容（路径形如~/.ssh/id_rsa.pub），粘贴到New SSH Key即可。  
        `cat ~/.ssh/id_rsa.pub`
        - 测试是否添加成功。在命令行（即Git Bash）依次输入以下命令，返回“You’ve successfully authenticated”即成功：  
            ```
            ssh -T git@github.com
            yes
            ```
        - 如果报`ssh_exchange_identification: read: Connection reset by peer`，尝试用以下命令解决
            ```
            #编辑文件
            vi /etc/hosts.allow
            
            #追加内容
            sshd: ALL
            
            #重启ssh
            service sshd restart
            ```
    1. 修改_config.yml  
        ```
        #hexo显示图片功能
        #将post_asset_folder值改成true
        post_asset_folder: true
        npm install hexo-asset-image --save
        npm audit fix
        
        #创建文章使用hexo n命令
        hexo n "文章名"
        
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
