title: 15分钟开发一个Chrome App 【二】
date: 2016-04-24 10:51 PM
categories: "develper"
tags: [chrome,developer]
---

本文将之前的简单 APP 优化了一小步.

<!--more-->

![http://harchiko.qiniudn.com/wechat128.png](http://harchiko.qiniudn.com/wechat128.png)


> 因为 Mac 下微信实在是难用，还不如直接使用网页版的，找了一圈Chrome App Store，竟然找不到一个简单的跳转微信的APP，心想着也不会太难，就自己动起了手。

### TL;DR

你可以在这里直接下载 [https://github.com/zhaochunqi/wechat_chrome_app/releases/tag/v0.2](https://github.com/zhaochunqi/wechat_chrome_app/releases/tag/v0.2)

### 优化

之前的版本实在是太简单了，虽然能满足自己的要求，但还是希望有一点改变。

1. 能够弹出新窗口。
2. 能够使用一个合适微信大小的窗口，并且不能随意的resize。

### 修改

1. 弹出新窗口，添加 container:panel

```json
{
  "name": "WeChat",
  "description": "WeChat Bookmark",
  "version": "0.2",
  "manifest_version": 2,
  "app": {
    "urls": [
      "http://blog.zhaochunqi.com"
    ],
    "launch": {
      "web_url": "http://web.wechat.com",
      "container": "panel"
    }
  },
  "icons": {
    "128": "wechat128.png"
  }
}

```

2. 更改窗口大小，如何确定怎么更改窗口大小更合适呢？

简单，更改下窗口大小，chrome中审查下元素:

![http://harchiko.qiniudn.com/%E6%88%AA%E5%9B%BE%202016-04-24%2021%E6%97%B651%E5%88%8611%E7%A7%92.jpg](http://harchiko.qiniudn.com/%E6%88%AA%E5%9B%BE%202016-04-24%2021%E6%97%B651%E5%88%8611%E7%A7%92.jpg)

确定大小后:

```json
{
  "name": "WeChat",
  "description": "WeChat Bookmark",
  "version": "0.2",
  "manifest_version": 2,
  "app": {
    "urls": [
      "http://blog.zhaochunqi.com"
    ],
    "launch": {
      "web_url": "http://web.wechat.com",
      "container": "panel",
      "height": 700,
      "width": 860,
      "minHeight": 700,
      "minWidth": 860,
      "maxHeight": 700,
      "maxWidth": 860
    }
  },
  "icons": {
    "128": "wechat128.png"
  }
}
```
> 参考连接：[http://stackoverflow.com/questions/22479716/fixed-window-size-for-chrome-web-app](http://stackoverflow.com/questions/22479716/fixed-window-size-for-chrome-web-app)

来看下效果:

![http://harchiko.qiniudn.com/sfdasdfasdfasdfasdfasdf.jpg](http://harchiko.qiniudn.com/sfdasdfasdfasdfasdfasdf.jpg)

只能说： 完美 ~

美中不足的是好像那个地址栏有点多余,然而查阅资料，为了安全考虑正常方法已经无法隐藏地址栏了，链接: [http://stackoverflow.com/questions/15230137/hide-bar-address-in-popup-chrome](http://stackoverflow.com/questions/15230137/hide-bar-address-in-popup-chrome)


### 打个包

打包其实很简单，打开插件页，点击打包即可

![http://harchiko.qiniudn.com/Screen%20Shot%202016-04-24%20at%2010.01.36%20PM.png](http://harchiko.qiniudn.com/Screen%20Shot%202016-04-24%20at%2010.01.36%20PM.png)

第一次的话 private key 是不需要填写的，它会帮你生成一个，注意保存，如果不需要上传到 Chrome App Store 我估计也没啥问题。


源码在这里: [https://github.com/zhaochunqi/wechat_chrome_app](https://github.com/zhaochunqi/wechat_chrome_app)
