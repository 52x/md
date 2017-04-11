title: 用Hexo与Github打造属于自己的博客
date: 2015-08-02 20:22:44
permalink: hexo001
tags:
- Hexo
categories:
- Hexo

---
根据转载：{% link 粉丝日志 http://blog.fens.me/hexo-blog-github/ true 粉丝日志 %}修改
## Hexo介绍

&emsp;&emsp;Hexo是一个简单的、轻量的、基于Node的一个静态博客框架。通过Hexo我们可以快速创建自己的博客，仅需要几条命令就可以完成。发布时，Hexo可以部署在自己的Node服务器上面，也可以部署github上面。对于个人用户来说，部署在github上好处颇多，不仅可以省去服务器的成本，还可以减少各种系统运维的麻烦事(系统管理、备份、网络)。所以，基于github的个人站点，正在开始流行起来….
&emsp;&emsp;Hexo的官方网站：http://hexo.io/
## Hexo安装
### 安装Git
&emsp;&emsp;自行百度
### 安装node.js
&emsp;&emsp;node.js官网https://nodejs.org/
### 安装Hexo
&emsp;&emsp;命令行输入*npm install -g hexo*，等待安装完成。

&emsp;&emsp;可能会出现下载不了的问题，可以设置npm国内镜像。镜像使用方法（三种办法任意一种都能解决问题，建议使用第三种，将配置写死，下次用的时候配置还在）:
1. 通过config命令
npm config set registry https://registry.npm.taobao.org 
npm info underscore （如果上面配置正确这个命令会有字符串response）
2. 命令行指定
npm --registry https://registry.npm.taobao.org info underscore 
3. 编辑 ~/.npmrc 加入下面内容
registry = https://registry.npm.taobao.org
搜索镜像: https://npm.taobao.org

&emsp;&emsp;建立或使用镜像,参考: https://github.com/cnpm/cnpmjs.org

&emsp;&emsp;安装成功后在命令行输入hexo可以查看hexo的命令。

## Hexo使用
&emsp;&emsp;hexo使用超级简单，仅仅几个命令还就可以运行博客的各种操作。下面列出了hexo使用的一个简单流程与命令。
1. 创建项目主目录
&emsp;&emsp;可以在电脑上任意创建作为hexo博客项目的文件夹，以后所有的hexo操作都要cd到项目文件夹根目录下运行。
3. 运行命令hexo init
&emsp;&emsp;我们看到当前在目录下，出现项目的各个文件夹与文。
4. 运行命令hexo g
&emsp;&emsp;在本地目录下，会生成一个public的目录，里面包括了所有静态化的文件。
&emsp;&emsp;要说明的一点是hexo的静态博客框架，那什么是静态博客呢？静态博客，是只包含html, javascript, css文件的网站，没有动态的脚本。虽然我们是用Node进行的开发，但博客的发布后就与Node无关了。在发布之前，我们要通过一条命令，把所有的文章都做静态化处理，就是生成对应的html, javascript, css，使得所有的文章都是由静态文件组成的。
5. 运行命令npm install
&emsp;&emsp;注意不要漏了运行*npm install*命令，否则hexo s命令将报错。
6. 运行命令hexo s
&emsp;&emsp;在命令行窗口会提示：Hexo is running iat http://localhost:4000 Press Ctrl+C to stop.。这时我们就可以打开连接访问我们的博客了。

&emsp;&emsp;是不是很简单呢！接下来，我们要对Hexo做更全面的了解，才能做出个性化的博客。

## 部署Hexo到github
### 配置Git
&emsp;&emsp;Git是分布式的代码管理工具，远程的代码管理是基于SSH的，所以要使用远程的Git则需要SSH的配置。github的SSH配置如下：
1. 设置Git的user name和email：
$ git config --global user.name "dongxiaoxia"
$ git config --global user.email "dongxiaoxia@example.com"
2. 生成SSH密钥过程：
查看是否已经有了ssh密钥：cd ~/.ssh
如果没有密钥则不会有此文件夹，有则备份删除
2. 生成密钥：
运行命令$ ssh-keygen -t rsa -C “dongxiaoxia@example.com”
文件夹默认，输入两次密码即可。
Your identification has been saved in /home/tekkub/.ssh/id_rsa.
Your public key has been saved in /home/tekkub/.ssh/id_rsa.pub.
The key fingerprint is:
………………
最后得到了两个文件：id_rsa和id_rsa.pub
4. 在github上添加ssh密钥：
这要添加的是“id_rsa.pub”里面的公钥。登录github，打开项目主页，在setting页面上找到SSH KEY设置，然后添加ssh，标题随意。
5. 接着打开git ，输入命令*$ ssh -T git@github.com*测试连接是否成功。

### 部署到github
&emsp;&emsp;首先在github上创建一个项目dongxiaoxia.github.io，项目地址为https://github.com/dongxiaoxia.dongxiaoxia.github.io 。这里dongxiaoxia为用户名，项目名称必须为username.github.io.
&emsp;&emsp;编辑全局配置文件：_config.yml，找到deploy的部分，设置github的项目地址。
+ deploy:
  type: git
  repository: https://github.com/dongxiaoxia/dongxiaoxia.github.io.git
  branch: master
  
&emsp;&emsp;然后通过命令hexo d部署
&emsp;&emsp;如果提示count not find github，则deploy的type改成git，然后运行命令npm install hexo-deployer-git --save 再 hexo d。

&emsp;&emsp;就这样这个静态的web网站就被部署到了github,点击”Settings”，找到GitHub Pages，提示“Your site is published at http://dongxiaoxia.github.io ”，打开网页http://dongxiaoxia.github.io ，就是我们刚刚发布的站点。
## 绑定域名
&emsp;&emsp;首先得有一个自己的域名，如果没有则可以跳过这步。
1. 在域名解析页面设置主机记录，类型为CNAME，到dongxiaoxia.github.io。
2. 在项目根目录上新建一个文件CNAME，文件内容为域名如www.dongxiaoxia.xyz。
3. 然后通过浏览器打开www.dongxiaoxia.xyz就可以直接访问博客站点了。

&emsp;&emsp;到这里为止，博客的搭建与Hexo的基本操作都已经掌握啦！剩下的就是怎么去搭建自己独一无二的博客了。





