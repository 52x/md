---
title: Hexo优化（1）：添加新主题
date: 2016-03-24 19:31:54
tags: Hexo
toc: true
---

## 配置主题landscape-plus

首先切换到博客根目录下，使用如下命令安装landscap-plus:

```bash
git clone https://github.com/xiangming/landscape-plus.git themes/landscape-plus
```

* 然后修改根目录下的配置文件_config.yml, 把theme选项的值设置为：landscape-plus。

<!--more-->

* 配置主题目录下的配置文件_config.yml， 把menu菜单项中的各选项配置为自己喜欢的样式，比如把英文的菜单改为中文的。

```bash
menu:
  首页: /
  文章列表: /archives
  关于: /about
```

## 总结

Hexo的其他主题安装配置也是类似的，大家可以去[Hexo主题官网](https://hexo.io/themes/)下载自己喜欢的主题。

## 参考
* [Hexo主题配置与优化(1)](http://starsky.gitcafe.io/2015/05/05/Hexo%E4%B8%BB%E9%A2%98%E9%85%8D%E7%BD%AE%E4%B8%8E%E4%BC%98%E5%8C%96%EF%BC%88%E4%B8%80%EF%BC%89/)
