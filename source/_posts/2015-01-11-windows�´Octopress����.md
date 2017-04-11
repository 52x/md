---
title: "windows下搭建Octopress博客"
date: 2015-01-11 21:43:31 +0800
tags:
comments: true
categories: 其他技术
keywords: octopress,blog,github
description: 在Windows下搭建Octopress博客。
---

<!--more-->
原文链接：<http://tianweili.github.io/blog/2015/01/11/create-octopress-blog-in-windows/>

## 将博客发布到GitHub
进入博客源代码所在目录。编辑markdown后

1. 执行`rake new_post['my first blog']`来生成一篇博文；
2. 执行`rake generate`生成博客网页；
2. 执行`rake preview`后在本地输入<localhost:4000>来预览博客；
3. 执行`rake setup_github_pages`命令后，按照提示输入对应的GitHub的repository地址：`git@github.com:TianweiLi/tianweili.github.com.git`；（不执行这一步会可能会报`No such file or directory - _deploy`错误）
3. 执行`rake deploy`将博客站点发布到GitHub`master`分支上，这样就可以访问博客了（这一步就是把public目录下文件push到master分支上）；
4. 将修改后的octopress源码push到GitHub的`source`分支上：

依次执行下面命令
```sh
	git add .
	git commit -m 'build octopress blog'
	git push origin source
```

## 换一台电脑写博客
如果需要在另一台电脑写博客并提交上去，那么可以采用下面步骤来实现。

先要找到GitHub的repository url，然后clone source分支到本地：

```sh
	git clone -b source git@github.com:TianweiLi/tianweili.github.com.git octopress
```
然后clone master分支到本地：
```sh
	cd octopress
	git clone git@github.com:TianweiLi/tianweili.github.com.git _deploy
```
然后进行一些相关依赖的安装，依次执行下面命令：

```sh
	gem install bundler
	bundle install
	rake install
	rake setup_github_pages
```

---

作者：[李天炜](http://tianweili.github.io/)

原文链接：<http://tianweili.github.io/blog/2015/01/11/create-octopress-blog-in-windows/>

转载请注明作者和文章出处，谢谢。