title: 如何在 Ubuntu 中安装 hexo
date: 2014-12-22 02:18:16
tags: linux
---

## 安装 nvm

cURL:
```bash
	$ curl https://raw.github.com/creationix/nvm/master/install.sh | sh
```

<!--more-->
Wget:
```bash
	$ wget -qO- https://raw.github.com/creationix/nvm/master/install.sh | sh
```

安装后重启 Terminal 。

> 因为使用的是zsh，所以最好在.zshrc 文件中加入这句`source ~/.nvm/nvm.sh` 

## 安装 NodeJS

运行：

```bash
$ nvm install 0.10 && nvm use 0.10
```
	
### 安装 Hexo
```bash
	$ npm install -g hexo
```
> Hexo 的使用以及注意事项见[链接](http://hexo.io/docs/)