title: 两台设备中同步Hexo博客
date: 2015-10-04 19:41:21
categories: Hexo|博客技术
tags: [Hexo, blog, git, github, Ubuntu, sync]
---
现在时不时会在Ubuntu系统中编辑文档，将这个系统中的.md编辑完在发回WIN7系统deploy到hexo是一个很烦的过程，尤其是这事儿以后会经常发生的时候。
所以必须寻求一个解决方案，可以在Ubuntu中同样设定一个hexo的文件夹，并通过github同步数据源。

Hexo文件夹并不需要完全同步，涉及到的修改主要涉及Hexo文件夹下的_config.yml, source文件夹和themes文件夹。_config.yml和themes一旦设定，并不需要经常更新，可以在新设备中的Hexo文件夹设定好了以后用老设备中的文件覆盖上去。而source是更新最频繁的，需要将这个文件夹跟github同步。

<!-- more -->

## 同步WIN7中的Hexo/source文件夹至Github
因为在搭建hexo博客的时候已经设定过一次ssh key，此处不用另外创建。

登陆Github的个人账户，然后新建一个repository，名字叫`source`。创建完成后你会看到以下页面：

![source-repo-creation](/img/source-repo-creation.png)

我的Hexo文件夹放在E盘中，在git bash中输入：
``` bash
$ cd E:\Hexo\source
$ git init
$ git remote add origin git@github.com:yourusername/source.git
$ git add .
$ git commit -m "initialize hexo folder as a local repo"  # customize your own message
$ git pull origin master
$ git push -u origin master
```

倒数第二步的`git pull origin master`，第一次做的时候，如果github的source仓库中中没有任何文件，会返回错误值，可以忽略。

Github为WINDOWS和OS X系统都提供了[Github Desktop](desktop.github.com)，这是一个十分方便的github桌面管理工具，能自动检测github repo和local repo之间的版本差别并提示更新。上面那些步骤也可以通过desktop这样实现：
1. 备份E:\Hexo\source文件夹，删除原始文件夹
2. 点击Github Desktop左上角的"+"号并点选"Clone"，从刷新后的目录中选择source这个repo
3. 继续上一步，选择local path: E:\Hexo。这样你可以在E:\Hexo中看到一个新生成的source文件夹
4. 将备份中的所有.md文件转移到source文件夹中
5. 点击Github Desktop右上角齿轮图标下面的"Sync"。这样本地source文件夹中的内容就全部上传到Github了。

## Ubuntu 14.04中Hexo文件夹设定
### 安装node.js
node.js和git是搭建hexo博客的两个基础。
在Ubuntu中可以用[nvm](https://github.com/creationix/nvm)安装node.js。
我尝试了各种nvm官方提供的自动安装方法，然而都失败了(RP啊。。。)，于是只好"manually install it". 安装需要用到管理员权限。
``` bash
$ sudo -i
$ git clone https://github.com/creationix/nvm.git ~/.nvm
$ . ~/.nvm/nvm.sh
```
同样也是官方文档建议，将以上命令的最后一行添加到~/.bashrc, ~/.profile或~/.zshrc三个文件中的任意一个中。
安装node.js，可以先查看一下可以安装的版本。
``` bash
$ nvm ls-remote
```
最近的版本是v4.1.1
``` bash
$ nvm install 4.1.1
```

检查是否安装成功，可以看现有node的版本。
``` bash
$ node --version
```

******
*编辑于10/5/2015*

重启电脑以后输入`hexo generate`发现hexo命令无法调用，多方查询，说有可能是nodejs的问题
``` bash
$ nvm ls
```
发现node的版本是`N/A`,说明之前安装的4.1.1并不成功(或许本身这个版本就不能用nvm来安装。。。)
那么还是回到稳妥的node版本
``` bash
$ nvm install 0.10
```
再次查看
``` bash
$ node --version
```
返回
```
v0.10.40
```
这次才是真的成功了。。。

******
*编辑于11/6/2015*

圈子里大牛们讨论上面的问题时，说到了nodejs的发展历史。
搜索条目里去年处理这个问题的解决方案之一就是用nvm安装0.10.xx版本的nodejs
今年官网上就已经推出5.0的版本了
版本演进之快，令人乍舌，不过hexo命令之所以无法使用，极有可能就是因为nodejs在发展过程中将底层调用命令(不是IT，不知道这样表述是否精确)从node变成了nodejs，而hexo仍然使用node去执行命令，并没有考虑nodejs版本上的巨大变化。
上面的解决方案是目前比较靠谱的方法。
期望有一天这个问题可以解决，让hexo可以使用更高级的nodejs。

******

### 安装git
Ubuntu有可能已经安装过git了，如果没有，用下面命令可以安装：
```
$ sudo apt-get install git-core
```

### 安装hexo
在Terminal中输入：
``` bash
$ npm install -g hexo-cli
$ exit     ## exit administrator mode
```
创建一个Hexo文件夹(这个在之前的博客中也有提到过[搭建Hexo博客](papacochon.com/2015/09/03/hexo-1_create-hexo-blog/))
``` bash
$ ## suppose we are in /home/username/
$ mkdir Hexo
$ hexo init /home/username/Hexo
$ cd Hexo
$ npm install
```
Hexo文件夹中所有文件生成以后，将WIN7系统中的themes，config.yml和source文件替换进去。

### 设置git
同样要在Ubuntu中新建SSH key并复制到Github中，让Github认证Ubuntu系统的这个电脑。此过程也可以参见原来的文章[搭建Hexo博客](http://papacochon.com/2015/09/03/hexo-1_create-hexo-blog/#3-2_新建SSH_key)。
用git命令行复制source到本地。大同小异。
这样基本设置就完成了。
我的麻烦点儿，除了这几个基本文件和文件夹的同步以外，还有一些小文件需要从WIN7中拷过来，no zuo no die。

## 注意事项
每次更新要注意先`git pull origin master`。这样才能保证本地是最新的状态。
同样，每次deploy以后，也要记得把source push到Github上，这样才能保证远端Github的repo上是最新的状态。


## Reference
1. [Installing Hexo on Ubuntu](http://blog.kavoori.com/2014-10-31/installing-hexo-on-ubuntu.html)
2. [Ubuntu下用Hexo搭建个人博客及常见问题的解决方案](blog.csdn.net/u012366161/article/details/42059813)
3. [使用GitHub + Hexo搭建个人博客（五）- Ubuntu升级后hexo命令失效](http://blog.csdn.net/yuguiyang1990/article/details/39522413)