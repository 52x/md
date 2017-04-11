title: Hexo 更改默认 Google 字体
date: 2015-01-27 01:11:15
tags: hexo

---

因为一些国内的客观原因，google的访问速度一直很慢，通过firebug看了下google 相关的服务加载速度超过了1s种，而且经常加载不成功。


<!--more-->

更改方法：

terminal 中查找themes里对应googlefont的数据。

```bash
	grep -ir googlefont themes/
```

找到对应的地方替换为360的字体源：

360 前端CDN网站：[http://libs.useso.com/](http://libs.useso.com/ "360 前端CDN")

**本人对360不黑不粉，请纠结360的让道**


 