title: 用github和hexo搭建blog
categories: hexo
tags: hexo
---

## 前言
win7上搭建自己的blog用于记录学习到的东西。

### 安装Node.JS
从[Node.JS](http://nodejs.org/)官网下载安装。

### 安装hexo
    npm install hexo -g

### 配置hexo
    hexo init <directory>
    cd <directory>
    npm install

### 安装hexo插件
    npm install hexo-renderer-ejs --save
    npm install hexo-renderer-stylus --save
    npm install hexo-renderer-marked --save

    npm install hexo-generator-index --save
    npm install hexo-generator-archive --save
    npm install hexo-generator-category --save
    npm install hexo-generator-tag --save
    npm install hexo-server --save
    npm install hexo-deployer-git --save
    npm install hexo-deployer-heroku --save
    npm install hexo-deployer-rsync --save
    npm install hexo-deployer-openshift --save
    npm install hexo-renderer-marked@0.2 --save
    npm install hexo-renderer-stylus@0.2 --save
    npm install hexo-generator-feed@1 --save
    npm install hexo-generator-sitemap@1 --save


### 查看效果
运行下面命令，然后打开浏览器，输入http://< yourip >:4000查看是否安装成功

    hexo server

### 发布
运行下面命令然后将public目录下的内容拷贝到github项目目录下并且上传

    hexo g

### 参考
[http://wsgzao.github.io/post/hexo-guide/](http://wsgzao.github.io/post/hexo-guide/)
