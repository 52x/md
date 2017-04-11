title: 搭建Hexo博客
date: 2015-09-03 16:29:49
categories: Hexo|博客技术
tags: [blog, Hexo, git]
---
这是我这个代码菜鸟第一次自己搭建一个博客，感谢[Hexo](https://hexo.io/)的设计者[Tommy Chen](http://zespia.tw/)给我一个自high的机会。=3=
趁着还没忘，先把搭建过程记录下来(其实Hexo网站上也有...)。
<!-- more -->
# 1. 配置
我是**64位WIN7系统**，搭建之前，电脑里几乎没有搭建网站需要的软件配置。配置不难，需要安装两个软件：

* [Node.js](https://nodejs.org/)
* [Git](http://git-scm.com/)

只需要根据自己设备的位数选择下载包就行，然后直接安装。官网里说用nvm的方法安装node.js的方法也可以忽略。*如果先安装好了node.js,千万就不要再安装nvm了，不然后面的命令无法运行*。

----------

# 2. 安装Hexo
## 2.1 安装Hexo
安装好以上软件后，打开Git Bash，输入：

	$ npm install -g hexo-cli

## 2.2 设置Hexo文件夹
上条命令用于安装Hexo。安装完毕以后，需要设置一个文件夹(我个人想设置在E盘，可以根据个人需要修改文件路径)，用于存放Hexo初始化以后的文件：

	$ hexo init E:\Hexo
	$ cd E:\Hexo
	$ npm install

三条命令运行完毕后，可以在设定好的文件夹下看到以下文件和文件夹(可以用`ls`命令看到)：

	_config.yml  node_modules/  package.json  scaffolds/  source/  themes/

在很多其它个人攻略里，也提到另外一种方法，即手动创建E:\Hexo，然后在这个文件夹里点击右键菜单里的`Git Bash`，然后输入`npm install`。两种方法异曲同工。

## 2.3 生成博客并本地查看
仍然在E:\Hexo位置下的bash输入以下命令。

	$ hexo generate
	$ hexo server

在浏览器窗口输入`localhost:4000`以后，可以看到一个大气层图片为页首的个人博客页面。

----------

# 3. 页面部署
## 3.1 创建Github repository
首先，要有一个[Github](https://github.com)账户。创建好了以后添加一个新的repository(首页有个按钮"+New repository")。repository的名字必须以`xxx.github.io`的形式添加，`xxx`为你的用户名。比如你的用户名是johndoe，那么当你进入你创建好的repository页面的时候，地址栏应该是`https://github.com/johndoe/johndoe.github.io`。

## 3.2 新建SSH key
可参见Github的[官方教程](https://help.github.com/articles/generating-ssh-keys/)自己设置SSH key，这里简单提一下。
在Git Bash里键入：

	$ ssh-keygen -t rsa -b 4096 -C "johndoe@gmail.com"

键入后会连续弹出一些问题，可以直接忽略，官方文档也建议不要做什么设定，按`Enter`键一路到底。完成后窗口会出现提示，告诉你产生的文件存在哪里。打开`C:\Users\Administrator\.ssh`，可以看到连个文件：`id_rsa`和`id_rsa.pub`。前者是你的个人文件，后者是ssh pulic key。打开(我用Sublime Text 2，如果没有也可以用WIN自带的notepad)，将文件中的内容全选并拷贝。

在登录状态下，点击Github右上角的下拉框，点选Settings，然后点击跳转页面左侧导航栏的SSH keys，然后点"Add SSH Key"，Title可依据自己喜好取名，Key的部分直接粘贴刚才拷贝的部分。

确认添加以后，在Git Bash中输入：

	$ ssh -T git@github.com

你可以看到：

	Hi johndoe! You've successfully authenticated, but GitHub does not
	# provide shell access.

`johndoe`处应该是你自己的github用户名，这样你的SSH key就已经设置成功了。

## 3.3 设置config文件
在E:\Hexo文件夹下找到_config.yml，打开，在文件底部可以看到：

	deploy:
		type:

看到很多攻略里改写配置的方法是这样：

	deploy:
	  type: github
	  repository: https://github.com/johndoe/johndoe.github.io.git
	  branch: master

我按照这种方法运行`hexo deploy`一直会报错，显示：

	bash: /dev/tty: No such device or address
	error: failed to execute prompt script (exit code 1)
	fatal: could not read Username for 'https://github.com': No error

在网上搜索出的解决方案是把`repository`那一行修改成ssh连接的形式(如果上面这种方法你成功了，可忽略下面这种方法)：

	deploy:
	  type: git
	  repo: git@github.com:johndoe/johndoe.github.io.git
	  branch: master

然后在Git Bash里运行：

	$ hexo deploy

如果运行成功，你会看到返回信息的最后一行消息是：

	INFO  Deploy done: git

这就说明你的个人网站已经建立啦~
在地址栏里输入repository的完整名字(如，johndoe.github.io)，你就可以看到自己的网页了。

------------------

**更新于 10/15/2015**
WIN7电脑换了硬盘，得重新安装hexo。运行到`hexo d -g`，遇到报错

	ERROR Deployer not found: git

解决方案是输入：

	npm install hexo-deployer-git --save

----------

*除了文中提到的一些官方网站外，还有一些参考信息：*

1. [Zippera's blog - hexo 分类](http://zipperary.com/categories/hexo/)
2. [金石开 Hexo搭建Github静态博客](http://www.cnblogs.com/zhcncn/p/4097881.html)
3. [不如 hexo你的博客](http://ibruce.info/2013/11/22/hexo-your-blog/)
4. [Hexo: 独立博客新玩法](http://www.aips.me/hexo-independent-blog-new-ways.html)
5. [在github上用hexo搭建博客](http://pleasureswx123.github.io/2014/08/29/%E5%9C%A8github%E4%B8%8A%E7%94%A8hexo%E6%90%AD%E5%BB%BA%E5%8D%9A%E5%AE%A2/)
6. [Using SSH over the HTTPS port](https://help.github.com/articles/using-ssh-over-the-https-port/)