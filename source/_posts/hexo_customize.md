title: hexo的私人订制
date: 2014-03-07 16:26:34
tags: hexo
---

#准备工作
##Fork it!
从0开始多费劲，先从hexo的主题中选一个看的过去的，从上面加工。
这次选的是hexo的默认主题`Landscape`，觉得一个大banner挺好看而已。
主题在github上，https://github.com/hexojs/hexo-theme-landscape  
废话不说，先fork一份，虽然不会再merge回去了。
fork完去setting页面改个名字，就叫它`present`了，因为当时看到群里正说`presentViewController`的事- -

<!--more-->

![][1]
 * hexo工程的`themes/`目录默认是在`.gitignore`里的，意思是主题和内容是应该分开的
theme作为主项目的`submodule`，所以主题更改时也应该单独提交了

##Clone it！
把刚fork的名为`present`的theme安装到hexo目录：
``` sh
$ git clone https://github.com/sunnyxx/present themes/present
```
去`/_config.yml`中找到并设置：
```
# Extensions
## Plugins: https://github.com/tommy351/hexo/wiki/Plugins
## Themes: https://github.com/tommy351/hexo/wiki/Themes
theme: present // 修改这儿
```
运行下`hexo server`就能立刻看到效果了

#开始定制theme
先得看看hexo theme里面的结构：

 - _config.yml - 主题总体配置
 - /layout/*.ejs - 网页布局
 - /source/css/*.styl - 网页样式

##定制banner
![banner][2]
默认的banner图是个地球星空图，先从它下手，这张图位于`/themes/present/source/css/images/banner.jpg`
这图分辨率有`1920x1200`之大，显示的部分很少，搞一张喜欢的banner图，PS成大概的尺寸（高度还就得设的很大才行，虽然只显示一小部分，否则会出现显示不出来图片的状况），我这儿PS过的是一张png，名为`banner.png`，文件名修改需找找到位于`/themes/present/source/css/_variables.styl`中，修改为`banner.png`  
**当然，考虑到github的访问速度，这张大图我最后决定用上传微博图床，使用生成的URL，这样加载就快很多**
```
// Header
logo-size = 40px
subtitle-size = 16px
banner-height = 404px // 看看多高合适
banner-url = "images/banner.png" // 修改这里
```
然后就变成这鸟样了：
![专业多了][3]
这个标题横在这儿太恶心了，我的banner里面已经有标题了，这就是需要修改布局了，header的布局在`/themes/present/layout/_partial/header.ejs`，打开修改：
```
// 把这一段都注释掉好了：
<div id="header-title" class="inner">
    <h1 id="logo-wrap">
        <a href="<%- config.root %>" id="logo"><%= config.title %></a>
    </h1>
    <% if (theme.subtitle){ %>
        <h2 id="subtitle-wrap">
          <a href="<%- config.root %>" id="subtitle"><%= theme.subtitle %></a>
        </h2>
    <% } %>
</div>
```
然后世界变清净了。
![丑][4]
默认主题里面banner上下都有个渐变，换了图之后就尤其丑，干掉之。
这个是样式的修改，所以肯定在`present/source/css/_partial`里面了，再看这位置明显是header嘛，所以`header.styl`就是你了：
```
#header
  height: banner-height
  position: relative
  border-bottom: 1px solid color-border
  &:before, &:after
    content: ""
    position: absolute
    left: 0
    right: 0
    height: 40px
  &:before
    top: 0
    background: linear-gradient(rgba(0, 0, 0, 0.2), transparent) // 找到你了，注释掉！
  &:after
    bottom: 0
    background: linear-gradient(transparent, rgba(0, 0, 0, 0.2)) // 也找到你了，注释掉！
```
瞬间清爽很多，但又发现去掉之后字看不清了：
![看不清！][5]
还是在这个文件：

```
$nav-link
  float: left
  color: #000 // 改个深色
  opacity: 1.0 // 别半透明的
  text-decoration: none
  /*text-shadow: 0 1px rgba(0, 0, 0, 0.2)*/ // 改个阴影
  transition: opacity 0.2s
  display: block
  padding: 20px 15px
  &:hover
    opacity: 1
```
much better
![better][6]

##定制样式

列几个常用的：

在`/themes/present/source/css/_variables.styl`中：

####调整主区域布局（很值得修改）
默认的主题的主区域太窄了，没几个字就得换行，下面的`main-column`控制主区域宽，`sidebar-column`控制sidebar宽，这两个值加一起凑成全部宽度，会居中对齐。
```
// Layout
block-margin = 20px
article-padding = 20px // 文章内缩进
mobile-nav-width = 280px
main-column = 12 // 主文章区域的宽度
sidebar-column = 3 // 侧边栏区域的宽度
```
####修改代码字体

```
font-mono = Menlo/*Menlo必须提前面啊*/, "Source Code Pro", Monaco, Consolas, Consolas, monospace
```
####修改正文字体和行高
```
font-size = 15px // Menlo字体我看15px的很清楚
line-height = 1.6em
line-height-title = 1.3em
```


-----
位于`/themes/present/source/css/_partial/article.styl`的样式文件负责文章里面的样式
####修改图片格式
```
  img, video
    max-width: 80%
    height: auto
    display: block
    margin: auto
```
去除图片的描述的caption的话，去`present/source/js/script.js`中修改：
```
// Caption
  $('.article-entry').each(function(i){
    $(this).find('img').each(function(){
      if ($(this).parent().hasClass('fancybox')) return;

      var alt = this.alt;

      // if (alt) $(this).after('<span class="caption">' + alt + '</span>'); 这个去掉

      $(this).wrap('<a href="' + this.src + '" title="' + alt + '" class="fancybox"></a>');
    });

    $(this).find('.fancybox').each(function(){
      $(this).attr('rel', 'article' + i);
    });
  });
```

####修改blockquote样式
```
  blockquote
    font-family: font-serif
    font-size: 2.0em // 搞大点
    margin: line-height 20px
    text-align: left // 必须应该左对齐啊
```

-----
位于`/themes/present/source/css/_extend.styl`的样式文件定义了基本样式
####修改标题样式
```
  h1
    font-size: 2em
  h2
    font-size: 1.5em
  h3
    font-size: 1.3em
  h4
    font-size: 1.2em
  h5
    font-size: 1em
  h6
    font-size: 1em
    color: color-grey
```

####修改文章背景
```
$block
  background: #fbfbfb // 白里透着灰
  /*box-shadow: 1px 2px 3px #eee*/ // 扁平化咋能要阴影
  border: 1.5px solid #ccc // 边框
  border-radius: 10px // 圆角矩形走起
```

##定制代码样式
这个必须单拿出来写  
> We shall show no mercy to those shit colored codes ----- sunnyxx

代码的高亮样式在`present/source/css/_partial/highlight.styl`中
```
$code-block
  background: highlight-background
  /*margin: 0 article-padding * -1*/
  margin: auto // 默认的顶边对齐是怎么回事？改个居中
  padding: 15px article-padding
  border-style: solid
  border-color: color-border
  border-width: 0px 0
  border-radius: 5px // 加个圆角~
  overflow: auto
  color: highlight-foreground
  line-height: font-size * line-height

$line-numbers
  color: #666
  font-size: 0.85em // 行号大小

...

.highlight
    @extend $code-block
    pre
      border: none
      margin: 0
      padding: 0
    table
      margin: 0
      width: auto
      font-size: 14px // 设置代码字体
      letter-spacing: 1px // 设置字间距，要不太挤了

```
Code block高亮：`我是小代码块高亮`
```
.article-entry
  pre, code
    font-family: font-mono
  code
    background: #e3e3e3
    color: #666
    border-radius: 3px // 也来个圆角
    border-width 1px
    border-color: #fff
    text-shadow: 0 1px #fff
    padding: 0.1em 0.3em // 控制大小
```

#开始定制widget

##添加多说评论
在`present/layout/_partial/article.ejs`中最下面，要不用discuss的话先注掉，换成下面的：

```
<% if (!index && post.comments){ %>
<section id="comments">
  <!-- Duoshuo Comment BEGIN -->
  <div class="ds-thread"></div>
    <script type="text/javascript">
      var duoshuoQuery = {short_name:"sunnyxx"};
        (function() {
          var ds = document.createElement('script');
          ds.type = 'text/javascript';ds.async = true;
          ds.src = 'http://static.duoshuo.com/embed.js';
          ds.charset = 'UTF-8';
          (document.getElementsByTagName('head')[0]
          || document.getElementsByTagName('body')[0]).appendChild(ds);
        })();
  </script>
<!-- Duoshuo Comment END -->
</section>
<% } %>
```
##添加友情链接
首先，在`present/layout/_widget/`目录下新建一个文件，随便copy个当前目录下的改名也行，我这儿叫`friends.ejs`
编辑这个文件：
```
<div class="widget tag">
	<h3 class="title">友情链接</h3>
	<ul class="entry">
	<li><a href="http://zhouxl.github.io" title="小六">小六的博客</a></li>
	</ul>
</div>
```
里面以上面的格式定义友情链接，css套用了tag定义好的格式，随后修改`present/_config.yml`
```
# Sidebar
sidebar: right
widgets:
- category
- tag
- tagcloud
- archive
- recent_posts
- friends // 新加的就是刚才`_widget`目录中新建的文件的文件名
```
然后刷新页面，效果就出来了~

  [1]: http://ww3.sinaimg.cn/large/51530583gw1ee7835uauoj20j804mwen.jpg
  [2]: http://ww4.sinaimg.cn/large/51530583gw1ee78mkdkspj20jp06bq3h.jpg
  [3]: http://ww2.sinaimg.cn/large/51530583gw1ee790pxkk9j20fv085t9c.jpg
  [4]: http://ww3.sinaimg.cn/large/51530583gw1ee79b2xop4j201u0bt0sm.jpg
  [5]: http://ww4.sinaimg.cn/large/51530583gw1ee79ex02pgj206901u0qo.jpg
  [6]: http://ww3.sinaimg.cn/large/51530583tw1ee79mmlwkaj205401iwe9.jpg
