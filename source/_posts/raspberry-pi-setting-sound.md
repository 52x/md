title: 树莓派声音设置
tags: [raspberry pi, alsa]
---

在树莓派上安装alsa-utils包，包含很多alsa相关的工具。

### 测试
可以通过speak-test和aplay命令来测试声音   
speak-test: 播放白噪声   
aplay: 用来播放wav文件, 如果安装了scratch包的话，可以在/usr/share/scratch/Media/Sounds下找到wav文件

### amixer
amixer命令可以显示出配置的信息

    pi@raspberrypi:~ $ amixer
	Simple mixer control 'Master',0
	  Capabilities: pvolume pswitch pswitch-joined
	  Playback channels: Front Left - Front Right
	  Limits: Playback 0 - 65536
	  Mono:
	  Front Left: Playback 43233 [66%] [on]
	  Front Right: Playback 43233 [66%] [on]
	Simple mixer control 'Capture',0
	  Capabilities: cvolume cswitch cswitch-joined
	  Capture channels: Front Left - Front Right
	  Limits: Capture 0 - 65536
	  Front Left: Capture 65536 [100%] [on]
	  Front Right: Capture 65536 [100%] [on]

获取可以控制的配置列表

	pi@raspberrypi:~ $ amixer controls
	numid=4,iface=MIXER,name='Master Playback Switch'
	numid=3,iface=MIXER,name='Master Playback Volume'
	numid=2,iface=MIXER,name='Capture Switch'
	numid=1,iface=MIXER,name='Capture Volume'

获取一个配置的当前信息

	pi@raspberrypi:~ $ amixer cget numid=3
	numid=3,iface=MIXER,name='Master Playback Volume'
	  ; type=INTEGER,access=rw------,values=2,min=0,max=65536,step=1
	  : values=43233,43233

改变配置

	pi@raspberrypi:~ $ amixer cset numid=3 40%
	numid=3,iface=MIXER,name='Master Playback Volume'
	  ; type=INTEGER,access=rw------,values=2,min=0,max=65536,step=1
	  : values=26231,26231

### 参考
[http://blog.scphillips.com/posts/2013/01/sound-configuration-on-raspberry-pi-with-alsa/](http://blog.scphillips.com/posts/2013/01/sound-configuration-on-raspberry-pi-with-alsa/)