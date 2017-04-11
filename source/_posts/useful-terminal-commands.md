title: 常用的终端命令
date: 2015-11-26 4:20 AM
categories: 
tags: [terminal, linux,]
---

介绍一些常用的Linux命令，这些命令能够在终端下极大的方便我们的操作。

<!--more-->

![](http://harchiko.qiniudn.com/S27YSIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIHAlIPhe3WkMAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIlAcG39LatBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAhcCQi+V3caQ4AAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIBASUDwLb1tKwECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECVwKC79WdxhAgQ.png)

```bash
sudo !!
```

使用sudo权限执行上一条命令。

```bash
python -m SimpleHTTPServer
```

Serve current directory tree at http://$HOSTNAME:8000/

```bash
^foo^bar
```

替换字符后重新执行上面的命令

```bash
ctrl-x e
```
![](http://harchiko.qiniudn.com/show-ctrl-x-e.gif)
迅速的在终端激活文本编辑器用来写命令。 具体用法是按住Ctrl+x 然后再按 e，系统会自动调用默认的编辑器。

```bash
<space>command
```

在命令之前添加空格，命令不会记录在bash记录中。

```bash
curl ifconfig.me
```

获取你的外网IP

```bash
ctrl-l
```

清除终端的信息。

```bash
(cd /tmp && ls)
```

进入一个目录，执行命令后回到原目录。

```bash
cat /etc/issue
```

查看安装的Linux发行版