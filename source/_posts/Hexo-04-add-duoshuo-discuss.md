---
title: Hexo优化（4）：添加多说评论最近评论
date: 2016-03-24 20:13:37
tags: Hexo
toc: true
---

## 安装多说评论系统

实际上landscape-plus主题上已经集成了多说评论系统，我们只需要添加duoshuo_shortname到两个配置文件就行了。

多说的shortname就是你注册多说时的用户名。

接下来在博客根目录下的配置文件和主题下的配置文件_config.yml中加入如下代码：

```bash
# Duoshuo
duoshuo_shortname: XXX
```

## 添加多说最近评论

以landscape-plus主题为例：

<!--more-->

在landscape-plus\layout\_widget\目录下新建recent_comments.ejs文件，内容如下：

```bash
<% if (theme.duoshuo_shortname){ %>
<div class="widget-wrap">
  <h3 class="widget-title"><%= __('最近评论') %></h3>
<div class="widget">
<ul class="ds-recent-comments" data-num-items="5" data-show-avatars="1" data-show-time="1" data-show-title="1" data-show-admin="1" data-excerpt-length="70"></ul>
</div>
  </div>
<% } %>
```

注：其中上述代码第5行`<ul class="ds-recent-comments" data-num-items="5" data-show-avatars="1" data-show-time="1" data-show-title="1" data-show-admin="1" data-excerpt-length="70"></ul>`按照官方提示可以自行修改：


```bash
//以下参数均为可选
data-num-items="10"     //显示最新评论的条数，最大支持200条
data-show-avatars="1"   //是否显示头像，1：显示，0：不显示
data-show-time="1"      //是否显示时间，1：显示，0：不显示
data-show-title="0"     //是否显示标题，1：显示，0：不显示
data-show-admin="1"     //是否显示管理员的评论，1：显示，0：不显示
data-excerpt-length="70"//最大显示评论汉字数
```

然后在landscape-plus目录下的_config.yml下的widgets中添加recent_comments：

```bash
widgets:
- category
- recent_posts
- archive
- tagcloud
- tag
- links
- recent_comments
```

## 参考

[如何搭建一个独立博客](http://www.jianshu.com/p/05289a4bc8b2#)
