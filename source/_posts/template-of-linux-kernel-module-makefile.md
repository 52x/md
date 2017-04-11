---
title: Linux内核模块编译Makefile模板
date: 2014-03-13 12:09:20
tags:
categories:
  - Kernel
  - 设备驱动
---

原文链接：[Linux内核模块LKM编译-自制Makefile模板](http://blog.chinaunix.net/uid-20543672-id-3241147.html) By Tekkaman Ninja

根据LDD3的内核模块makefile和原理说明，根据自己的需要做了适当的修改使得这个Makefile脚本可以方便被应用于不同的简单模块编译，并可以在模块需要编译进内核的时候直接放入内核源码目录中，脚本如下：
```make
MODULE_NAME = hello_linux_simple
MODULE_CONFIG = CONFIG_HELLO_LINUX_SIMPLE
CROSS_CONFIG = y
# Comment/uncomment the following line to disable/enable debugging
DEBUG = y

# If KERNELRELEASE is defined, we've been invoked from the
# kernel build system and can use its language.
ifneq ($(KERNELRELEASE),)

# Add your debugging flag (or not) to CFLAGS
ifeq ($(DEBUG),y)
	DEBFLAGS = -O -g -DDEBUG # "-O" is needed to expand inlines
else
	DEBFLAGS = -O2
endif

ccflags-y += $(DEBFLAGS)

obj-$($(MODULE_CONFIG)) := $(MODULE_NAME).o
#for Multi-files module
$(MODULE_NAME)-objs := hello_linux_simple_dep.o ex_output.o

# Otherwise we were called directly from the command line;
# invoke the kernel build system.
else

ifeq ($(CROSS_CONFIG), y)
#for Cross-compile
KERNELDIR = （内核源码路径）
ARCH = arm
#FIXME:maybe we need absolute path for different user. eg root
#CROSS_COMPILE = arm-none-linux-gnueabi-
CROSS_COMPILE = （交叉编译工具路径）
INSTALLDIR := （目标模块所安装的根文件系统路径）

else
#for Local compile
KERNELDIR = /lib/modules/$(shell uname -r)/build
ARCH = x86
CROSS_COMPILE =
INSTALLDIR := /

endif

################################
PWD := $(shell pwd)

.PHONY: modules modules_install clean

modules:
	$(MAKE) ARCH=$(ARCH) CROSS_COMPILE=$(CROSS_COMPILE) $(MODULE_CONFIG)=m -C $(KERNELDIR) M=$(PWD) modules

modules_install: modules
	$(MAKE) ARCH=$(ARCH) CROSS_COMPILE=$(CROSS_COMPILE) $(MODULE_CONFIG)=m -C $(KERNELDIR) INSTALL_MOD_PATH=$(INSTALLDIR) M=$(PWD) modules_install

clean:
	@rm -rf *.o *~ core .depend .*.cmd *.ko *.mod.c .tmp_versions *.symvers *.order .*.o.d modules.builtin

endif
```
<!--more-->

这个脚本与模块源码放置于同一个目录。针对不同的模块，只要简单的修改部分参数，使用时只需要在该目录下执行一个简单的“make”命令即可。下面我简单分析一下：  
上面的示例脚本利用了扩展的 GNU make 语法，这个 makefile 在编译内核模块的时候会要被读取 2 次。

第一次：当从命令行执行“make”命令时，“make”会读取这个makefile。此时由于“`KERNELRELEASE`”变量没有被设置，所以会执行“`else`”的部分，也就是“`modules`”目标下的指令，类似我们上面讲的的编译命令“`make -C $() M=$() modules`”。只不过这里为了通用性添加了一些变量而已。

第二次：当执行了上面的指令后，make 命令( 在 makefile 里参数化成 `$(MAKE)` )调用内核编译系统。再次读取这个makefile。由于内核编译系统设置了“`KERNELRELEASE`”变量，所以此次内核编译系统看到了“`obj-$($(MODULE_CONFIG)) := $(MODULE_NAME).o`”也就是类似之前我们描述的“`obj-m`”。这样内核的编译系统就可以完成实际的模块编译工作。

这种模块编译Makefile只需做很小的改动就可以方便的应用于不同的模块中。对于不同的模块你可能需要修改：
```make
MODULE_NAME =   （模块名）
MODULE_CONFIG = （在模块编译进内核时的配置选项）
CROSS_CONFIG = y（是否为交叉编译）
DEBUG = y       （是否定义调试标志）
......
$(MODULE_NAME)-objs := （若为多文件模块，则在此列出。否则用#屏蔽)
......
ifeq ($(CROSS_CONFIG), y)
#for Cross-compile
KERNELDIR = （内核源码路径）
ARCH = arm（交叉编译时，目标CPU构架名，此处为arm）
#FIXME:maybe we need absolute path for different user. eg root
#CROSS_COMPILE = arm-none-linux-gnueabi-
CROSS_COMPILE = （交叉编译工具路径及前缀）
INSTALLDIR := （目标模块所安装的根文件系统路径）
else
#for Local compile
......
ARCH = x86(这个根据本地构架可能需要修改)
......
endif
```
对于这个Makefile，还有一点就是考虑到直接放入内核目录，编译进内核的情况。如果只是简单的模块，可以再次利用这个Makefile。这就是为什么上面的Makefile比较繁琐，因为它同时支持直接放入内核源码树中使用。

假设我们将一个名为`hello_linux_simple`的模块编译入内核中，我们需要做的工作就是将包含以上Makefile和源码的目录拷贝到一个目录（例如`drivers/misc`）下，并适当修改该目录下的内核编译系统`Kconfig`和`Makefile`文件。

在（`drivers/misc/`）`Kconfig`中添加：
```
config HELLO_LINUX_SIMPLE
	tristate "simple hello_linux module"
	# depends on
	help
	  simple hello_linux module
```
由于此模块不依赖其他模块，“`depends on`”就可以屏蔽了。

在（`drivers/misc/`）`Makefile`中添加：
```make
obj-$(CONFIG_HELLO_LINUX_SIMPLE) += hello_linux_simple/
```
注意：上面的`CONFIG_HELLO_LINUX_SIMPLE`必须要和模块源码Makefile中的`MODULE_CONFIG`的值一致。  
文件修改好后，就可以配置内核了。在内核的`make menuconfig`中，我们可以看到：
```bash
Device Drivers -→
[*] Misc devices --->
< > simple hello_linux module
```
既可以用“`M`”编译成模块，也可以用“`Y`”编译进内核中。

**关于调试选项DEBUG**  
上面定义了`DEBUG=y`的选项，是为了在调试的时候启用`pr_debug`和`pr_devel`宏，这些宏是`printk`的封装（参见[《内核日志及printk结构浅析》](http://blog.chinaunix.net/uid-20543672-id-3211832.html)），或者可以开启其他依赖`DEBUG`定义的宏。这样在调试结束之后就可以方便的通过屏蔽`#DEBUG=y`来关闭调试信息的输出，不产生调试信息代码。
