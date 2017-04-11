title: 通过dlan在raspberry pi上播放歌曲
---
将手机或电脑中的歌曲通过raspberry pi播放。

### 安装gstreamer
	$ sudo apt-get install libupnp-dev libgstreamer1.0-dev \
	gstreamer1.0-plugins-base gstreamer1.0-plugins-good \
	gstreamer1.0-plugins-bad gstreamer1.0-plugins-ugly \
	gstreamer1.0-alsa

### 安装gmediarender
	$ sudo apt-get install gmediarender

### 参考
[http://blog.scphillips.com/posts/2013/01/sound-configuration-on-raspberry-pi-with-alsa/](http://blog.scphillips.com/posts/2013/01/sound-configuration-on-raspberry-pi-with-alsa/)   
[http://blog.scphillips.com/posts/2013/07/playing-music-on-a-raspberry-pi-using-upnp-and-dlna-revisited/](http://blog.scphillips.com/posts/2013/07/playing-music-on-a-raspberry-pi-using-upnp-and-dlna-revisited/)