---
title: iOS dicom文件的解析及操作
date: 2017-04-01 14:19:07
tags: DICOM
categories: DICOM
---

最近老大要研究一下dicom方面的知识，刚听到这个词的时候，完全懵逼了。dicom是什么？吓得赶紧去度娘：<!--more-->[DICOM（Digital Imaging and Communications in Medicine）即医学数字成像和通信，是医学图像和相关信息的国际标准（ISO 12052）。它定义了质量能满足临床需要的可用于数据交换的医学图像格式](http://baike.baidu.com/link?url=-ToIkDi93p12jv0_E1a6T3j4u22zuI5OO33UyDNWFBfj_y3ix8wDSsFplJpj8ivcguvhqlaB8bQuc_q4SPwwK_)。这一看，好家伙，无数个问号再加无数个😳在脑中冉冉升起。

不停的google和baidu，但是网上的资料都是大部分都是c++的，对我这样一个iOS的来说挑战性很大。柳暗花明又一村。找了好久，找到了一个[大神的杰作](http://blog.csdn.net/kesalin/article/details/6986274)。这套源代码是2011年写的，所以拿到后改了改，然后在网上遇到了一个同样是在做dicom移动端的哥们儿，俩人不停的探讨，才有了现在一点点的小成果。

先上一个效果图：

![这是文件列表](http://upload-images.jianshu.io/upload_images/2074437-e693b3f9651ee192.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

再来一张：

![这是解析粗来的](http://upload-images.jianshu.io/upload_images/2074437-f0733ed269364e20.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

再来个标注效果的：

![这就是标注效果的](http://upload-images.jianshu.io/upload_images/2074437-7511b735e05a438a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


由于现在很多功能还没有实现，所以还需要不断完善！后面会不定期更新的！

最后附上：[源码地址](https://github.com/ZJQian/Dicom)


