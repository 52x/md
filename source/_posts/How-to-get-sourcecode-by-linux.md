---
title: Linux下获取软件源码的几种方法
date: 2016-07-15 12:59:56
tags: Linux
categories: Linux
---

## 直接在源码网站下载

- [github](https://github.com/)
- [gnu软件列表](http://ftp.gnu.org/)
- Linux各种发行版的在线软件列表，列如[ArchLinux在线软件包](https://www.archlinux.org/packages/?q=coreutils)

## 在Linux发行版下通过包管理器下载

因为不同的发行版有不同的软件包管理机制，所以在此我只简单介绍ArchLinux和Ubuntu的源码下载方法，其他的发行版请自行参考网上相关文档。

<!--more-->

### **ArchLinux下通过abs(Arch Build System)下载**

首先，通过pacman安装abs工具

`sudo pacman -S base-devel abs`

然后，下载abs树

`sudo abs`

接着，下载特定的软件包

```bash
sudo abs [package_name]
列如find包：
pacman -Qo  $(which find)
结果显示："/usr/bin/find is owned by findutils 4.4.2-3"
cp -r /var/abs/core/findutils /home/your_name/findutils
cd /home/your_name/findutils
makepkg -od
```
makepkg简单用法

- `makepkg -od` 获取软件源码，不进行构建
- `makepkg -s` 自动处理软件相关依赖
- `makepkg -e` 构建本地软件包

如果你想要手动安装构建的软件包

`pacman -U name-of-package.xz`

### **Ubuntu下通过apt系列工具下载**

Ubuntu下依然以find命令作为列子

```bash
dpkg -s $(which find)
结果显示：findutils: /usr/bin/find
sudo apt-get source findutils
cd /usr/src/findutils-XXX #XXX表示版本号  
sudo tar zxvf findutils-XXX.tar.gz  
```

## 参考

- [ArchLinux Wiki](https://wiki.archlinux.org/)
- [GNU FTP](http://ftp.gnu.org/)
- [如何查看linux命令源代码](http://blog.csdn.net/earbao/article/details/17955815)
