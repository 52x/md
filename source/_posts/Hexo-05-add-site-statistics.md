---
title: Hexo优化（5）：添加网站访问统计
date: 2016-03-25 14:51:59
tags: Hexo
toc: true
---

本文主要说明[不蒜子](http://ibruce.info/2015/04/04/busuanzi/)访问统计在Landscape-plus主题下的应用，理论上下面的方法同样适用于Landscape主题。
关于light主题下不蒜子的应用可以参考：[给hexo配置上评论和访问量](http://jackroyal.github.io/2015/05/30/hexo-setting-with-comments-and-visitors/)

## 安装脚本

这是使用不蒜子的前提，即要使用它必须先添加它的脚本。
打开themes\landscape-plus\layout\_partial\after-footer.ejs,在最后添加上下面的脚本即可，当然你也可以添加到 header.ejs 或 footer.ejs 中。目前最新版本2.3。

```bash
<script async src="https://dn-lbstatics.qbox.me/busuanzi/2.3/busuanzi.pure.mini.js"></script>
```

## 网站访问量统计

<!--more-->

打开themes/landscape-plus/layout/_partial/footer.ejs，在需要的地方添加下面的代码，可以选择添加任意一种代码或同时添加两种代码。
算法a：pv的方式，单个用户连续点击n篇文章，记录n次访问量。

```bash
<span id="busuanzi_container_site_pv">
    本站总访问量<span id="busuanzi_value_site_pv"></span>次
</span>
```

算法b：uv的方式，单个用户连续点击n篇文章，只记录1次访客数。

```bash
<span id="busuanzi_container_site_uv">
  本站访客数<span id="busuanzi_value_site_uv"></span>人次
</span>
```

本站是在Theme by <a href="https://github.com/xiangming/landscape-plus" target="_blank">Landscape-plus</a>语句后面添加下面的代码：

```bash
<!-- 不蒜统计 -->
<br>
<span style="display: inline;" id="busuanzi_container_site_uv">本站总访客数 <span id="busuanzi_value_site_uv" font="微软雅黑" style="color:white"></span> 次</span>
<span style="display: inline;" id="busuanzi_container_site_pv">本站总访问量 <span id="busuanzi_value_site_pv" font="微软雅黑" style="color:white"></span> 次</span>
```

## 文章访问量和评论数统计

不蒜子之所以称为极客的算子，正是因为不蒜子自身只提供标签+数字，至于显示的style和css动画效果，任你发挥。

busuanzi_value_site_pv 的作用是异步回填访问数，这个id一定要正确。
busuanzi_container_site_pv的作用是为防止计数服务访问出错或超时（3秒）的情况下，使整个标签自动隐藏显示，带来更好的体验。这个id可以省略。

因此，你也可以使用极简模式：

```bash
本站总访问量<span id="busuanzi_value_site_pv"></span>次
本站访客数<span id="busuanzi_value_site_uv"></span>人次
本文总阅读量<span id="busuanzi_value_page_pv"></span>次
```

## 修改多说样式

由于landscape-plus已经集成了多说评论，因此，只需要按照[给hexo配置上评论和访问量](http://jackroyal.github.io/2015/05/30/hexo-setting-with-comments-and-visitors/)一文中，在多说的管理页面找到配置->自定义文本，找到暂无评论,1条评论,{num}条评论,这几个设置,修改成自己要的格式,也可以参照我的修改,0,1,{num}。

## 配置文章访问量和评论数

1.修改themes/landscape-plus/layout/_partial/article.ejs,在header标签的末尾添加以下代码:

```bash
<% if (post.excerpt && index){ %> 
<% } else { %> 
<div class="busuanzi_container_page_pv"> 
  <span class="head-plus">   
	<i class="fa fa-user"></i>
 <span id="busuanzi_value_page_pv">
	<i class="fa fa-spinner fa-spin"></i>
 </span>次访问   </span>  
 <span class="head-plus">  
 	<i class="fa fa-comments"></i>
 <span class="ds-thread-count" data-thread-key="<%= post.path %>">
	<i class="fa fa-spinner fa-spin"></i></span>条评论  
 </span> 
</div> 
<% } %>
```

## 优化样式

打开themes\landscape-plus\source\css\_partial\article.styl,在.article-entry前面添加下面的代码：

```bash
.busuanzi_container_page_pv
    margin:20px 0
    color: #817C7C
    font-size: 12px
#busuanzi_value_page_pv
    padding-left:4px
.head-plus
    padding-left:4px
.ds-thread-count
    padding-left:4px
```

## 参考

* [Hexo博客添加网站统计](http://blog.wleyuan.me/2015/07/26/Hexo-AddStatistics/)
* [给hexo配置上评论和访问量](http://jackroyal.github.io/2015/05/30/hexo-setting-with-comments-and-visitors/)
* [Hexo使用多说教程](http://dev.duoshuo.com/threads/541d3b2b40b5abcd2e4df0e9)
* [不蒜子](http://ibruce.info/2015/04/04/busuanzi/)
