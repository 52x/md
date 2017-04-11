title: 我用的很爽的工作流
date: Aug 31, 2016 12:34 PM
categories: ""
tags: [tools,linux]
---
分享一下个人用的很爽的一些工作流，包含 翻墙、下载、同步等。

<!--more-->

图片镇楼:
![http://harchiko.qiniudn.com/54686763_p0.jpg](http://harchiko.qiniudn.com/54686763_p0.jpg)

## 翻墙&域名解析

搬瓦工 最低价格的vps，足够用，还可以迁移服务器。

这里，我把搬瓦工的 ip 地址配置在我子域名的 A 记录上（需要自己域名，可使用 DNSPOD 解析），主要有几个好处:

* 登录 vps 可以使用域名地址 如: `ssh root@linux.zhaochunqi.com`
* 作为翻墙服务时，由于每个地区节点的网速有所不同，有时候需要迁移，迁移过后，只需要修改相应的 A 记录，其他地方可以不用动。

## SSH 无密码登录

作为一个懒人，甚至我连 `ssh root@linux.zhaochunqi.com` 都觉得很累，很麻烦,更不要说每次还要输入密码了。

分几步，有些步骤可能之前配置 `git`之类的你有做过，可跳过

1. 生成 ssh-key . 查看是否已经存在 `cat ~/.ssh/id_rsa.pub`,不存在可以使用 `ssh-keygen -f ~/.ssh/id_rsa -t rsa -N ''`来生成。
2. 安装 ssh-copy-id `brew install ssh-copy-id`
3. 使用 ssh-copy-id命令 `ssh-copy-id -i ~/.ssh/id_rsa.pub [用你自己的地址替换，如 root@linux.zhaochunqi.com]`，输入密码，这样你的vps可信任的key中就有你了，下次登录 `ssh root@linux.zhaochunqi.com`，不用输密码了。
4. alias, 把alias命令添加到 `.bashr`或`.zshrc`中  alias vps="ssh root@linux.zhaochunqi.com"。

我们来登录下vps，terminal中输入 `vps`即可:
![http://harchiko.qiniudn.com/vps-my-work-flow.gif](http://harchiko.qiniudn.com/vps-my-work-flow.gif)

## 下载国外资源

国内有些资源即便翻着墙也很难下载，反观国外，基本分分钟就能下载好，如何利用下这个呢？

使用 `BitTorrent Sync` ,免费版就足够用，创建一个文件夹用来同步，如 `vps`,本地也创建一个同步文件夹，因为 VPS 空间有限，注意把缓存已删除文件取消勾选。

举个例子：

你下载 `Android Studio`，翻墙下载速度很慢，
1. 打开Terminal，
2. 输入vps，
3. 进入你的vps同步文件夹，
4. wget 链接地址，2s下完，
5. 等待 BTsync 同步到你本机 

你可能会有疑问（并没有），意义在哪里呢？
* 速度快
* 你同步可以断点续传啊！！
* 稳定

顺便说一下下载 `youtube`视频：

VPS 上安装 `youtube-dl`,使用 `youtube-dl 链接地址`就可以下载了，需要提醒的是，youtube-dl 支持的网站不知道多到哪里去了！ 自己去`google`一下吧。
