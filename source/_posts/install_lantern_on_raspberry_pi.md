title: 使用 lantern 畅游互联网
date: 2016-05-30 2:51 AM
categories: "raspberry"
tags: [raspberry,]
---

就在昨天，一直使用的 shadowsocks 的 VPS 到期了，临时试用 lantern 来应急，结果发现 lantern 竟然比我一直在用的 shadowsocks 速度还快。

<!--more-->

![http://harchiko.qiniudn.com/Screen%20Shot%202016-05-30%20at%202.54.45%20AM.png](http://harchiko.qiniudn.com/Screen%20Shot%202016-05-30%20at%202.54.45%20AM.png)

## lantern

> Lantern, a new software programme which allows internet users to circumvent government-imposed censorship, is seeing rapid growth in China as more people are using it to bypass the Great Firewall of China to access websites like Facebook, YouTube and Twitter.

简言之，专为中国不能上网的宝宝设计！就是为我设计的！

## 下载

下载地址: [https://getlantern.org/](https://getlantern.org/)

呃...网站好像访问不了？

哦，那就对了，贴心的我给你准备了下载链接！不用谢！

Mac客户端: [https://mega.nz/#!gBthnBCI!7mhorsXKWbZgsJvp4BYoEn6ulEj8ZSke1pFczKW4Tf0](https://mega.nz/#!gBthnBCI!7mhorsXKWbZgsJvp4BYoEn6ulEj8ZSke1pFczKW4Tf0)

Android客户端: [https://mega.nz/#!FM0XCR4Q!nvNyQaZU3o76eAg_OzJMgzimZf8XokjCkglbAdOcZ9g](https://mega.nz/#!FM0XCR4Q!nvNyQaZU3o76eAg_OzJMgzimZf8XokjCkglbAdOcZ9g)

Windows客户端: 呃，我手头没有windows设备，暂时没下下来。

除了 ios ， 各种客户端都有。

## For Raspberry Pi

如果只是简简单单的下载使用，也就没有这篇文章了。下面介绍下如何在 pi 上安装，并用 pi 作为翻墙代理。

直接使用网上别人打包好的树莓派版本的包: 下载地址: [https://mega.nz/#!IZdE2Yxa!HU7PBAuRQdUomS5MIt0VvFgB2K3Upv-bdrd4o0UHY98](https://mega.nz/#!IZdE2Yxa!HU7PBAuRQdUomS5MIt0VvFgB2K3Upv-bdrd4o0UHY98) 下载 lantern*.deb 放到你的树莓派中。

```bash
dpkg -i lantern_2.2.4-1_armhf.deb
sudo systemctl start lantern
sudo systemctl enable lantern
```

设定完毕后运行

```bash
curl -x https://127.0.0.1:8787 https://www.google.com/
```

查看是否正常设置

## Chrome 设定

lantern 就是有一点不好，如果上的是国内网站的话，他跑的比谁都慢！

设定下

1. chrome 安装 SwitchySharp
2. 设定代理地址，刚刚你树莓派的ip，端口8787
3. 在 AutoProxy 那里设定 online rules: [https://raw.githubusercontent.com/gfwlist/gfwlist/master/gfwlist.txt](https://raw.githubusercontent.com/gfwlist/gfwlist/master/gfwlist.txt)
4. 设定，当符合 online rules 时，使用代理。
5. 保存，并切换到 Autoproxy

好了，保存

## 题外

lantern 在本地运行在 8787 端口，这意味着你可以用它来代理你的 Dropbox ， 使用 Proxifier 来代理你的一切流量。

好了，暂且就到这里，去睡觉了！
