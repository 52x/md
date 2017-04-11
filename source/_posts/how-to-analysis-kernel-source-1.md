---
title: 分析内核源码如何入手（上）
tags:
categories:
  - Kernel
  - 内核修炼之道
date: 2014-04-17 10:40:37
---

原文：[《Linux内核修炼之道》精华分享与讨论（6）——分析内核源码如何入手？（上） ](http://blog.csdn.net/fudan_abc/article/details/5347687)

透过现象看本质，兽兽门无非就是一些人体艺术展示。同样往本质里看过去，学习内核，就是学习内核的源代码，任何内核有关的书籍都是基于内核，而又不高于内核的。

既然要学习内核源码，就要经常对内核代码进行分析，而内核代码千千万，还前仆后继的不断往里加，这就让大部分人都有种雾里看花花不见的无助感。不过不要怕，孔老夫子早就留给我们了应对之策：敏于事而慎于言，就有道而正焉，可谓好学也已。这就是说，做事要踏实才是好学生好同志，要遵循严谨的态度，去理解每一段代码的实现，多问多想多记。如果抱着走马观花，得过且过的态度，结果极有可能就是一边看一边丢，没有多大的收获
<!--more-->
下面就以USB子系统的实现分析为标本看看分析内核源码应该如何入手。

## 分析README

内核中USB子系统的代码位于目录drivers/usb，进入到该目录，执行命令ls，结果显示如下：
```
atm  class  core  gadget  host  image  misc  mon  serial  storage Kconfig
Makefile  README usb-skeleton.c
```
目录drivers/usb共包含有10个子目录和4个文件，usb-skeleton.c是一个简单的USB driver的框架，感兴趣的可以去看看，目前来说，它还吸引不了我们的眼球。那么首先应该关注什么？如果迎面走来一个ppmm，你会首先看脸、脚还是其它？当然答案依据每个人的癖好会有所不同。不过这里的问题应该只有一个答案，那就是Kconfig、Makefile、README。

README里有关于这个目录下内容的一般性描述，它不是关键，只是帮助你了解。再说了，面对“read我吧read我吧”这么热情奔放的呼唤，善良的我们是不可能无动于衷的，所以先来看看里面都有些什么内容。
```
Here is a list of what each subdirectory here is, and what is contained in
them.

core/        - This is for the core USB host code, including the
            usbfs files and the hub class driver ("khubd").

host/        - This is for USB host controller drivers.  This
            includes UHCI, OHCI, EHCI, and others that might
            be used with more specialized "embedded" systems.

gadget/        - This is for USB peripheral controller drivers and
            the various gadget drivers which talk to them.


Individual USB driver directories.  A new driver should be added to the
first subdirectory in the list below that it fits into.

image/        - This is for still image drivers, like scanners or
            digital cameras.
input/        - This is for any driver that uses the input subsystem,
            like keyboard, mice, touchscreens, tablets, etc.
media/        - This is for multimedia drivers, like video cameras,
            radios, and any other drivers that talk to the v4l
            subsystem.
net/        - This is for network drivers.
serial/        - This is for USB to serial drivers.
storage/    - This is for USB mass-storage drivers.
class/        - This is for all USB device drivers that do not fit
            into any of the above categories, and work for a range
            of USB Class specified devices.
misc/        - This is for all USB device drivers that do not fit
            into any of the above categories.
```
这个README文件描述了前边使用ls命令列出的那10个文件夹的用途。那么什么是USB Core？Linux内核开发者们，专门写了一些代码，负责实现一些核心的功能，为别的设备驱动程序提供服务，比如申请内存，比如实现一些所有的设备都会需要的公共的函数，并美其名曰USB Core。

时代总在发展，当年胖杨贵妃照样迷死唐明皇，而如今人们欣赏的则是林志玲这样的魔鬼身材。同样，早期的Linux内核，其结构并不是如今天这般有层次感，远不像今天这般错落有致，那时候drivers/usb/这个目录下边放了很多很多文件，USB Core与其他各种设备的驱动程序的代码都堆砌在这里，后来，怎奈世间万千的变幻，总爱把有情的人分两端。于是在drivers/usb/目录下面出来了一个core目录，就专门放一些核心的代码，比如初始化整个USB系统，初始化Root Hub，初始化主机控制器的代码，再后来甚至把主机控制器相关的代码也单独建了一个目录，叫host目录，这是因为USB主机控制器随着时代的发展，也开始有了好几种，不再像刚开始那样只有一种，所以呢，设计者们把一些主机控制器公共的代码仍然留在core目录下，而一些各主机控制器单独的代码则移到host目录下面让负责各种主机控制器的人去维护。

那么USB gadget那？gadget白了说就是配件的意思，主要就是一些内部运行Linux的嵌入式设备，比如PDA，设备本身有USB设备控制器（USB Device Controller），可以将PC，也就是我们的主机作为master端，将这样的设备作为slave端和主机通过USB进行通信。从主机的观点来看，主机系统的USB驱动程序控制插入其中的USB设备，而USB gadget的驱动程序控制外围设备如何作为一个USB设备和主机通信。比如，我们的嵌入式板子上支持SD卡，如果我们希望在将板子通过USB连接到PC之后，这个SD卡被模拟成U盘，那么就要通过USB gadget架构的驱动。

剩下的几个目录分门别类的放了各种USB设备的驱动，比如U盘的驱动在storage目录下，触摸屏和USB键盘鼠标的驱动在input目录下，等等。

我们响应了README的热情呼唤，它便给予了我们想要的，通过它我们了解了USB目录里的那些文件夹都有着什么样的角色。到现在为止，就只剩下内核的地图——Kconfig与Makefile两个文件了。有地图在手，对于在内核中游荡的我们来说，是件很愉悦的事情，不过，因为我们的目的是研究内核对USB子系统的实现，而不是特定设备或host controller的驱动，所以这里的定位很明显，USB Core就是我们需要关注的对象，那么接下来就是要对core目录中的内容进行定位了。

## 分析Kconfig和Makefile

进入到drivers/usb/core目录，执行命令ls，结果显示如下：
```
Kconfig  Makefile  buffer.c  config.c  devices.c  devio.c  driver.c
endpoint.c  file.c  generic.c  hcd-pci.c  hcd.c  hcd.h  hub.c  hub.h
inode.c  message.c  notify.c  otg_whitelist.h  quirks.c  sysfs.c  urb.c
usb.c  usb.h
```
然后执行wc命令，如下所示。
```bash
# wc –l ./*
   148 buffer.c
   607 config.c
   706 devices.c
  1677 devio.c
  1569 driver.c
   357 endpoint.c
   248 file.c
   238 generic.c
  1759 hcd.c
   458 hcd.h
   433 hcd-pci.c
  3046 hub.c
   195 hub.h
   758 inode.c
   144 Kconfig
    21 Makefile
  1732 message.c
    68 notify.c
   112 otg_whitelist.h
   161 quirks.c
   710 sysfs.c
   589 urb.c
   984 usb.c
   160 usb.h
 16880 total
 ```
drivers/usb/core目录共包括24个文件，16880行代码。core不愧是core，为大家默默的做这么多事。不过这么多文件里不一定都是我们所需要关注的，先拿咱们的地图来看看接下来该怎么走。先看看Kconfig文件，可以看到下面的选项。
```
config USB_DEVICEFS
        bool "USB device filesystem"
        depends on USB
        ---help---
          If you say Y here (and to "/proc file system support" in the "File
          systems" section, above), you will get a file /proc/bus/usb/devices
          which lists the devices currently connected to your USB bus or
          busses, and for every connected device a file named
          "/proc/bus/usb/xxx/yyy", where xxx is the bus number and yyy the
          device number; the latter files can be used by user space programs
          to talk directly to the device. These files are "virtual", meaning
          they are generated on the fly and not stored on the hard drive.

          You may need to mount the usbfs file system to see the files, use
          mount -t usbfs none /proc/bus/usb

          For the format of the various /proc/bus/usb/ files, please read
          <file:Documentation/usb/proc_usb_info.txt>.

          Usbfs files can't handle Access Control Lists (ACL), which are the
          default way to grant access to USB devices for untrusted users of a
          desktop system. The usbfs functionality is replaced by real
          device-nodes managed by udev. These nodes live in /dev/bus/usb and
          are used by libusb.
```
选项USB_DEVICEFS与usbfs文件系统有关。usbfs文件系统挂载在/proc/bus/usb目录，显示了当前连接的所有USB设备及总线的各种信息，每个连接的USB设备在其中都会有一个对应的文件进行描述。比如文件/proc/bus/usb/xxx/yyy，xxx表示总线的序号，yyy表示设备所在总线的地址。不过不能够依赖它们来稳定地访问设备，因为同一设备两次连接对应的描述文件可能会不同，比如，第一次连接一个设备时，它可能是002/027，一段时间后再次连接，它可能就已经改变为002/048。

就好比好不容易你暗恋的mm今天见你的时候对你抛了个媚眼，你心花怒放，赶快去买了100块彩票庆祝，到第二天再见到她的时候，她对你说你是谁啊，你悲痛欲绝的刮开那100块彩票，上面清一色的谢谢你。

因为usbfs文件系统并不属于USB子系统实现的核心部分，与之相关的代码我们可以不必关注。
```
74 config USB_SUSPEND
75       bool "USB selective suspend/resume and wakeup (EXPERIMENTAL)"
76       depends on USB && PM && EXPERIMENTAL
77       help
78         If you say Y here, you can use driver calls or the sysfs
79         "power/state" file to suspend or resume individual USB
80         peripherals.
81
82         Also, USB "remote wakeup" signaling is supported, whereby some
83         USB devices (like keyboards and network adapters) can wake up
84         their parent hub.  That wakeup cascades up the USB tree, and
85         could wake the system from states like suspend-to-RAM.
86
87         If you are unsure about this, say N here.
```
这一项是有关USB设备的挂起和恢复。开发USB的人都是节电节能的好孩子，所以协议里就规定了，所有的设备都必须支持挂起状态，就是说为了达到节电的目的，当设备在指定的时间内，如果没有发生总线传输，就要进入挂起状态。当它收到一个non-idle的信号时，就会被唤醒。节约用电从USB做起。不过这个与主题也没太大关系，相关代码也可以不用关注了。

剩下的还有几项，不过似乎与咱们关系也不大，还是去看看Makefile。
```make
usbcore-objs    := usb.o hub.o hcd.o urb.o message.o driver.o /
                        config.o file.o buffer.o sysfs.o endpoint.o /
                        devio.o notify.o generic.o quirks.o

ifeq ($(CONFIG_PCI),y)
         usbcore-objs    += hcd-pci.o
 endif

 ifeq ($(CONFIG_USB_DEVICEFS),y)
         usbcore-objs    += inode.o devices.o
 endif

 obj-$(CONFIG_USB)       += usbcore.o

 ifeq ($(CONFIG_USB_DEBUG),y)
 EXTRA_CFLAGS += -DDEBUG
 endif
```
Makefile可比Kconfig简略多了，所以看起来也更亲切点，咱们总是拿的money越多越好，看的代码越少越好。这里之所以会出现CONFIG_PCI，是因为通常USB的Root Hub包含在一个PCI设备中。hcd-pci和hcd顾名而思义就知道是说主机控制器的，它们实现了主机控制器公共部分，按协议里的说法它们就是HCDI（HCD的公共接口），host目录下则实现了各种不同的主机控制器。

CONFIG_USB_DEVICEFS前面的Kconfig文件里也见到了，关于usbfs的，与咱们的主题无关，inode.c和devices.c两个文件也可以不用管了。

那么我们可以得出结论，为了理解内核对USB子系统的实现，我们需要研究buffer.c、config.c、driver.c、endpoint.c、file.c、generic.c、hcd.c  hcd.h、hub.c、message.c、notify.c、otg_whitelist.h、quirks.c、sysfs.c、urb.c 和usb.c文件。这么看来，好像大都需要关注的样子，没有减轻多少压力，不过这里本身就是USB Core部分，是要做很多的事为咱们分忧的，所以多点也是可以理解的。
