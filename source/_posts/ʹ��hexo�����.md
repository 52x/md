title: 使用hexo搭建博客
date: 2014-02-06 18:17:12
tags: [hexo, Node.js, Git, Markdown]
categories: [writing]
description: 折腾，折腾！策马奔腾！终于搭建好我的博客，使用的是GitHub+hexo，也撰写了第一篇博客，记录下这次折腾之旅。顺带着也简要介绍了Node.js，Git和Markdown，主要是将chorme中的书签整理一下。使用Markdown书写博客，果真看着清爽，用起来也简单，还在尝试中！既然已经上路了，就迈开步子吧。

---
开始我并没有打算搭建一个博客，只是想找个*平台*潦草地记录平时遇到的问题，再稍微规划下一些日常事情。最近半年终于干了点计算机工程方面的实事，之前都是水过的，说来实在惭愧。去年暑期实习的时候，我接触到了*Python*，*Git*，*Node.js*以及*MongoDB*，在此期间用[高效e人][]来记录点滴，但总感觉有点不符合程序猿的身份；后来寒假实习遇到了[树上][]，跟着他后面接触到了一些比较流行的技术，尤其是适合折腾的技术，这段时间学会使用[trello][]来管理时间，另外有幸接触到了神的编辑器*emacs*和标记语言*markdown*。最近，烦于乱七八糟的记录以及臃肿的chrome书签，加之缺少整理的动力，最终的最终我准备搭建一个博客，以便自己整理总结平时所学所闻，之前也开通过几个BSP<sup>[[1]](#[1])</sup>类的博客，装饰好后没写一个字，实在没有写的动力；多亏了emacs，最近喜欢折腾一些技术，这次下定决心搭一个独立博客，寻找时发现[Wordpress][]挺受人青睐，而且*emacs*也有很好的插件支持。就在折腾的初始阶段，看各种评论的时候发现了*Jekyll+github*，进而又邂逅了最终的选择[hexo][]，看到关于[hexo][]的描述时，犹如一股清泉注入心底，怎一个爽字了得！我就是冲着*Node.js*，*Git*和写博客神器*markdown*情定了[hexo][]，当然后来发现[hexo][]中的命令，配置等都封装好了好多技术细节。好了，闲话少说，进入正题，记录下这次博客折腾之旅，也是我的第一篇博客，与有缘人分享！PS：没有详细的教程，只有我参考的文章出处以及我纠结过的坎儿。

# hexo是啥
[hexo][]是一个基于[Node.js][node]的静态博客程序，我在寻觅博客的时候，发现好多博客都标榜了**静态博客程序**，我的理解就是本地生成了静态html文件，然后再部署到服务器上。由于没有折腾过其他博客，博客之间的差异我也无从比较，只能人云亦云了。关于[hexo][]的特点，出现频率最高的就是：

> 快速！简单！可扩展性！

# 万事具备
不懂这些也没关系，我只是想稍微整理下。

## Node.js
[Node.js][node]是一个让*JavaScript*运行在服务端的开发平台，[Node.js][node]的*JavaScript*引擎是V8，来自Google Chrome项目。特点：

> 事件驱动；异步编程；回调函数；适合I/O密集型需求

我也是去年暑假在一家初创企业实习时，偶然接触到了[Node.js][node]，后来在学校折腾小论文，也没怎么进一步研究。总的感觉就是：理解其思想相对容易，想要深入掌握[Node.js][node]的话，需有较为深厚的JS功力（往往是前端JS）；对我而言，之前没有写过JS，所以瓶颈就在JS这门语言上，这里我只能补充一句**JS博大精深**啊！
* [《Node.js开发指南》][book1]：入门级书，按照书上搭建一个简易的博客还是不错的！
* [《深入浅出Node.js》][book2]：上个月刚入手的一本书，书很实用，就是对我来说还有点深奥，JS是硬伤！里面很多高级的用法，比如我的代码中尽是嵌套很深的回调函数，书中有几种优美的解决方案。慢慢研习！

## Git & GitHub
> [Git][]是一个免费的开源分布式版本控制系统，可以使用它来快速，高效的处理一切大小项目。<br />
> [GitHub][]是这个星球上最流行的开源托管服务，是一个程序员的名片，这年头不逛GitHub，都不好意思说自己是程序猿。

开始遇到[Git][]的时候，建立的是本地仓库，我也只会用`git clone`，`git init`，`git add`，`git commit`和`git push`这些简单命令，后来慢慢接触到[GitHub][]以及可以免费建立私人仓库的[Bitbucket][]，也熟悉了更多应用场合和命令，本地和远程仓库的基本使用还是能hold住的。列出一些参考资料，其中我也只看了[Git 简易指南][ref6]和[Git图解][ref7]：
### Git入门
* 入门宝典：[Git 简易指南][ref6]，简洁明了。
* [Git][]的官方网站提供了[Git][]的所有[文档][ref1]。也可以参考[Git Reference][ref2]。
* 书籍：
 * 《Pro Git》提供在线的[中文版][ref3]和[英文版][ref4]。
 * [《Git权威指南》][book3]：内容略充实了点。
* Github与Code School联合提供了一个在线互动教程：[Try Git][ref5]。
* 进阶篇：
 * [Git图解][ref7]：简洁，有事没事可以瞅瞅。
 * [Git Magic][ref8]。
 * [Think like a Git][ref9]。

### GitHub入门
* [Github][]提供了官方的[帮助信息][ref10]和他们的[博客][ref11]。
* [《Git权威指南》][book3]作者蒋鑫编写的[GotGitHub][ref12]。
* 如果还对加入[Github][]犹豫不决，看下[如何高效利用Github][ref13]。

## Markdown
第一次知道Markdown的时候，脑海里突然闪现出N多`.md`后缀的文件，典型的如[Github][]中仓库的`README.md`，相见恨晚啊！。后来各种机会跟Markdown碰面，最终也成了我挑选博客的必备条件之一！特点：
> 轻量级标记语言；格式即内容；易读易写；兼容HTML

兼容HTML意义重大，比如Markdown中插入图片没法调整大小，但是使用HTML中`<img>`标签就相当easy了！各种学习文档自行[Google][]之，这里再留下几点：
* 查阅语法：[Markdown语法说明][ref14]
* 编辑器：
 * [MaDe][]：Chrome插件，还行。
 * [Cmd][]：在线Markdown编辑阅读器，很人性化，同步滚动。
 * *emacs*: 写博客使用，需要一定的决心来折腾此神器。

# 搭建
搭建详细过程参考：
* [官方文档][ref15]
* [Alimon's blog][blog1]
* [Zippera's blog][blog2]

## 关于版本
* 系统：Ubuntu 12.04.3 LTS 32bit
* [Node.js][node]：v0.10.25
* *npm*：1.3.24
* [hexo][]：2.4.5

## 安装
安装时使用[Node.js][node]的包管理器*npm*，一般安装[Node.js][node]的时候会自带安装。这里给个在各种平台安装[Node.js][node]的[链接][ref18]。

主要命令：
```
$ npm install -g hexo
$ hexo init <folder>
```
`npm install`加上参数`-g`表示全局安装，否则你会发现在当前目录文件下多了`node_modules`文件夹，工具都安装在其中。我在安装插件时就是不带参数`-g`安装：
```
$ npm install <plugin-name> # 安装hexo-generator-feed和 hexo-generator-sitemap
```
## 配置
全局配置文件存储在博客根目录下，名为`_config.yml`，参考上面的文档，尽情配置吧！

## 主题
执行下面的命令拷贝主题包到本地，然后修改`_config.yml`中的`theme`项为`theme-name`。目前已有主题见[官方主题][ref16]。
```
$ git clone <repository> themes/<theme-name>
```
我的博客采用的是[Pacman][]主题，并且借用了[Zippera's blog][blog2]的背景图片，然后将主题颜色配置为红蓝色<sup>[[2]](#[2])</sup>。

## 本地运行与调试
在本地如果需要预览或者调试你的博客，可以启动本地服务器：
```
$ hexo server
```
网页运行在`http://localhost:4000`，更多高级启动方式参见[官方文档][ref15]。

## 部署
我将博客部署到[Github][]上，配置文件`_config.yml`：

```
deploy:
  type: github
  repository: https://github.com/leebug38/leebug38.github.io.git
  branch: master
```
需要在[Github][]上创建仓库，并且仓库名一定要为`username.github.io`，`username`为你的[Github][]注册用户名，即二级域名。配置文件中代码仓库`URL`最好使用`https`形式的。其实上面的步骤就是布置[GitHub Pages][Pages]。我没有使用个性域名，若有其他需求，可以看上面的参考文档以及[GitHub Pages][Pages]的[帮助文档][ref17]。最后部署到[Github][]上：
```
$ hexo clean  # 清除缓存文件(db.json)和hexo generate生成的文件(public)
$ hexo generate # 生成静态文件，存储在public文件夹中
$ hexo deploy # 部署你的网页
```
等十分钟左右，在浏览器中输入`username.github.io`就见到成果了。注意`hexo deploy`的部署过程：将`public`中文件复制到`.deploy`中，然后就是利用[Git][]来`push`到远程[GitHub][]仓库了。

## 写博客
博客根目录下一行命令即可：
```
$ hexo new [layout] <title>
```
[hexo][]有三种`layout`: `post`，`page`，和`draft`。不带此参数，则采用默认`layout`，文件`_config.yml`配置`default_layout`项。详情可以参考[writing][]。

## 简单优化
参照上面三篇文档，我也进行了简单优化：添加了`多说评论`，`RSS`，`sitemap`，以后优化完成后一并补上折腾过程。

# 后话
* 博客的搭建就暂且到这里，优化的路还很长，比如前端按照自己喜好修改等等。
* 第一篇博客写的也有点冗长，大招时间憋长了，见谅！
* *Markdown*真乃神器，初次使用，总感觉写的不好，多看多写！
* 学习一样东西最好的方法是：先模仿大神，再根据自己的需要来折腾！

[Google]: http://www.google.com.hk/
[Wordpress]: http://cn.wordpress.org/
[高效e人]: http://cn.efficientsoftware.net/
[树上]: http://cn0popeye.com/
[trello]: http://trello.com
[hexo]: http://zespia.tw/hexo/
[node]: http://nodejs.org/
[Git]: http://git-scm.com/
[GitHub]: http://github.com/
[Bitbucket]: http://bitbucket.org/
[MaDe]: https://chrome.google.com/webstore/detail/made/oknndfeeopgpibecfjljjfanledpbkog
[Cmd]: http://www.zybuluo.com/mdeditor
[Pacman]: http://github.com/tommy351/hexo/wiki/Themes
[Pages]:http://pages.github.com/
[writing]: http://zespia.tw/hexo/docs/writing.html
[blog1]: http://yangjian.me/workspace/building-blog-with-hexo/
[blog2]: http://zipperary.com/categories/hexo/
[book1]: http://book.douban.com/subject/10789820/
[book2]: http://book.douban.com/subject/25768396/
[book3]: http://book.douban.com/subject/6526452/
[ref1]: http://git-scm.com/doc
[ref2]: http://gitref.org/
[ref3]: http://git-scm.com/book/zh
[ref4]: http://git-scm.com/book
[ref5]: http://try.github.io/levels/1/challenges/1
[ref6]: http://rogerdudler.github.io/git-guide/index.zh.html
[ref7]: http://marklodato.github.io/visual-git-guide/index-zh-cn.html
[ref8]: http://www-cs-students.stanford.edu/~blynn/gitmagic/intl/zh_cn/index.html
[ref9]: http://think-like-a-git.net
[ref10]: http://help.github.com/
[ref11]: http://github.com/blog
[ref12]: http://www.worldhello.net/gotgithub/01-explore-github/index.html#id8
[ref13]: http://www.yangzhiping.com/tech/github.html
[ref14]: http://wowubuntu.com/markdown/
[ref15]: http://zespia.tw/hexo/docs/index.html
[ref16]: http://github.com/tommy351/hexo/wiki/Themes
[ref17]: http://help.github.com/categories/20/articles
[ref18]: http://www.infoq.com/cn/articles/nodejs-npm-install-config

-------------------------------------------------------------------------------

<span id="[1]">[1]</span> BSP：博客服务提供商。

<span id="[2]">[2]</span> 红蓝色：西甲豪门巴塞罗那的队服颜色，代表了巴萨。本人正努力成为合格的巴萨球迷。