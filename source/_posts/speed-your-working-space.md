title: 加速开发环境的搭建
date: 2015-02-15 07:29:41
tags: [mac,python,linux,nodejs]
---

在国内，一些服务或者被墙，或者访问速度过慢，对开发者而言，这是最痛苦的，下面，介绍几种减轻痛苦的良方以飨读者。

<!--more-->

# Linux 篇

linux 下更改源：[http://mirrors.ustc.edu.cn/](http://mirrors.ustc.edu.cn/ "中科大源")，选择自己的Linux版本，然后选择对应的源配置，具体使用见[https://lug.ustc.edu.cn/wiki/mirrors/help](https://lug.ustc.edu.cn/wiki/mirrors/help "配置帮助")

# HomeBrew 篇

github 时不时的网络中断，brew update经常不成功，更改git地址为中科大git源的地址：

```bash
cd /usr/local
git remote set-url origin http://mirrors.ustc.edu.cn/homebrew.git
brew update
```

# Node 篇

## NVM

clone nvm到本地，假设目录位于 `～`
```bash
$ git clone https://github.com/creationix/nvm.git
```

更改nvm镜像地址，在.bashrc中添加
```
# nvm
export NVM_NODEJS_ORG_MIRROR=http://dist.u.qiniudn.com
source ~/nvm/nvm.sh
```

`source .bashrc` 后运行 nvm

## NPM

通过`--registry`更改参数：
`$ npm --registry=https://registry.npm.taobao.org install koa`



# Pypi 篇

通过 `-i` 更改参数:

如:`$ pip install web.py -i http://pypi.mirrors.ustc.edu.cn/simple`

**注意后面有`/simple`**

要配制成默认的话，需要创建或修改配置文件（linux 的文件在`~/.pip/pip.conf`，windows在`%HOMEPATH%\pip\pip.ini`），修改内容为：

```
[global]

index-url = http://pypi.mirrors.ustc.edu.cn/simple
```

> 注意，由于更改的都是非官方源，但由于选择的这些企业或学校都是属于比较靠谱的，基本不存在突然不可用的可能，如果有问题，改回来即可。