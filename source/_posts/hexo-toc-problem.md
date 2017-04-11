title: Hexo toc 无法显示
date: 2015-02-11 03:00:55
tags: [hexo,]
---
## 问题
使用Hexo渲染的时候发现有的问题有TOC，有的文章没有TOC。经过一番探索研究，找到了问题所在。

<!--more-->

## 排查



- 换了Chrome浏览器之后发现问题依旧。排除浏览器的原因。



- `hexo clean`  `hexo g`之后发现问题依旧，排除旧模板的原因。



- 更改_config模板配置文件TOC模块之后问题依旧，排除模板配置原因。



- 打开Firebug，查看JS代码之后发现H2如果标题没有的话会被设定`display:none`
![](http://harchiko.qiniudn.com/hexo-toc-problem/toc_not_show.png)

至此！谜题就此揭开!

## 解决办法

写的时候添加个二级标题。也就是`##`
