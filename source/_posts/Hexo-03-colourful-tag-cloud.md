---
title: Hexo优化（3）：彩色标签云
date: 2016-03-24 20:01:04
tags: Hexo
toc: true
---

在你的主题目录\layout\_widget\下找到tagcloud.ejs文件，编辑这个文件，找到标签<%-tagcloud，然后把整个代码修改为如下样式：

```js
<% if (site.tags.length){ %>
  <div class="widget-wrap">
    <h3 class="widget-title"><%= __('tagcloud') %></h3>
    <div class="widget tagcloud">
      <%- tagcloud(site.tags, {
        min_font: 13,
        max_font: 23,
        amount: 65,
        orderby: 'count',
        color: true,
        start_color: '#9900FF',
        end_color: '#FF0000'
      }) %>
    </div>
  </div>
<% } %>
```

其中，start_color为颜色变化的起始端，end_color为颜色变化的结束端。

<!--more-->

## 参考

* [Hexo彩色云标签](http://starsky.gitcafe.io/2015/05/16/Hexo%E5%BD%A9%E8%89%B2%E6%A0%87%E7%AD%BE%E4%BA%91/)
