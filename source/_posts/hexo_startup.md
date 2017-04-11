title: hexo边搭边记
date: 2014-02-27 14:56:47
tags: hexo
---

#Install
**安装nvm（Node Version Manager）**，Terminal中运行

    $ curl https://raw.github.com/creationix/nvm/master/install.sh | sh
会提示：

    => Close and reopen your terminal to start using NVM
退出Terminal重启后nvm命令才能生效。  使用nvm安装node.js：
<!--more-->
    $ nvm install 0.10
下完后安装hexo，这一步时间比较长：

    $ npm install -g hexo

然后找个文件初始化blog：

    $ cd ~/git/blog  
    $ hexo init .
    $ ls
生成出的目录结构：

    .
    ├── _config.yml
    ├── package.json
    ├── scaffolds
    ├── scripts
    ├── source
    |   ├── _drafts
    |   └── _posts
    └── themes

新建一篇文章：

    $ hexo new mac下使用hexo搭建blog
$ open source/_posts/mac下使用hexo搭建blog.md
编辑md后生成html：

    $ hexo generate
本地预览：

    $ hexo server
    => [info] Hexo is running at localhost:4000/. Press Ctrl+C to stop.
Theme，去官方提供的[主题列表][1]中选个现成的，按照里面的方法pull下来，如light主题


    $ git clone git://github.com/tommy351/hexo-theme-light.git themes/light

_config.yml配置文件中设置：

    theme: light
重新generate和server预览，就看到变化了。

##deploy

github上建个respository，设置里设一下
在`_config.yml`中：
``` yml
deploy:
type: github
repository: https://github.com/sunnyxx/blog-hexo.git
```
然后执行：

    $ hexo deploy
就行了，github会多一个branch，比octopress简单

##绑定域名
去万网买了这个域名`sunnyxx.com`，以`blog.sunnyxx.com`作为博客的域名，
看万网是阿里的才从那儿买的，后来发现`DNSPod`貌似比较好，万网的后台做的那叫一个*，但愿解析速度上别再不行就行。
托管在github上，首先建一个CNAME文件，里面写`最终指向`的域名：
``` bash
$ blog.sunnyxx.com > public/CNAME
```
然后去域名后台配置下，由于github表示说我托管的页面的域名是：`sunnyxx.github.io`
所以建一个CNAME记录，将`blog.sunnyxx.com`解析到`sunnyxx.github.io`
DNS域名解析最常用的是**A记录**和**CNAME记录**，A记录把域名解析到服务器IP，CNAME相当于把一个域名指向另一个域名，因此我这个用的是CNAME，要是托管的服务器也是自己搭的那就用A记录了。
完事之后得等一段时间（DNSPod说几秒内就同步完，要是这样真心是它好）
使用下面的命令测一下域名的解析
``` bash
$ dig blog.sunnyxx.com +nostats +nocomments +nocmd

=> output:
;; global options: +cmd
;blog.sunnyxx.com.      IN  A
blog.sunnyxx.com.   1778    IN  CNAME   sunnyxx.github.io.
sunnyxx.github.io.  1778    IN  CNAME   github.map.fastly.net.
github.map.fastly.net.  30  IN  A   103.245.222.133
```
这说明是成功了，发现解析过程是`blog.sunnyxx.com`->`sunnyxx.github.io`->`github.map.fastly.net`->`103.245.222.133` 最终指向了github的web server
由于国内GreatWall，解析速度明显不稳定，有时候都连不上，以后再看怎么办吧

##添加sitemap
同样的，我们使用hexo提供的插件，方法与添加RSS类似。
安装sitemap到本地：
```
npm install hexo-generator-sitemap
```
开启sitemap功能：编辑hexo/_config.yml，添加如下代码：
```
plugins:
- hexo-generator-sitemap
```
访问zipperary/sitemap.xml即可看到站点地图。不过，sitemap的初衷是给搜索引擎看的，为了提高搜索引擎对自己站点的收录效果，我们最好手动到google和百度等搜索引擎提交sitemap.xml。



##文章中插入图片

原来用octopress写的时候在目录下面建个`images`目录来保存图片，引用时使用了相对路径就行了，但这是让我最蛋疼的事，想发个图片还得命个名，然后`mv`过去，再引进来，特别墨迹。hexo中当然也可以用这种方法，但是发现使用个`图床`来搞定图片真是一劳永逸了。


**微博图床**，地址http://weibotuchuang.sinaapp.com/，我是chrome用户，所以下了个他的插件，装完了点开发现直接把图片拖进去就行了：
![enter image description here][3]
生成的地址直接用就行了
有时候需要使用QQ截屏，只要随便截到某个人的聊天窗口里面，不用发送，然后直接拖拽到这个小窗口里面就OK了。
> Life saver.

PS：gif也没问题
![1231231][2]
##StackEdit - 超赞的Markdown在线编辑器  

<img src="http://ww2.sinaimg.cn/large/51530583gw1ee1a7efwa0j20hk03kwen.jpg" width="500px" />

界面干净，所见即所得，支持同步到`Dropbox`和`github`，这个特别好，比如在公司写了点东西就可以一键同步，保留犯罪现场，回来之后继续写剩下的。

遇到的问题
-----
第二次用的时候发现安装的hexo命令找不到了，重新use一下：
```
$ nvm use 0.10
```

**使用StackEdit时改变图片大小**

```
<img src="http://ww2.sinaimg.cn/large/51530583gw1ee1a7efwa0j20hk03kwen.jpg" width="200px align="center" />
```
效果：
  <img src="http://ww2.sinaimg.cn/large/51530583gw1ee1a7efwa0j20hk03kwen.jpg" width="200px" />

##添加友情链接

`themes/xxxxxx/layout/_widget/blogroll.ejs`
```
<div class="widget tag">
<h3 class="title">友情链接</h3>
<ul class="entry">
<li><a href="http://zhouxl.github.io" title="小六">小六的博客</a></li>
</ul>
</div>
```

  [1]: https://github.com/tommy351/hexo/wiki/Themes
  [2]: http://ww2.sinaimg.cn/large/51530583gw1ee17y3p11zg207804lnhh.gif
  [3]: http://ww2.sinaimg.cn/large/51530583gw1ee18d6ak6yj208c08ydg6.jpg
  [4]: https://stackedit.io/res-min/img/logo-promo-128.png
