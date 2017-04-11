---
title: 使用 Hexo+GitHub 搭建独立博客
date: 2016-03-19 10:35:42
tags: [Hexo, GitHub]
toc: true
---

## 什么是博客

个人感觉博客就是在线的公开日记本，用来记录一些东西，或好、或环，都值得一记。
并且个人感觉，不要把博客弄得太社交化（或许本人只是太喜欢安静的思考和写些东西），太社交化容易使作者分心，哈哈，这也只是本人的个人看法了。

对于“博客”，wiki 上的解释如下：

>博客 (Blog) 是 (Web log 网络日志）的简称，又译为网络日志、部落格或部落阁等，是一种通常由个人管理、不定期张贴新的文章的网站。博客上的文章通常根据张贴时间，以倒序方式由新到旧排列。许多博客专注在特定的课题上提供评论或新闻，其他的则被作为比较个人的日记。

<!--more-->

## 为什么搭建独立博客

对于这个问题，我只想借用 Linus 的一句话：“just for fun.”

## 选择什么方式搭建

既然喜欢折腾，那就折腾到底呗，所以，我就选用 Github + Hexo 去搭建独立的博客。

## 搭建过程

### 在 GitHub 创建一个特殊仓库

注意仓库命名得是：username.github.io【username 为你 GitHub 账户名称，如果没有 GitHub 账户，那就请先注册吧】

### 在本地电脑上下载安装 [Hexo](https://hexo.io/zh-cn/docs/index.html)，并配置好相关环境

首先，请先去 Hexo 官网查看下 Hexo 的介绍文档吧。

了解了什么是 Hexo 的话，那就按照官方的文档来进行安装吧。

首先在安装 Hexo 之前系统必须有 Node.js 和 Git，如果没有，就按照下面的步骤进行安装吧。

Node.js 安装【本教程假设你的系统为 Ubuntu LTS 14.04，若是其他系统，请自行上网查询安装方法吧】

安装 Node.js 的最佳方式是使用 nvm（nodejs 版本管理器），那就先安装 nvm 吧。

``` bash
$ curl https://raw.github.com/creationix/nvm/master/install.sh | sh
```
如果系统报告没有 curl，请自行安装

``` bash
sudo apt-get install curl
```
Git 安装【同上】
``` bash
$ sudo apt-get install git
```
上述软件安装完毕之后，现在可以进行安装 Hexo 了。

``` bash
$ npm install -g hexo-cli
```
如果系统报告没有 npm，请自行安装

``` bash
$ sudo apt-get install npm
```
## 发布文章

现在，所有环境已经配置完毕了，让我们开始创建第一个博客站点吧。

### 建站

``` bash
$ hexo init folder
$ cd folder
$ npm install
```

这样一个新的博客站点也就建好了，指定的文件夹目录如下：

``` bash
.
├── _config.yml
├── package.json
├── scaffolds
├── source
|   ├── _drafts
|   └── _posts
└── themes
```
（注解：folder 指的是，网站博客的根目录）

### 写博客

站点新建完毕后可以开始写博客了。

``` bash
$ hexo new title
```
（注解：title 指的是，文章标题）

然后切换到 source/_posts 目录下，找到刚才新建的文章，进行编辑、保存。

保存好文章之后，现在可以发布文章了。

``` bash
$ hexo g && hexo s
```
打开 http://localhost:4000 就可以看到网站博客了。

### 部署到 GitHub 上

首先得安装 [hexo-deployer-git](https://github.com/hexojs/hexo-deployer-git)

``` bash
$ npm install hexo-deployer-git --save
```
修改配置文件_config.yml

``` bash
deploy:
type:git
repo:repository url
```
（repository url 指的是你先前创建的仓库的公网访问地址，既 https://username.github.io）

## 总结

在此，我们就初步成功搭建完了我们的独立博客了，嘻嘻。
至于美化、优化等步骤，有兴趣的可以慢慢折腾，我也会继续折腾我的博客的。

另外，如果有什么不清楚的问题，可以请教 Google（哦，Google 不能访问啊，那就将就百度吧，哈哈），也可以私信本人，联系方式请见关于页面。
