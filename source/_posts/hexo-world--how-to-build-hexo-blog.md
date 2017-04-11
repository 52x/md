layout: post
title: Hexo world ！—— 博客搭建教程
date: 2015-1-27 21:11:00
tags: 
- 教程
- 知识管理
category: Hexo
description: 博客刚刚搭好，总结一下这两天的成果。本教程着重介绍我在搭建这个博客时所犯的一些错误。同时也尽量将所能涉及的东西全部写进去，但也难免有所疏漏，还请谅解。

---

##前言

博客刚刚搭好，总结一下这两天的成果。本教程着重介绍我在搭建这个博客时所犯的一些错误。同时也尽量将所能涉及的东西全部写进去，但也难免有所疏漏，还请谅解。
在 [GitHub](http://github.com/) 上托管博客的优点，免费300M空间，速度较国外无需备案的空间快不少。
至于使用 [Hexo](http://hexo.io/) 这个博客程序的原因就如官网介绍的那样 `A fast, simple & powerful blog framework,powered by Node.js.` 
Hexo **足够轻，足够快**。


##配置 GitHub Pages 

GitHub Pages 本用于介绍托管在GitHub的项目，不过，由于他的空间免费稳定，用它来托管博客越来越流行。GitHub Pages 可以被认为是一个可以用来展示某个开源项目或者是 GitHub 用户的个人主页，这个页面只能托管静态网页。不过不用担心无法给博客添加评论系统，樱井优许多大神为 Hexo 开发的主题都支持很方便的添加评论系统。

1. 注册 GitHub 账户：
这里是 GitHub 网站的 [注册页](http://github.com/)。      
2. 创建GitHub Pages：
注册并登录 GitHub 之后可以在右上角用户名右侧看到一个 + ，New repositoy 。在这里 repositoy 是仓库的意思。仓库的名字必须以是 `yourname.github.io` ，这里的 yourname 是你的帐户名。其他的部分可以按个人喜好随便填写。


##配置 Git

[Git](http://git-scm.com/download/) 是一款免费、开源的分布式版本控制系统，用于敏捷高效地处理任何或小或大的项目。在这里 Git 的主要作用是将本地用 Hexo 生成的静态网页上传到 GitHub 指定的的代码仓库里，除此以外 Git 还负责下载更新 Hexo 、 Hexo 的主题以及 Hexo 的各种插件。

###下载并安装 Git 
[下载地址](http://git-scm.com/download/)        
如果你不知道某些选项的意思， **安装时一直点下一步即可** 。
###配置 SSH Key 

 
#### 检查SSH Key 的设置
在任意文件夹右键，找到 Git Bash ,打开 Git Bash ，输入如下命令（如不特殊提示下文所有的明令均是在 Git Bash 中使用,//后的内容为注释不包含在命令里）：
```
cd ~/.ssh
```

如果提示：No such file or directory 说明你是第一次使用git。
#### 生成新的SSH Key
```
ssh-keygen -t rsa -C "邮件地址@youremail.com"
```
提示：
```
Generating public/private rsa key pair.
Enter file in which to save the key (/Users/your_user_directory/.ssh/id_rsa):
```
回车就可以。
 **注意** : 此处的邮箱地址，你可以输入自己的邮箱地址；此处的「-C」的是大写的「C」。
 然后系统会要你输入密码：
 ```
 Enter passphrase (empty for no passphrase):<输入加密串>
Enter same passphrase again:<再次输入加密串>
```
在回车中会提示你输入一个密码，这个密码会在你提交项目时使用，如果为空的话提交项目时则不用输入。这个设置是防止别人往你的项目里提交内容。

 **注意** ：输入密码的时候没有*字样的，你直接输入就可以了。

最后看到这样的界面，就成功设置ssh key了：
![SSH Key](http://7u2qla.com1.z0.glb.clouddn.com/hello-worldssh-key.png)

####添加SSH Key到GitHub

在本机设置SSH Key之后，需要添加到GitHub上，以完成SSH链接的设置。

* 打开本地C:\User \你的用户名\.shh\id_rsa.pub文件(windows操作系统)。此文件里面内容为刚才生成的密钥。如果看不到这个文件，你需要设置显示隐藏文件。准确的复制这个文件的内容（用记事本打开即可），才能保证设置的成功。
* 登陆github系统。点击右上角的 Account Settings—->SSH Public keys —-> add another public keys
* 把你本地生成的密钥复制到里面（key文本框中）， 点击 add key 就ok了

![SSH Key set](http://7u2qla.com1.z0.glb.clouddn.com/hello-worldSSH-Key-set.jpg)

####测试

 可以输入下面的命令，看看设置是否成功，git@github.com的部分不要修改：
 
 ```
 ssh -T git@github.com
 ```
 
 如果是下面的反馈：
 
 ```
The authenticity of host 'github.com (207.97.227.239)' can't be established.
RSA key fingerprint is 16:27:ac:a5:76:28:2d:36:63:1b:56:4d:eb:df:a6:48.
Are you sure you want to continue connecting (yes/no)?
```

不要紧张，输入yes就好，然后会看到：

```
Hi cnfeat! You've successfully authenticated, but GitHub does not provide shell access.
```
####设置用户信息

现在你已经可以通过SSH链接到GitHub了，还有一些个人信息需要完善的。

Git会根据用户的名字和邮箱来记录提交。GitHub也是用这些信息来做权限的处理，输入下面的代码进行个人信息的设置，把名称和邮箱替换成你自己的，名字必须是你的真名，而不是GitHub的昵称。

```
$ git config --global user.name "yourame"//用户名
$ git config --global user.email  "yourame@gmail.com"//填写自己的邮箱
```


##配置 node.js 

[Node.js](http://nodejs.org/)  是一个基于Chrome JavaScript 运行时建立的一个平台，用来方便地搭建快速的易于扩展的网络应用· Node.js 借助事件驱动，非阻塞I/O模型变得轻量和高效，非常适合 运行在分布式设备的数据密集型的实时应用。

下载好 node.js ，安装时一路点下一步即可。 [下载地址](http://nodejs.org/download/)            

##配置 Hexo

[Hexo](http://hexo.io/) a fast, simple & powerful blog framework,powered by Node.js.
###安装 npm

```
git clone --recursive git://github.com/isaacs/npm.git 
```
```
node cli.js install npm -gf  
```

###安装Hexo
打开git。

```
npm install -g hexo
```

###部署Hexo
在你的电脑里找一个中意的文件夹（此后所有数据保存在本文件夹里,此后的所有操作均为从本文件夹右键打开 Git Bash，后文所提到的主目录就是这个文件夹），然后在此文件夹中右键打开Git Bash。

```
hexo init
```

Hexo随后会自动在目标文件夹建立网站所需要的所有文件。

```
hexo g
hexo s
```

现在我们已经搭建起本地的hexo博客了，到浏览器输入[localhost:4000](http://localhost:4000/)查看。
###获得满意的 Hexo 主题
到目前为止 Hexo 已经有不少的主题了，所有主题查看请[点这里](https://github.com/hexojs/hexo/wiki/Themes)。        

你可以从这里挑选一个你满意的主题，并记下这个主题的 GitHub 的地址。
下面的操作均是在选择[wuchong](http://wuchong.me/)的[jacman](https://github.com/wuchong/jacman)主题之后进行的操作。
```
git clone https://github.com/wuchong/jacman.git themes/jacman 
<注意：此处的jacman是主体的名字，可以任意，这个文件夹称为主题主目录>
```
 **注意：** 此处的网址就是你选择的主题的 GitHub 地址。
 更新主题可以使用如下命令：
 ```
 cd themes/jacman
 git pull
```

###配置主目录下_config.yml

找到主目录下的_config.yml ，用文本编辑器打开，推荐[Notepad2](http://xhmikosr.github.io/notepad2-mod/)或者下载[汉化版](http://7u2qla.com1.z0.glb.clouddn.com/hello-worldNotepad2.zip)      本汉化版来自某下载站。

* 找到 theme 属性，将它设置为你的主题主目录
```
theme: jacman
```

 **注意：** 所有的文件在修改的时候都应该注意，冒号为英文输入法状态下的半角字符，并且每个冒号后面都有一个空格，这个空格也是英文状态下的。
 * 找到 deploy 类属性
 ```
deploy:
type: github
repository: git@github.com:yourname/yourame.github.com.git
branch: master
```

 **注意：** 此处的 `yourame` 是你的 GitHub 帐户名。
 
### 将所选择的主题修改成你想要样子
 
####修改博客的主要参数
 打开注目下的_config.yml就可以修改博客的主要参数

```
## Hexo Configuration
## Docs: http://hexo.io/docs/configuration.html
## Source: https://github.com/hexojs/hexo/

# Site
title:  #博客名
subtitle:  #博客描述，副标题 
description: #博客描述，用于搜索引擎，不展示在博客里
author: #作者
email: #邮箱
language: zh-CN #语言

# URL
## If your site is put in a subdirectory, set url as 'http://yoursite.com/child' and root as '/child/'
url:  #你的博客的域名，后面就会说到如何绑定独立域名
root: /
permalink: :year/:month/:day/:title/ #时间格式
tag_dir: tags
archive_dir: archives
category_dir: categories
code_dir: downloads/code
permalink_defaults:
about_dir: about

# Directory
source_dir: source
public_dir: public

# Writing
new_post_name: :title.md # File name of new posts
default_layout: post
titlecase: false # Transform title into titlecase
external_link: true # Open external links in new tab
filename_case: 0
render_drafts: false
post_asset_folder: false
relative_link: false
highlight:
  enable: true
  line_number: true
  tab_replace:

# Category & Tag
default_category: uncategorized
category_map:
tag_map:

# Archives
## 2: Enable pagination
## 1: Disable pagination
## 0: Fully Disable
archive: 2
category: 2
tag: 2

# Server
## Hexo uses Connect as a server
## You can customize the logger format as defined in
## http://www.senchalabs.org/connect/logger.html
port: 4000
server_ip: localhost
logger: false
logger_format: dev

# Date / Time format
## Hexo uses Moment.js to parse and display date
## You can customize the date format as defined in
## http://momentjs.com/docs/#/displaying/format/
date_format: MMM D YYYY 
time_format: H:mm:ss

# Pagination
## Set per_page to 0 to disable pagination
per_page: 10
pagination_dir: page

# Disqus
disqus_shortname:

# Extensions
## Plugins: https://github.com/hexojs/hexo/wiki/Plugins
## Themes: https://github.com/hexojs/hexo/wiki/Themes
# Extensions
Plugins:
- hexo-generator-feed
- hexo-generator-sitemap

#Feed Atom
feed:
  type: atom
  path: atom.xml
  limit: 20

#sitemap
sitemap:
  path: sitemap.xml
  
#theme
theme: jacman #主题主目录
stylus:
  compress: true
exclude_generator:

# Deployment
## Docs: http://hexo.io/docs/deployment.html
deploy:  #GitHub 参数
  type: github
  repository: https://github.com/luozongtong123/luozongtong123.github.io.git
  branch: master
```
 
####修改主题的主要参数
 打开主题主目录下的_config.yml就可以修改主题的主要参数
 
```
##### 菜单
menu:
  主页 | Index: /
  归档 | Archives: /archives
  标签 | Tags: /tags
  关于 | About: /about

#### 控件
widgets: 
- category
- tag
- links
- rss

#### RSS 
rss: /atom.xml 

#### 图片相关
imglogo:
  enable: false               ## 是否显示网站 logo
  src: img/logo.png        
favicon: img/ico1.ico     ## 网站图标    
apple_icon: img/ico2.jpg   ## 苹果设备上的图标，背景不要透明
author_img: img/author.jpg   ## 网站底部的博主头像

#### 首页相关
index:
  expand: false              ## 首页文章是否展开。默认为展开式，显示 Read More。
  excerpt_link: Read More    

#### 作者信息
author:
  intro_line1:  "Hello , I'm balabal ."    ## 网站底部的个人介绍
  intro_line2:  "This is my blog , believe it or not ."  
  weibo_verifier:  ## 微博秀的验证码
  tsina:           ## 用于微博秀和微博分享
  weibo:           ## 用于显示网站底部社交按钮（填写账号名即可），下同
  douban:          
  zhihu: 
  email:     
  twitter:   
  github:     
  facebook: 
  linkedin:   
  google_plus:   
  stackoverflow:  


#### 目录
toc:
  article: true   ## 是否在文章中显示目录
  aside: true     ## 是否在侧边栏显示目录

#### 友情链接
links:
  码农圈: https://coderq.com,一个面向程序员交流分享的新一代社区
  Jark's Blog: http://wuchong.me

#### 评论
duoshuo_shortname:  #多说社会化评论的域名的三级域名（你特有的那部分） 
disqus_shortname:  

#### 分享按钮
jiathis:
  enable: false   ## 默认使用主题内建分享
  id:    
  tsina: 

#### 网站统计
google_analytics:
  enable: false
  id:            ## google analytics ID.
  site:          ## 网站地址.
baidu_tongji:
  enable: true
  sitecode:       ## 百度统计站点特征码，可以到百度统计申请
cnzz_tongji:
  enable: false
  siteid:        ## CNZZ统计站点ID

#### 杂项
ShowCustomFont: true  
fancybox: true        
totop: true           

#### 自定义搜索
google_cse: 
  enable: true
  cx: 001711285529756717210:xbaab-fgpk4  #谷歌站内搜索的特征码，可以到谷歌申请
baidu_search:    
  enable: false
  id:   #百度站内搜索的特征码
  site: http://zhannei.baidu.com/cse/search  

```

####修改局部页面
页面展现的全部逻辑都在每个主题中控制，源代码在hexo\themes\jackman\（jackman主题的，具体你的主题如何修改查看你的主题的帮助文档）中：

```
.
├── languages  #多语言
|   ├── default.yml#默认语言
|   └── zh-CN.yml  #中文语言
├── layout #布局，根目录下的*.ejs文件是对主页，分页，存档等的控制
|   ├── _partial   #局部的布局，此目录下的*.ejs是对头尾等局部的控制
|   └── _widget#小挂件的布局，页面下方小挂件的控制
├── source #源码
|   ├── css#css源码 
|   |   ├── _base  #*.styl基础css
|   |   ├── _partial   #*.styl局部css
|   |   ├── fonts  #字体
|   |   ├── images #图片
|   |   └── style.styl #*.styl引入需要的css源码
|   ├── fancybox   #fancybox效果源码
|   └── js #javascript源代码
├── _config.yml#主题配置文件
└── README.md  #用GitHub的都知道
```

####修改主题配色

包括颜色在内的很多变量都在jacman/source/css/_base/variable.styl（jackman主题的，具体你的主题如何修改查看你的主题的帮助文档）文件中

```
//Color
color-background = #ddd	        //文章以外的背景颜色，背景色
color-content = #413F3F         //文章字体颜色
color-font = #817C7C            //各个标签字体颜色以及友链,微博分享等的图标颜色
color-white = #ffffff           //导航栏字体颜色，右侧RSS订阅，底栏字体已及个人信息图标颜色
color-blue = #008080	        //标题字体颜色，包括超链接<a></a>以及作者名字，右侧主标签（分类，标签，友链）颜色
color-orange = #413F3F          //标题鼠标移到标题等部分color-blue处后颜色ea6753
color-theme = #008080	        //主题主颜色
color-font-nav =#ffffff         //导航栏鼠标移至后的颜色E9CD4C
color-section = #fafafa	        //文章的背景颜色
color-footer = #1f1f1f          //页面最下方的背景颜色
color-gray = #CCC               //分割线的颜色，以及标签的底色
color-meta = #808080            //图片信息字体颜色
color-heading = #333333         //部分字体颜色 ##号字体
color-quote = #f5f5f5           //四个空格后的代码块背景颜色
color-code = #eee               //小块之间的代码背景颜色
color-twitter = #00aced         //分享至Twitter处 鼠标移至后的颜色
color-facebook = #3b5998        //分享处facebook鼠标移至后的颜色
color-weibo = #c64d3e           //分享处weibo鼠标移至后的颜色
color-google = #dd4b39          //分享处google鼠标移至后的颜色
color-qrcode= #49ae0f           //分享处qrcode鼠标移至后的颜色
color-renren= #369              //分享处renren鼠标移至后的颜色
color-top = #762c54             //分享处top鼠标移至后的颜色
```

jackman的帮助文档[在这](http://wuchong.me/blog/2014/11/20/how-to-use-jacman/)。      

####404页面
直接在 `主目录\source\` 下创建自己的404.html或者404.md就可以。但是自定义404页面仅对绑定顶级域名的项目才起作用，GitHub默认分配的二级域名是不起作用的，使用hexo server在本机调试也是不起作用的。

推荐使用[腾讯公益404](http://www.qq.com/404/)。       
####发表新文章

```
hexo n postname
```
其中 postname 为文章标题，执行命令后，会在 `主目录\source_posts\` 中生成 `postname.md` 文件，用编辑器打开编写即可。

使用jacman或pacman主题，建议按此标准语法写：
```
title: postName #文章页面上的显示名称，可以任意修改，不会出现在URL中
date: 2013-12-02 15:30:16 #文章生成时间，一般不改，当然也可以任意修改
categories: example #分类
tags: [tag1,tag2,tag3] #文章标签，可空，多标签请用格式，注意:后面有个空格
description: 附加一段文章摘要，字数最好在140字以内。
---

以下正文
```

博客正文使用 Markdown 语法书写。 具体可以看这里[献给写作者的 Markdown 新手指南](http://www.jianshu.com/p/q81RER)。                  

### 本地调试以及上传部署到 GitHub

* 清除已经生成的静态网页，这个命令可以解决有时明明修改对了却没有达到本应的效果的问题。
```
hexo clean
```
* 生成静态网页
```
hexo generate
```
或
```
hexo g
```
* 启动本地服务，进行文章预览调试
```
hexo serve
```
或
```
hexo s
```
* 上传部署到 GitHub
```
hexo deploy
```

或

```
hexo d
```

也可以使用生成静态网页和部署合二为一的命令

```
hexo d -g
```

在此过程中需要输入你的 GitHub 密码。
如果你不想每次部署都输入密码的话，就请看[这里](http://zipperary.com/2013/05/26/ssh-errors-with-github/)。                       
至此你的网页已经成功部署到 GitHub ，访问 `yourname.github.io`     就可以查看你的网站了。

### 安装其他插件

在主目录下使用以下命令可以给博客生成，RSS和网站sitmap
```
npm install hexo-generator-feed 
npm install hexo-generator-sitemap
```

评论系统、网站统计、站内搜索请查看主题的帮助文档。

##设置独立域名

###购买独立域名
推荐到狗爹（[godaddy](https://www.godaddy.com)）购买，支持支付宝付款。相关内容请自行利用搜索引擎查找。
###将你的 GitHub Pages 与独立域名绑定
在 `主目录\source\` 下新建一个名为 `CNAME` 的没有文件扩展名的纯文本文档，并且在里面写好你购买的独立域名。

##DNS 设置与监控

推荐使用[DNSpod](https://www.dnspod.cn)，免费，稳定。
相关设置请自行利用搜索引擎查找。

##图床

推荐使用[七牛云存储](https://portal.qiniu.com/)，         免费10G空间，还有网站镜像加速。

##参考资料
[ 如何搭建一个独立博客——简明Github Pages与Hexo教程
](http://cnfeat.com/2014/05/10/2014-05-11-how-to-build-a-blog/)
[Hexo常见问题解决方案](http://xuanwo.org/2014/08/14/hexo-usual-problem/)
[搭建一个免费的，无限流量的Blog----github Pages和Jekyll入门](http://www.ruanyifeng.com/blog/2012/08/blogging_with_jekyll.html)                              
[一步步在GitHub上创建博客主页](http://www.pchou.info/web-build/2013/01/03/build-github-blog-page-01.html)
[hexo你的博客](http://ibruce.info/2013/11/22/hexo-your-blog/)

##鸣谢
[wuchong](http://wuchong.me/)
[cnfeat](http://cnfeat.com/)
##关于我

[关于我](http://luoluodafang.info/about/)