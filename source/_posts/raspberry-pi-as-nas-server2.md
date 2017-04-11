title: 从零开始将树莓派打造成家用下载机【二】 BT下载篇
date: 2016-04-22 10:48:22 PM
categories: "raspberry"
tags: [linux,pi]
---
本篇介绍一下如何使用树莓派进行下载。
<!--more-->

![http://harchiko.qiniudn.com/Pi_3_Model_B.png](http://harchiko.qiniudn.com/Pi_3_Model_B.png)

## 准备工作 换源

出于速度考虑，将系统默认的源修改为 中科大源

1. 备份 `cp /etc/apt/sources.list /etc/apt/sources.list.bak`
2. 修改文本 `/etc/apt/sources.list` :
    ```
    deb http://mirrors.ustc.edu.cn/raspbian/raspbian/ jessie main contrib non-free rpi
    ```
3. 更新 `sudo apt-get update`

## 树莓派上安装 Deluge 客户端

命令行输入如下命令安装

```bash
sudo apt-get install deluged

sudo apt-get install deluge-console
```

启动deluge,并杀掉进程（创建配置文件)

```bash
deluged

sudo pkill deluged
```

备份配置文件
```bash
cp ~/.config/deluge/auth ~/.config/deluge/auth.old

vi ~/.config/deluge/auth
```

按如下格式添加配置:

```
user:password:level
```

上面的用户名、密码、level级别（具体这个选项我也不知道什么等级对应什么功能,10全功能），方便起见使用:

```
pi:raspberry:10
```

保存之后重新启动

```bash
deluged

deluge-console
```
进入交互式界面

![http://harchiko.qiniudn.com/376x233x2013-03-24_163942.jpg.pagespeed.gp+jp+jw+pj+js+rj+rp+rw+ri+cp+md.ic.68wn4b8TgH.jpg](http://harchiko.qiniudn.com/376x233x2013-03-24_163942.jpg.pagespeed.gp+jp+jw+pj+js+rj+rp+rw+ri+cp+md.ic.68wn4b8TgH.jpg)

```bash
config -s allow_remote True

config allow_remote

exit
```

以上命令开启了远程链接

现在，重启deluge使配置生效

```
sudo pkill deluged

deluged
```

到这里，实际上树莓派上的工作已经做完了，现在进行电脑上的操作

## 电脑安装 Deluge 客户端

1. 下载Deluge: [http://dev.deluge-torrent.org/wiki/Download](http://dev.deluge-torrent.org/wiki/Download)

2. 安装好之后，打开Preferences->Interface，取消勾选Classic Mode下的 Enable。

    ![http://harchiko.qiniudn.com/537x271x2013-03-24_173041.jpg.pagespeed.gp+jp+jw+pj+js+rj+rp+rw+ri+cp+md.ic.HQH6hFYrAP.jpg](http://harchiko.qiniudn.com/537x271x2013-03-24_173041.jpg.pagespeed.gp+jp+jw+pj+js+rj+rp+rw+ri+cp+md.ic.HQH6hFYrAP.jpg)
3. 重启之后添加 Host（还记得第一天绑定的域名嘛,也可以填写ip),port,Username,Password.

    如下图: 

    ![http://harchiko.qiniudn.com/358x183x2013-03-24_164526.jpg.pagespeed.gp+jp+jw+pj+js+rj+rp+rw+ri+cp+md.ic.B59WSsSlJk.jpg](http://harchiko.qiniudn.com/358x183x2013-03-24_164526.jpg.pagespeed.gp+jp+jw+pj+js+rj+rp+rw+ri+cp+md.ic.B59WSsSlJk.jpg)

    ![http://harchiko.qiniudn.com/401x359x2013-03-24_173602.jpg.pagespeed.gp+jp+jw+pj+js+rj+rp+rw+ri+cp+md.ic.K7h7np4726.jpg](http://harchiko.qiniudn.com/401x359x2013-03-24_173602.jpg.pagespeed.gp+jp+jw+pj+js+rj+rp+rw+ri+cp+md.ic.K7h7np4726.jpg)

    点击连接进入主界面:
    ![http://harchiko.qiniudn.com/650x235x2013-03-24_173917.jpg.pagespeed.gp+jp+jw+pj+js+rj+rp+rw+ri+cp+md.ic.VMsLC5uovk.jpg](http://harchiko.qiniudn.com/650x235x2013-03-24_173917.jpg.pagespeed.gp+jp+jw+pj+js+rj+rp+rw+ri+cp+md.ic.VMsLC5uovk.jpg)

### 配置

#### 下载配置：

在Downloads目录下:

```bash
mkdir /home/pi/Downloads/downloading
mkdir /home/pi/Downloads/completed
mkdir -p /home/pi/Downloads/torrents/watch
mkdir -p /home/pi/Downloads/torrents/torrent-backup
```

按照图中修改即可

![http://harchiko.qiniudn.com/Screen%20Shot%202016-04-23%20at%201.06.13%20AM.png](http://harchiko.qiniudn.com/Screen%20Shot%202016-04-23%20at%201.06.13%20AM.png)

#### 网络配置
按图示修改即可。

![http://harchiko.qiniudn.com/Screen%20Shot%202016-04-23%20at%2012.05.33%20AM.png](http://harchiko.qiniudn.com/Screen%20Shot%202016-04-23%20at%2012.05.33%20AM.png)

这下，你可以远程添加种子到树莓派下载了

![http://harchiko.qiniudn.com/Screen%20Shot%202016-0aa4-23%20at%2012.08.42%20AM.jpg](http://harchiko.qiniudn.com/Screen%20Shot%202016-0aa4-23%20at%2012.08.42%20AM.jpg)

## One More thing

设置定时任务，免得影响正常上网:

```bash
crontab -e
```
在后面添加内容
```
# 周一到周五
0 1 * * 1,2,3,4,5 deluged
0 18 * * 1,2,3,4,5 pkill deluged 

# 周六周日
0 2 * * 0,6 deluged 
0 8 * * 0,6 sudo pkill deluged 
```
周一到周五，每天凌晨1点开启，晚上6点结束；
周六周日，凌晨两点开启，早上8点结束；

至此，下载已经告一段落。

参考链接: 1. [http://www.howtogeek.com/142044/how-to-turn-a-raspberry-pi-into-an-always-on-bittorrent-box/](http://www.howtogeek.com/142044/how-to-turn-a-raspberry-pi-into-an-always-on-bittorrent-box/)