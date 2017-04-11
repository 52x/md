title: Thinkpad x240 Linux无线网卡驱动
date: 2015-06-26 23:46:23
categories: Life
tags: [Linux, C++]
---
刚刚换了硬盘，新装了Ubuntu 14.10，但是却遇到了无线网卡驱动问题。Thinkpad X240的无线网卡是rtl8192ee，这是realtek比较新的网卡，Linux没有自带他的驱动。
<!--more-->
解决方案可以自行编译驱动，[rtlwifi_new](https://github.com/lwfinger/rtlwifi_new)。将源代码下载到Linux系统，安装linux-source来编译,`sudo apt-get install linux-source`,之后通过终端运行`sudo make`和`sudo make install`，驱动便安装成功了。

在Linux中会遇到很多意想不到的小问题，其中很多都要手动编译代码解决。例如，Sublime Text 3不能支持中文输入法（搜狗拼音），下载相应的CPP补丁，编译成.so文件，在Sublime的配置文件中替换相应的.so即可。玩Linux就是要不断折腾。
