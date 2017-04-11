---
title: Debian 更新Hosts文件的办法
tags: [vps,gfw]
date: 2017-03-24 16:17:45
updated: 2017-03-24 16:17:45
categories: [Gfw]
keywords:
description:
---

新弄了一台华为云的服务器试用

先更新个hosts 记录下来。



切换为root用户（懒癌已经没治了，，哈哈）

先备份现在的host

```
cp hosts hosts.bak
```

从网上更新一个hosts内容，直接附加到hosts后面

```
 curl  https://raw.githubusercontent.com/racaljk/hosts/master/hosts >> hosts
```

然后重启网卡

```
/etc/init.d/networking restart
```

这样就可以了。ping一下需要ping的地址就知道了。



再次更换的时候，rm掉现在的hosts，将hosts.bak恢复 然后再来一遍就好。