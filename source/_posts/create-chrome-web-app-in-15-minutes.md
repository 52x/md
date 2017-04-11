title: 15分钟开发一个Chrome App【一】
date: 2016-04-24 12:51 PM
categories: "develper"
tags: [chrome,developer]
---

> 本文介绍了如何在很短的时间内开发一个 Chrome APP.

<!--more-->

![http://harchiko.qiniudn.com/Screen%20Shot%202016-04-24%20at%2012.53.40%20PM.png](http://harchiko.qiniudn.com/Screen%20Shot%202016-04-24%20at%2012.53.40%20PM.png)

## 准备图标

网上搜索一个图标，这里使用微信的图标

![http://harchiko.qiniudn.com/wechat128.png](http://harchiko.qiniudn.com/wechat128.png)

## 创建manifest.json 文件

创建一个文件夹，在文件夹中添加 `manifiest.json`。

```json
{
	"name": "WeChat",
	"description": "Just A Wechat Web App",
	"version": "0.1",
	"manifest_version": 2,
	"app": {
		"urls": ["http://blog.zhaochunqi.com"],
		"launch": {
			"web_url": "http://web.wechat.com"
		}
	},
	"icons": {
		"128": "wechat128.png"
	}
}
```

> **注意：** manifest_version 需要是 2。

### 测试

打开Chrome的扩展界面，勾选 `Developer Mode`, 点击 `Load unpacked extention`,找到文件夹：

![http://harchiko.qiniudn.com/Screen%20Shot%202016-04-24%20at%201.03.21%20PM.png](http://harchiko.qiniudn.com/Screen%20Shot%202016-04-24%20at%201.03.21%20PM.png)

打开Chrome App Launcher，你的应用已在其中:

![http://harchiko.qiniudn.com/Screen%20Shot%202016-04-24%20at%201.03.57%20PM.png](http://harchiko.qiniudn.com/Screen%20Shot%202016-04-24%20at%201.03.57%20PM.png)

点击后跳转到 微信界面。

Easy, hah?

源码在这里: [https://github.com/zhaochunqi/wechat_chrome_app](https://github.com/zhaochunqi/wechat_chrome_app)
