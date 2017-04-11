title: xpath 简明教程
date: 2015-01-23 06:45:22
tags: xpath
toc: true

---
## 什么是xpath?

> XPath 是一门在 XML 文档中查找信息的语言。XPath 可用来在 XML 文档中对元素和属性进行遍历。

<!--more-->

## XPath 术语

|术语|描述|
|-|-|
|节点(node)|在 XPath 中，有七种类型的节点：元素、属性、文本、命名空间、处理指令、注释以及文档（根）节点。XML 文档是被作为节点树来对待的。树的根被称为文档节点或者根节点。|
|基本值（或称原子值，Atomic value）|无父无子的节点|
|项目（Item）|基本值或节点|
|节点关系|父、子、同胞等|


## XPath 用法

选取节点

> 下面列举了最有用的节点表达式

|表达式|描述|
|-|-|
|nodename|选取此节点的所有子节点。|
|/|从根节点选取。|
|//|从匹配选择的当前节点选择文档中的节点，而不考虑它们的位置。|
|.| 	选取当前节点。|
|.. 	|选取当前节点的父节点。|
|@ |	选取属性。|
**text() 选取节点内元素**

下面以一段xml代码为例：

```xml
<?xml version="1.0" encoding="ISO-8859-1"?>

<bookstore>

<book>
  <title lang="eng">Harry Potter</title>
  <price>29.99</price>
</book>

<book>
  <title lang="eng">Learning XML</title>
  <price>39.95</price>
</book>

</bookstore>
```

|路径表达式 |	结果|
|-|-|
|bookstore |	选取 bookstore 元素的所有子节点。|
|/bookstore 	|选取根元素 bookstore。注释：假如路径起始于正斜杠( / )，则此路径始终代表到某元素的绝对路径！|
|bookstore/book |	选取属于 bookstore 的子元素的所有 book 元素。|
|//book |	选取所有 book 子元素，而不管它们在文档中的位置。|
|bookstore//book |	选择属于 bookstore 元素的后代的所有 book 元素，而不管它们位于 bookstore 之下的什么位置。|
|//@lang |	选取名为 lang 的所有属性。|

## 谓语

谓语用来查找某个特定的节点或者包含某个指定的值的节点。
谓语被嵌在方括号中。

|路径表达式 |	结果|
|-|-|
|/bookstore/book[1] |	选取属于 bookstore 子元素的第一个 book 元素。|
|/bookstore/book[last()] |	选取属于 bookstore 子元素的最后一个 book 元素。|
|/bookstore/book[last()-1] |	选取属于 bookstore 子元素的倒数第二个 book 元素。|
|/bookstore/book[position()<3] |	选取最前面的两个属于 bookstore 元素的子元素的 book 元素。|
|//title[@lang] |	选取所有拥有名为 lang 的属性的 title 元素。|
|//title[@lang='eng'] |	选取所有 title 元素，且这些元素拥有值为 eng 的 lang 属性。|
|/bookstore/book[price>35.00] |	选取 bookstore 元素的所有 book 元素，且其中的 price 元素的值须大于 35.00。|
|/bookstore/book[price>35.00]/title |	选取 bookstore 元素中的 book 元素的所有 title 元素，且其中的 price 元素的值须大于 35.00。|

## 实战

以 [开源中国][1]为例，我们来试试XPath能做什么？


1. 假设我们想要获取图示部分的内容：
![开源中国][2]

2. 打开Firebug，定位到这一部分：
![Firebug][3]
xpath 写为: `id('ProjectNews')/ul/li/a`

3. 使用插件 xpath checker 查看到获取到的信息
![xpath][4]

获取到了。

## 美女

比如： 下载花瓣中这个画板的图片[http://huaban.com/boards/293742/](http://huaban.com/boards/293742/)

图片如图：
![花瓣美女](http://harchiko.qiniudn.com/how-to-use-xpath/huaban.png)

要下载这里面每一个美女的图片，重复我们之前的步骤：

为了找到图片的URL，打开firebug，找到图片对应的xpath路径，经过一番尝试：`//div/div/div/div/a/img[1][@width="236"]`可以顺利找到所有的图片。

![xpath](http://harchiko.qiniudn.com/how-to-use-xpath/huaban_xpath_01.png)

通过`//div/div/div/div/a/img[1][@width="236"]/@src`即可顺利找到所有图片的URL。

![xpath](http://harchiko.qiniudn.com/how-to-use-xpath/huaban_xpath_02.png)

这样所有的xpath都获取到了。


这样就获取到了这些地址，但这些地址并不是原图大小的，对比原图与这个地址，发现后面236更改为658即可获取到大图地址。


	

  [1]: http://www.oschina.net/ "开源中国"
  [2]: http://harchiko.qiniudn.com/how-to-use-xpath/oschina.png "开源中国"
  [3]: http://harchiko.qiniudn.com/how-to-use-xpath/oschina-firebug.png "firebug"
  [4]: http://harchiko.qiniudn.com/how-to-use-xpath/xpath.png "xpath"