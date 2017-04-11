---
title: 使用webstorm调试Hexo
date: 2017-02-21 11:00:25
updated: 2017-02-21 11:00:25
tags: [hexo,webstorm]
categories: [Hexo]
keywords: [hexo,webstorm,debug]
---
# 为何做
  之前在编辑hexo下的插件的时候，就觉得用notepad++有点麻烦，需要确认的都要打印出来，因此考虑使用用惯了的编辑器WebStorm，但是会者不难，难者不会，牵扯到hexo命令，确实有点挠头，查找了半天，还绕了一个弯才最终解决。
  
# 如何做
  说麻烦麻烦，说简单其实也很简单，就是需要做了一个配置而已。
  我使用的环境是WebStorm，其他版本应该类似：
  `Run-->Edit Configurations...`
  ![Setup config](2017-02-21-hexo-in-webstorm/HowtoSetupHexoInStorm.png)
  如上图，选好Node的位置，设定好工作目录。稍微有点不一样的是，启动文件选择`node_modules/hexo/bin/hexo` 这个文件只有是使用了hexo-cli的初始化应该都会有。
  然后最后加上`g`这个参数，表示generate就好。当然 `g -d`也尅
  
# 绕弯子
  开始的时候，我是碰到了问题的。
  
  我使用的目录，是dropbox的目录，路径比较深。当时其实是做了一个Junction的，直接把目录映射到了F:\hexo下面。结果呢，能运行，不能断点调试，当时找了不少方法，都不行，最后想到是不是目录连接的问题，切换回正常目录，回复正常。
  当时其实还是有另一个连接，因为我之前写了一个hexo-moses的插件，用来支持我的图片，结果呢，因为放在了node_modules下面，没法git同步，于是直接在根目录下面建立了一个package的目录，结果直接在package下的hexo-moses目录下面，打开index.js文件，做debug根本不断。直到使用node_modules下面的才可以。
  嗯，就写到这里吧。

  