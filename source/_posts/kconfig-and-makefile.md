---
title: Kernel地图：Kconfig与Makefile
tags:
categories:
  - Kernel
  - 内核修炼之道
date: 2014-04-17 10:18:09
---

原文：[《Linux内核修炼之道》精华分享与讨论（5）——Kernel地图：Kconfig与Makefile ](http://blog.csdn.net/fudan_abc/article/details/5340408)

Kconfig和Makefile就是Linux Kernel迷宫里的地图。地图引导我们去认识一个城市，而Kconfig和Makefile则可以让我们了解一个Kernel目录下面的结构。我们每次浏览kernel寻找属于自己的那一段代码时，都应该首先看看目录下的这两个文件。
<!--more-->

## 利用Kconfig和Makefile寻找目标代码

就像利用地图寻找目的地一样，我们需要利用Kconfig和Makefile来寻找所要研究的目标代码。

比如我们打算研究U盘驱动的实现，因为U盘是一种storage设备，所以我们应该先进入到drivers/usb/storage/目录。但是该目录下的文件很多，那么究竟哪些文件才是我们需要关注的？这时就有必要先去阅读Kconfig和Makefile文件。

对于Kconfig文件，我们可以看到下面的选项。
```
config USB_STORAGE_DATAFAB
        bool "Datafab Compact Flash Reader support (EXPERIMENTAL)"
       depends on USB_STORAGE && EXPERIMENTAL
       help
         Support for certain Datafab CompactFlash readers.
         Datafab has a web page at <http://www.datafabusa.com/>.
```
显然，这个选项和我们的目的没有关系。首先它专门针对Datafab公司的产品，其次虽然CompactFlash reader是一种flash设备，但显然不是U盘。因为drivers/usb/storage目录下的代码是针对usb mass storage这一类设备，而不是针对某一种特定的设备。U盘只是usb mass storage设备中的一种。再比如：
```
 config USB_STORAGE_SDDR55
         bool "SanDisk SDDR-55 SmartMedia support (EXPERIMENTAL)"
         depends on USB_STORAGE && EXPERIMENTAL
         help
             Say Y here to include additional code to support the Sandisk SDDR-55
             SmartMedia reader in the USB Mass Storage driver.
```
很显然这个选项是有关SanDisk产品的，并且针对的是SM卡，同样不是U盘，所以我们也不需要去关注。

事实上，很容易确定，只有选项CONFIG_USB_STORAGE才是我们真正需要关注的。
```
config USB_STORAGE
      tristate "USB Mass Storage support"
       depends on USB && SCSI
      ---help---
        Say Y here if you want to connect USB mass storage devices to your
        computer's USB port. This is the driver you need for USB
        floppy drives, USB hard disks, USB tape drives, USB CD-ROMs,
        USB flash devices, and memory sticks, along with
        similar devices. This driver may also be used for some cameras
        and card readers.

        This option depends on 'SCSI' support being enabled, but you
          probably also need 'SCSI device support: SCSI disk support'
        (BLK_DEV_SD) for most USB storage devices.

        To compile this driver as a module, choose M here: the
        module will be called usb-storage.
```
接下来阅读Makefile文件。
```make
#
# Makefile for the USB Mass Storage device drivers.
#
# 15 Aug 2000, Christoph Hellwig
# Rewritten to use lists instead of if-statements.
#

EXTRA_CFLAGS    := -Idrivers/scsi

obj-$(CONFIG_USB_STORAGE)    += usb-storage.o

 usb-storage-obj-$(CONFIG_USB_STORAGE_DEBUG)    += debug.o
 usb-storage-obj-$(CONFIG_USB_STORAGE_USBAT)    += shuttle_usbat.o
 usb-storage-obj-$(CONFIG_USB_STORAGE_SDDR09)    += sddr09.o
 usb-storage-obj-$(CONFIG_USB_STORAGE_SDDR55)    += sddr55.o
 usb-storage-obj-$(CONFIG_USB_STORAGE_FREECOM)    += freecom.o
 usb-storage-obj-$(CONFIG_USB_STORAGE_DPCM)    += dpcm.o
 usb-storage-obj-$(CONFIG_USB_STORAGE_ISD200)    += isd200.o
 usb-storage-obj-$(CONFIG_USB_STORAGE_DATAFAB)    += datafab.o
 usb-storage-obj-$(CONFIG_USB_STORAGE_JUMPSHOT)    += jumpshot.o
 usb-storage-obj-$(CONFIG_USB_STORAGE_ALAUDA)    += alauda.o
 usb-storage-obj-$(CONFIG_USB_STORAGE_ONETOUCH)    += onetouch.o
 usb-storage-obj-$(CONFIG_USB_STORAGE_KARMA)    += karma.o

 usb-storage-objs :=    scsiglue.o protocol.o transport.o usb.o /
              initializers.o $(usb-storage-obj-y)

 ifneq ($(CONFIG_USB_LIBUSUAL),)
      obj-$(CONFIG_USB)    += libusual.o
 endif
```
前面通过Kconfig文件的分析，我们确定了只需要去关注CONFIG_USB_STORAGE选项。在Makefile文件里查找CONFIG_USB_STORAGE，从第9行得知，该选项对应的模块为usb-storage。

因为Kconfig文件里的其他选项我们都不需要关注，所以Makefile的11~22行可以忽略。第24行意味着我们只需要关注scsiglue.c、protocol.c、transport.c、usb.c、initializers.c以及它们同名的.h头文件。

Kconfig和Makefile很好的帮助我们定位到了所要关注的目标，就像我们到一个陌生的地方要随身携带地图，当我们学习Linux内核时，也要谨记寻求Kconfig和Makefile的帮助。
