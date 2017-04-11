title: 网页中font-family的属性解析
date: 2016-01-01 22:38:43
tags: [前端,css]
---

## web中文字体的选择

web应用程序因其跨平台性被广泛应用，但是也为web应用程序运行带来了复杂的运行环境，比如各个系统字体的区别以及中英文字体显示的区别。

#### 字体分类
网页常用字体通常分为5类：serif(衬线)、sans-serif(无衬线)、monospace(等宽)、fantasy(梦幻)、cuisive(草体)，这些通用的名称允许用户代理从相应集合中选择一款字体。

* serif 字体在字符笔画末端有叫做衬线的小细节，这些细节在大写字母中特别明显。
* sans-serif 字体在字符笔画末端没有任何细节，与serif字体相比，他们的外形更简单。
* monospace 字体，每个字母的宽度相等，通常用于计算机相关书籍中排版代码块。
* fantasy 和 cuisive 字体在浏览器中不常用，在各个浏览器中有明显的差异。

#### 网页常用字体

 ##### Sans-serif:

* Helvetica: 被评为设计师最爱的字体，Realist风格，简洁现代的线条，非常受到追捧。在Mac下面被认为是最佳的网页字体，在Windows下由于Hinting的原因显示很糟糕。

* Arial: Helvetica的「克隆」，和Helvetica非常像，细节上比如R和G有小小差别。如果字号太小，文字太多，看起来会有些累眼。Win和Mac显示都正常

* Lucida Family: Lucida Grande是Mac OS UI的标准字体，属于humanist风格，稍微活泼一点。Mac下的显示要比Win下好。

* Verdana: 专门为了屏显而设计的字体，humanist风格，在小字号下仍可以清楚显示，但是字体细节缺失严重，最好别做标题。

* Tahoma: 也是humanist风格，字体和Verdana有点像，但是略窄一些，counter略小，曾经是Windows的标准字体，Mac 10.5之后默认也有安装。

* Trebuchet MS: 为微软设计的一个humanist风格字体，个人觉得个性太过突出，用得不好会不搭。

##### Serif：

* Georgia: 基本上适合正文屏显的衬线字体，非Georgia莫属了。笔画粗重，衬线明线，轮廓较大，小字体显示也很清晰，同时细节还算OK。

* Times: Times是为了报纸而设计的，特点是可以在有限的空间塞进去更多的文字，笔画较弱，小字号正文屏显看起来累眼。曾经Engadget改版的时候用了Times作为正文，被骂得很惨之后换成了Georgia。

##### 中文：

* 宋体：Win最常见的字体，小字体点阵，大字体TrueType，但是大字体并不好看，所以最好别做标题。

* 微软雅黑：Vista之后新引入的字体，打开Cleartype之后显示效果不错，不开Cleartype发虚。

* 华文细黑：Mac下的默认中文。


## font-family写法
* 由于某些系统可能不存在中文字体的中文名，所以写中文字体别忘了添加英文名

* 因为英文字体的渲染通常比用中文字体渲染的效果好，所以英文字体声明在中文字体前。

* 尽量在各个系统中都显示的最好

由此可得出font-family的写法：

```js
font-family: Arial, STXihei, "华文细黑", "Microsoft YaHei", "微软雅黑",SimSun, "宋体" ;
```

这样写基本可以保证主流系统在显示网页字体的时候效果最好，当然要注意的是：设计师在设计网页的时候也需要照顾不同的平台的字体属性