---
title: 写插件自动部署虚拟空间上的hexo博客
tags: [hexo,webhost,deploy]
date: 2017-02-23 11:24:48
updated: 2017-02-23 11:24:48
categories: [Hexo]
keywords: [hexo,deploy,webhost,hostinger]
description:
---

# 碰到的问题

我的网站是挂到了Hosinger的web hosting上的，不是VPS，只是一个虚拟主机托管空间上的。大概情况是这样的：

1. 能ssh

2. 能用git

   这么一看正好啊，用来部署hexo还是满好的
   实际用起来还是有点问题。每次我都是先要`hexo d` 部署到github上，然后再等了服务`git pull`

   麻烦，是我等懒人的最大的天敌。于是又开始了折腾之旅


# 解决办法

第一个想到的办法是服务器端直接用crontab实现。

过程中发现，ssh上bash中的crontab 不支持。（执行无反应）

这家支持的是cpanel的cron tab，是基于php的

测试了发现`exec, shell_exec, proc_open, passthru` 全都被禁掉了。。

怎么办，只能从hexo自身想办法，看看能不能自动让服务器签出呢？

于是有了我的这个结合：

- 同步源定位github
- hexo deploy是自动调用ssh
- ssh远程执行git pull命令

具体怎么做：

## 第一步 建立好git目录，设置好remote

在我的空间，我是将信息内容，直接放到了index目录下，然后配置.htaccess文件，实现转向

```
ErrorDocument 401 "Unauthorized"
ErrorDocument 403 "Forbidden"
RewriteEngine On

#### PERSISTENT CONTENT ####
DirectoryIndex index/index.html index.php index.cgi index.html
RewriteCond %{REQUEST_FILENAME} !-f
RewriteCond %{REQUEST_FILENAME} !-d
RewriteRule ^(.*)$ index/$1 [L,QSA]
```

然后再该目录中，签出最新版本，都配置好。

## 第二步 建立ssh调用的shell文件

然后在`cd ~`目录下面，新建`gitgit.sh`文件

```shell
cd public_html/index
git pull origin master
```

赋予该文件运行权限。

在本地，试试语句是不是成功吧`ssh uXXXXXXX@31.220.110.153 -p 22 "bash gitgit.sh"`

## 第三步 添加改hexo的调用吧

### 定义调用

我是这么做的，首先在_config.yml中修改配置，新增加了`- type: remotegit` ，如下：

```
deploy:
- type: git  
  repo:
      github: git@github.com:mosliu/mosliu.github.io.git
      coding: git@git.coding.net:mosliu/mosliu.git  
  branch: master
  message: "Hexo Deploy updated: {{ now('YYYY-MM-DD HH:mm:ss') }}"
- type: remotegit

```

这里定义了调用remotegit的deploy功能

### 编写调用

在hexo目录中新建scripts文件夹，新建一个js文件，文件名随意，我用的是remotegit.js

内容：

```
'use strict';
/**
 * Created by Moses on 2017/2/23.
 */
const exec = require('child_process').exec;
const cmdStr = 'ssh uXXXXXXX@31.220.110.153 -p 22 "bash gitgit.sh"';


hexo.extend.deployer.register('remotegit', function(args){
    console.log("call remotegit!!");
    exec(cmdStr, function(err,stdout,stderr){
        if(err) {
            console.log('ssh error:'+stderr);
        } else {
            console.log(stdout);
        }
    });
});

```

完成之后，执行hexo d的时候，就可以看到服务器也一起更新啦。

# 后记

- 我是配置了秘钥登录的，不知道用密码怎么样，应该一样的
- 我用的是windows系统。要先在cmd中执行过命令，保存了服务器的公钥才行。

