title: Hexo博客优化
date: 2015-09-03 22:55:10
categories: Hexo|博客技术
tags: [Hexo, blog, git]
---
现在这个博客基本处于裸奔状态，除了默认的Landscape模板和仅有的两三篇文章以外，其它设置几乎是空白。对于处女座而言简直不能忍。
<!-- more -->
# 1. 设置Hexo博客主题
## 1.1 Hexo主题库
干正事儿之前，先去Hexo的[主题库](https://github.com/hexojs/hexo/wiki/Themes)里溜达溜达，找一些喜欢的主题备着：
* [Alex](https://github.com/ppoffice/hexo-theme-alex)-[demo](http://ppoffice.github.io/hexo-theme-alex/): 色彩和字体搭配都是我喜欢的。
* [Hueman](https://github.com/ppoffice/hexo-theme-hueman)-[demo](http://blog.zhangruipeng.me/hexo-theme-hueman/): 类似lofter的板式，视觉效果好，对个人博客来说略繁琐。
* [Landscape plus](https://github.com/xiangming/landscape-plus)-[demo](http://jasonxiang.com/landscape-plus/): 对每篇文章长度无限制，限制了单页文章浏览。
* [Mabao](https://github.com/moretwo/hexo-theme)-[demo](http://moretwo.github.io/): 纯粹喜欢顶置图片。。。
* [Metro Light](https://github.com/halfer53/metro-light)-[demo](http://halfer53.github.io/): 灰白底色搭配有点费眼，但是整体搭配赞。
* [Modernist](https://github.com/heroicyang/hexo-theme-modernist)-[demo](http://heroicyang.com/): 最喜欢的模板，TL归档设定和标签页独立的设计都不错，不知道有没有分类归档。[NexT](https://github.com/iissnan/hexo-theme-next)模版也是同样效果。
* [TKL](https://github.com/SuperKieran/TKL)-[demo](http://go.kieran.top/): 如文档描述所说，一个设计优雅的响应式主题。

## 1.2 安装主题
相较之下，我选择了modernist。
用一句命令即可安装主题：

```bash
git clone https://github.com/heroicyang/hexo-theme-modernist.git themes/modernist
```

注意这句命令需要在Hexo搭建的文件夹(我的是E:\Hexo)中打开Git Bash，如果右键无法打开，需要在Git Bash中输入：

```bash
cd E:\Hexo
```

然后再输入`git clone`的命令。安装完成后，打开E:\Hexo下的_config.yml文件，修改theme参数。我的默认值是Landscape，把它修改成`theme: modernist`，注意`theme:`后的空格。

# 2. 模板参数设置
## 2.1 默认参数列表
然后再打开E:\Hexo\themes\modernist目录下的_config.yml文件，默认参数如下：

```YAML
menu:
  Home: /
  Archives: /archives
rss: /atom.xml

archive_date_format: MMM DD
fancybox: true

duoshuo_shortname:

google_analytics:
favicon: /favicon.ico
```
可根据自己的喜好进行修改，我参考了NextT模板的文档设置。

## 2.2 置顶菜单

```YAML
  Home: /					#首页
  Categories: /categories 	#分类归档
  Archives: /archives 		#时间线归档
  Tags: /tags 				#标签
  About: /about 			#关于
```

## 2.3 博客统计

众所周知的原因，G.A.不稳定，所以用百度统计(百度统计需要注册)：

```YAML
baidu_tongji: true
```

## 2.4 边栏
三种参数：`post`, `always`, `hide`
`always`表示在所有页面自动展开边栏；`hide`表示只在点击边栏按钮的时候展开；默认设置`post`表示在文章旁边自动展开。
```YAML
sidebar: post
```

## 2.5 摘要
每篇文章的摘要，每篇文章摘要前1500字。
```YAML
auto_excerpt:
  enable: true
  length: 1500
```

国内最常用的评论框多说，安装后会直接拷贝文章信息和所有评论信息。所以评论框和其它设置有待研究。

# 3. 更新主题
用Git Bash进入themes/modernist，输入命令
```Bash
cd themes/modernist
git pull
```

看看更换主题后的效果吧！

---------
# 更新主题 9/6/2015

发现Modernist的demo是NexT的效果。。。于是果断换成NexT
NexT有完整的[安装教程](http://theme-next.iissnan.com/)，安装好以后记得将蘸点配置文件里的`theme: `项改成`theme: next`。
知乎上也有hexo主题推荐：[有哪些好看的Hexo主题](www.zhihu.com/question/24422335)

---------
**参考资料**
1. [hexo博客的配置、使用](http://zipperary.com/2013/05/29/hexo-guide-3/)
2. [hexo博客的优化技巧](http://zipperary.com/2013/05/30/hexo-guide-4/)
3. [NexT主题设置参考](https://github.com/iissnan/hexo-theme-next/blob/master/_config.yml)
3. [Wordpress第三方评论插件的利弊](http://www.williamlong.info/archives/3893.html)
