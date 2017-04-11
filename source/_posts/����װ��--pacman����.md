title: 博客装扮--Pacman主题
date: 2014-02-11 11:26:55
tags: [hexo]
categories: [writing]
description: 人靠衣装，佛靠金装，hexo如此牛逼的博客，怎能没个让自己看着爽心的主题装扮下呢？本博客基于Pacman主题，根据自己需求，做了少量修改，现将这次折腾之旅记录下来。

---
人靠衣装，佛靠金装，[Hexo][]如此牛逼的博客，怎能没个让自己看着爽心的主题装扮下呢？[Hexo][]版本`2.4+`的默认主题是`Landscape`，看着实在没什么感觉，于是下定决心换一个自己看着爽心的主题。看到[Alimon's blog][blog1]后，感觉这个博客的主题不错，尤其是`header`，`footer`部分，既然*中*了，那还等啥，走你！

# Pacman简介
Pacman是一款为[Hexo][]打造的*扁平化*，有着*响应式*设计的主题。*扁平化*和*响应式*在如今的手机应用、网页设计领域已是随处可见，「Win8 Metro」和「ios7」就是典型的扁平化设计，简约清晰！前端牛逼框架[Bootstrap][]也支持响应式设计，使得用户在移动设备上也有很好的体验。Pacman中所有的css都采用[Stylus][]编写，自己动手改动也比较方便，不懂前端的我对此深有感触。关于Pacman的更多详细介绍，参见[《Pacman主题介绍》][ref1]，安装、设置、更新、配置等都很详细。

# 小改动
## 主题色
Pacman默认颜色是`#ea6753`，蛮有活力的颜色，我本打算直接换一个主题色，但试了好几个，感觉一般。经过了漫无目的地思考与搜索，最后打算结合主题[paperbox][]的背景色。有了想法，动手开搞就是水到渠成了，直接贴操作过程：
```
$ vi themes/pacman/source/css/_base/variable.styl # vi仅仅是编辑器
```
修改其中的各种`颜色代码`，可以改变配色方案。我的主题的背景色是一张图片，Pacman代码的组织非常「模块化」，按照其思想，改动如下：
```
$ vi themes/pacman/_config.yml  # 添加background_img: img/noise.png
  # 下面文件中在//image位置出添加：background-img = hexo-config("background_img")
$ vi themes/pacman/source/css/_base/variable.styl
```
接下来的修改中如果需要改成该背景图片颜色，将颜色设置为`url(root+background-img)`即可。「模块化」的好处就是：接下来你如果有其它背景图片，只需在`themes/pacman/_config.yml`中修改`background_img`属性就行了。

背景色设置成图片后，本着少改动的原则，我将`header`、`footer`的主题色设置为`transparent`，就有了现在的效果。

## hover配色
简而言之，css中`:hover`伪类作用：在鼠标移到元素上时向此元素添加特殊的样式。我也是这次该主题才涉足css，还是菜鸟中的菜鸟。受益于「模块化」的好处，改动还是比较容易的。改动Pacman的css代码，请进入`themes/pacman/source/css/_partial`文件夹改动。原先Pacman的鼠标悬停时元素变成「主题色」或者「蓝色」，我统一换成了「红蓝切换」，即巴萨队服的主颜色切换了。PS：一般都是改动`&:hover`前后代码的`color`值。

## code样式
默认的代码高亮配色感觉不是很搭，Pacman代码高亮使用的是[highlight.js][]，我打算参照在[配色方案][ref2]中选取一种替代原先的配色，折腾了会不知道如何直接换，最后还是自己手动配色：
```
$ vi themes/pacman/source/css/_base/code.styl
```
无奈笨拙，只能自己选颜色代码，手动改，以后在琢磨高级的方法。

## 图片
直接替换`themes/pacman/source/img`中的图片即可，再一次，费了不少心思搞了几张图片替换了。`header`处原先是`.svg`格式的图片，我也使用软件「inkscape」转换了，可惜没有达到应有的效果，主要是在IE浏览器中图片还是残了，只能自我安慰「珍爱前端程序员生命，请远离IE」。

## 其它
想法还是挺多的，就是水平受限，其它的改动也不多，主要就是某些组件的排版，代码块背景设成圆角等等。以后边学习，边改动吧。

# 写在最后
* Pacman需要安装[Hexo][] 2.4.5 或以上版本。
* 改网页布局啥的可以使用Chrome的「F12」，各种方便。
* 不懂代码没关系，知道改哪里就行了，在折腾过程中，我经常使用全局搜索包含某行代码的文件：`find <folder> | xargs grep "目标代码"`，然后或者修改，或者参考，还是挺方便的。
* 谢谢[yangjian.me][blog1]，这么好的主题，以及代码。
* 我的博客源文件[GitHub地址][leebug]。
* 以此主题致敬「红蓝巴萨」！

[Bootstrap]: http://www.bootcss.com/
[Stylus]: http://learnboost.github.io/stylus/
[paperbox]: https://github.com/sun11/hexo-theme-paperbox
[leebug]: https://github.com/leebug38/hexo
[highlight.js]: http://highlightjs.org
[Hexo]: http://zespia.tw/hexo/
[blog1]: http://yangjian.me
[ref1]: http://yangjian.me/workspace/introducing-pacman-theme/
[ref2]: http://highlightjs.org/static/test.html