title: 在树莓派中安装 docker 
date: 2016-05-29 7:45 PM
categories: "raspberry"
tags: [docker,]
---

一直以为树莓派中没有办法安装 docker , 今天在一位博主的博客中看到有在树莓派中使用。

<!--more-->

```bash
# 安装docker
wget https://github.com/vimagick/rpi-bin/\
raw/master/deb/docker_1.10.3-1_armhf.deb  
dpkg -i docker_1.10.3-1_armhf.deb
```

```bash

# 安装docker-compose
wget https://github.com/vimagick/rpi-bin/\
raw/master/deb/docker-compose_1.7.1-1_armhf.deb
dpkg -i docker-compose_1.7.1-1_armhf.deb

```


参考连接: [https://blog.easypi.info/deploying-shadowsocks-with-docker/](https://blog.easypi.info/deploying-shadowsocks-with-docker/)
