---
title: OpenWRT的SDK编译介绍
date: 2016-05-11 15:26:25
tags:
 - openwrt
 - sdk
categories:
 - openwrt
---

### SDK位置 ###

openwrt的项目代码放在GitHub上面，check this: [https://github.com/openwrt](https://github.com/openwrt "https://github.com/openwrt")。

openwrt的SDK位置在此：[https://github.com/openwrt/openwrt](https://github.com/openwrt/openwrt "https://github.com/openwrt/openwrt")。

在`Branches`那里选择需要下载的SDK版本：`attitude_adjustment`、`barrier_breaker` or `chaos_calmer`。

### SDK编译 ###

下完后执行`./scripts/feeds update -a`获取`feeds.conf`/`feeds.conf.default`里面的包。

并且执行`./scripts/feeds install -a`来安装。

然后`make menuconfig`选择型号。

最后`make V=s -j 4`进行编译。

### make menuconfig出错 ###

出错基本上就是`linux`系统有些依赖没有安装，注意查看执行`make menuconfig`之后，顶部的checking ...... fail。

checking哪个软件包fail就安装哪个软件包。

### SDK编译出错 ###

因为第一次编译SDK时需要从网上下载一些软件包到`dl`目录里（下载的软件包可能需要科学上网方式），所以SDK编译不通过基本上就是无法下载对应的软件包。

解决方法是`cd dl`目录，`ls *.dl`列出下载出错的软件包，手动下载对应的软件包，并放至此`dl`目录。

`*.dl`和`*.md5sum`是下载过程的一些中间产物，手动下载了软件包之后，记得删掉对应的`*.dl`和`*.md5sum`，否则重新编译的时候，`dl`目录还存在那些中间产物时，还会继续下载该软件包。

可以去这里下载软件包：[http://dl.openwrtdl.com/](http://dl.openwrtdl.com/ "http://dl.openwrtdl.com/")

tips:

编译报错时可能找不到出错在哪，可以这样`make V=s -j 4 > /tmp/test.log`。

这样的目的是把正常的打印输出导向`/tmp/test.log`文件，但是错误信息还是正常输出到`STDOUT`，这样就很方便看到哪个地方出错了。

### root编译 ###

`barrier_breaker`版本默认不允许root用户编译，`vi include/prereq-build.mk`如下即可：

```
define Require/non-root
        # [ "$$(shell whoami)" != "root" ]
endef
```
