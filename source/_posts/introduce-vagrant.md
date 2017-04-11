title: Parallels 下 Vagrant 使用指南
date: 2015-02-05 01:59:07
categories: soft
tags: [mac,vagrant,parallels]

---

环境：OSX yosemite/Parallels Desktop 10


## Vagrant 介绍

Vagrant 是一个基于 Ruby 的工具，用于创建和部署虚拟化开发环境。它 使用 Oracle 的开源 VirtualBox 虚拟化系统，使用 Chef 创建自动化虚拟环境。

<!--more-->
## 安装 Vagrant

从这里 [http://www.vagrantup.com/downloads](http://www.vagrantup.com/downloads "vagrant 下载") 下载对应版本的 vagrant 安装。

## 安装 Vagrant 插件

打开 Terminal，安装 vagrant 插件：
    
```bash
$ vagrant plugin install vagrant-parallels
```

## 使用 Vagrant

```bash
$ mkdir new_vagrant_project
$ cd new_vagrant_project
$ vagrant init parallels/centos-6.5
$ vagrant up --provider=parallels
```

这样就成功的安装了 centos-6.5
 
## 修改默认的 Provider (可选)
```bash
$ export VAGRANT_DEFAULT_PROVIDER=parallels
```
可添加到.bashrc 或者 .zshrc(如果你使用zsh)

## 使用SSH连接

进入 vagrant 所在目录：
```bash
$ vagrant ssh
```
## box 选择

如果你想要其他的发行版：如ubuntu等，在这个网页中查找是否有你所需要的：
[https://vagrantcloud.com/boxes/search?provider=parallels](https://vagrantcloud.com/boxes/search?provider=parallels "box")

## vagrant manager（可选/推荐）

Vagrant Manager 是在通知栏的可以管理Vagrant的工具。（Mac版本/Windows版本都有）

具体表现如图：
![vagrant manager](http://harchiko.qiniudn.com/introduce-vagrant/vagrant_manager.gif)

下载见：[https://github.com/lanayotech/vagrant-manager](https://github.com/lanayotech/vagrant-manager "vagrant manager")

### Happy Coding ~ 