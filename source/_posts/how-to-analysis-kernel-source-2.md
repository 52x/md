---
title: 分析内核源码如何入手（下）
tags:
categories:
  - Kernel
  - 内核修炼之道
date: 2014-04-17 12:46:02
---

原文：[《Linux内核修炼之道》精华分享与讨论（7）——分析内核源码如何入手？（下） ](http://blog.csdn.net/fudan_abc/article/details/5355062)

## 从初始化函数开始

有了地图Kconfig和Makefile，我们可以在庞大复杂的内核代码中定位以及缩小了目标代码的范围。那么现在，为了研究内核对USB子系统的实现，我们还需要在目标代码中找到一个突破口，这个突破口就是USB子系统的初始化代码。
<!--more-->
针对某个子系统或某个驱动，内核使用`subsys_initcall`或`module_init`宏指定初始化函数。在drivers/usb/core/usb.c文件中，我们可以发现下面的代码。
```c
subsys_initcall(usb_init);
module_exit(usb_exit);
```
我们看到一个`subsys_initcall`，它也是一个宏，我们可以把它理解为`module_init`，只不过因为这部分代码比较核心，开发者们把它看作一个子系统，而不仅仅是一个模块。这也很好理解，usbcore这个模块它代表的不是某一个设备，而是所有USB设备赖以生存的模块，Linux中，像这样一个类别的设备驱动被归结为一个子系统。比如PCI子系统，比如SCSI子系统，基本上，drivers/目录下面第一层的每个目录都算一个子系统，因为它们代表了一类设备。

`subsys_initcall(usb_init)`的意思就是告诉我们`usb_init`是USB子系统真正的初始化函数，而`usb_exit()`将是整个USB子系统的结束时的清理函数。于是为了研究USB子系统在内核中的实现，我们需要从`usb_init`函数开始看起。
```c
static int __init usb_init(void)
{
    int retval;
    if (nousb) {
        pr_info("%s: USB support disabled/n", usbcore_name);
        return 0;
    }

    retval = ksuspend_usb_init();
    if (retval)
        goto out;
    retval = bus_register(&usb_bus_type);
    if (retval)
        goto bus_register_failed;
    retval = usb_host_init();
    if (retval)
        goto host_init_failed;
    retval = usb_major_init();
    if (retval)
        goto major_init_failed;
    retval = usb_register(&usbfs_driver);
    if (retval)
        goto driver_register_failed;
    retval = usb_devio_init();
    if (retval)
        goto usb_devio_init_failed;
    retval = usbfs_init();
    if (retval)
        goto fs_init_failed;
    retval = usb_hub_init();
    if (retval)
        goto hub_init_failed;
    retval = usb_register_device_driver(&usb_generic_driver, THIS_MODULE);
    if (!retval)
        goto out;

    usb_hub_cleanup();
hub_init_failed:
    usbfs_cleanup();
fs_init_failed:
    usb_devio_cleanup();
usb_devio_init_failed:
    usb_deregister(&usbfs_driver);
driver_register_failed:
    usb_major_cleanup();
major_init_failed:
    usb_host_cleanup();
host_init_failed:
    bus_unregister(&usb_bus_type);
bus_register_failed:
    ksuspend_usb_cleanup();
out:
    return retval;
}
```

### __init标记

关于`usb_init`，第一个问题是，`__init`标记具有什么意义？

写过驱动的应该不会陌生，它对内核来说就是一种暗示，表明这个函数仅在初始化期间使用，在模块被装载之后，它占用的资源就会释放掉用作它处。它的暗示你懂，可你的暗示，她却不懂或者懂装不懂，多么让人感伤。它在自己短暂的一生中一直从事繁重的工作，吃的是草吐出的是牛奶，留下的是整个USB子系统的繁荣。

受这种精神所感染，我觉得有必要为它说的更多些。`__init`的定义在include/linux/init.h文件里
```c
#define __init    __attribute__ ((__section__ (".init.text")))
```
好像这里引出了更多的疑问，`__attribute__`是什么？Linux内核代码使用了大量的GNU C扩展，以至于GNU C成为能够编译内核的唯一编译器，GNU C的这些扩展对代码优化、目标代码布局、安全检查等方面也提供了很强的支持。而`__attribute__`就是这些扩展中的一个，它主要被用来声明一些特殊的属性，这些属性主要被用来指示编译器进行特定方面的优化和更仔细的代码检查。GNU C支持十几个属性，`section`是其中的一个，我们查看GCC的手册可以看到下面的描述：
> ‘section ("section-name")'>
> 　　 Normally, the compiler places the code it generates in the `text'>
> 　　 section.　Sometimes, however, you need additional sections, or you>
> 　　 need certain particular functions to appear in special sections.>
> 　　 The `section' attribute specifies that a function lives in a>
> 　　 particular section.　For example, the declaration:>
>
> 　　　　　extern void foobar (void) __attribute__ ((section ("bar")));>
>
> 　　 puts the function ‘foobar' in the ‘bar' section.>
>
> 　　 Some file formats do not support arbitrary sections so the>
> 　　 ‘section' attribute is not available on all platforms.　If you>
> 　　 need to map the entire contents of a module to a particular>
> 　　 section, consider using the facilities of the linker instead.
通常编译器将函数放在`.text`节，变量放在`.data`或`.bss`节，使用`section`属性，可以让编译器将函数或变量放在指定的节中。那么前面对`__init`的定义便表示将它修饰的代码放在`.init.text`节。链接器可以把相同节的代码或数据安排在一起，比如`__init`修饰的所有代码都会被放在`.init.text`节里，初始化结束后就可以释放这部分内存。

问题可以到此为止，也可以更深入，即<span style="color:red;">内核又是如何调用到这些`__init`修饰的初始化函数？</span>要回答这个问题，还需要回顾一下`subsys_initcall`宏，它也在include/linux/init.h里定义
```c
#define subsys_initcall(fn)    __define_initcall("4",fn,4)
```
这里又出现了一个宏`__define_initcall`，它用于将指定的函数指针`fn`放到`.initcall.init`节里 而对于具体的`subsys_initcall`宏，则是把`fn`放到`.initcall.init`的子节`.initcall4.init`里。要弄清楚`.initcall.init`、`.init.text`和`.initcall4.init`这样的东东，我们还需要了解一点内核可执行文件相关的概念。

内核可执行文件由许多链接在一起的对象文件组成。对象文件有许多节，如文本、数据、init数据、bss等等。这些对象文件都是由一个称为链接器脚本的文件链接并装入的。这个链接器脚本的功能是将输入对象文件的各节映射到输出文件中；换句话说，它将所有输入对象文件都链接到单一的可执行文件中，将该可执行文件的各节装入到指定地址处。vmlinux.lds是存在于arch/&lt;target&gt;/ 目录中的内核链接器脚本，它负责链接内核的各个节并将它们装入内存中特定偏移量处。

我可以负责任的告诉你，要看懂vmlinux.lds这个文件是需要一番功夫的，不过大家都是聪明人，聪明人做聪明事，所以你需要做的只是搜索initcall.init，然后便会看到似曾相识的内容
```asm
__inicall_start = .;
.initcall.init : AT(ADDR(.initcall.init) – 0xC0000000) {
*(.initcall1.init)
*(.initcall2.init)
*(.initcall3.init)
*(.initcall4.init)
*(.initcall5.init)
*(.initcall6.init)
*(.initcall7.init)
}
__initcall_end = .;
```
这里的`__initcall_start`指向`.initcall.init`节的开始，`__initcall_end`指向它的结尾。而`.initcall.init`节又被分为了7个子节，分别是
```asm
.initcall1.init
.initcall2.init
.initcall3.init
.initcall4.init
.initcall5.init
.initcall6.init
.initcall7.init
```
我们的`subsys_initcall`宏便是将指定的函数指针放在了`.initcall4.init`子节。其它的比如`core_initcall`将函数指针放在`.initcall1.init`子节，`device_initcall`将函数指针放在了`.initcall6.init`子节等等，都可以从include/linux/init.h文件找到它们的定义。<span style="color:red;">各个子节的顺序是确定的，即先调用`.initcall1.init`中的函数指针，再调用`.initcall2.init`中的函数指针，等等。`__init`修饰的初始化函数在内核初始化过程中调用的顺序和`.initcall.init`节里函数指针的顺序有关，不同的初始化函数被放在不同的子节中，因此也就决定了它们的调用顺序。</span>

至于实际执行函数调用的地方，就在/init/main.c文件里，内核的初始化么，不在那里还能在哪里，里面的`do_initcalls`函数会直接用到这里的`__initcall_start`、`__initcall_end`来进行判断。

相关宏定义(include/linux/init.h)：
```c
/* initcalls are now grouped by functionality into separate
 * subsections. Ordering inside the subsections is determined
 * by link order.
 * For backwards compatibility, initcall() puts the call in
 * the device init subsection.
 *
 * The `id' arg to __define_initcall() is needed so that multiple initcalls
 * can point at the same handler without causing duplicate-symbol build errors.
 */

#define __define_initcall(level,fn,id) \
        static initcall_t __initcall_##fn##id __attribute_used__ \
        __attribute__((__section__(".initcall" level ".init"))) = fn

/*
 * A "pure" initcall has no dependencies on anything else, and purely
 * initializes variables that couldn't be statically initialized.
 *
 * This only exists for built-in code, not for modules.
 */
#define pure_initcall(fn)               __define_initcall("0",fn,1)

#define core_initcall(fn)               __define_initcall("1",fn,1)
#define core_initcall_sync(fn)          __define_initcall("1s",fn,1s)
#define postcore_initcall(fn)           __define_initcall("2",fn,2)
#define postcore_initcall_sync(fn)      __define_initcall("2s",fn,2s)
#define arch_initcall(fn)               __define_initcall("3",fn,3)
#define arch_initcall_sync(fn)          __define_initcall("3s",fn,3s)
#define subsys_initcall(fn)             __define_initcall("4",fn,4)
#define subsys_initcall_sync(fn)        __define_initcall("4s",fn,4s)
#define fs_initcall(fn)                 __define_initcall("5",fn,5)
#define fs_initcall_sync(fn)            __define_initcall("5s",fn,5s)
#define rootfs_initcall(fn)             __define_initcall("rootfs",fn,rootfs)
#define device_initcall(fn)             __define_initcall("6",fn,6)
#define device_initcall_sync(fn)        __define_initcall("6s",fn,6s)
#define late_initcall(fn)               __define_initcall("7",fn,7)
#define late_initcall_sync(fn)          __define_initcall("7s",fn,7s)

#define __initcall(fn) device_initcall(fn)

/**
 * module_init() - driver initialization entry point
 * @x: function to be run at kernel boot time or module insertion
 *
 * module_init() will either be called during do_initcalls() (if
 * builtin) or at module insertion time (if a module).  There can only
 * be one per module.
 */
#define module_init(x)  __initcall(x);
```

### 模块参数

关于`usb_init`函数，第二个问题是，`nousb`表示什么？

知道C语言的人都会知道`nousb`是一个标志，只是不同的标志有不一样的精彩，这里的`nousb`是用来让我们在启动内核的时候通过内核参数去掉USB子系统的，Linux社会是一个很人性化的世界，它不会去逼迫我们接受USB，一切都只关乎我们自己的需要。不过我想我们一般来说是不会去指定`nousb`的吧。如果你真的指定了`nousb`，那它就只会幽怨的说一句“USB support disabled”，然后退出`usb_init`。

nousb在drivers/usb/core/usb.c文件中定义为：
```c
static int nousb; /* Disable USB when built into kernel image */
/* format to disable USB on kernel command line is: nousb */
__module_param_call("", nousb, param_set_bool, param_get_bool, &nousb, 0444);

static int usb_autosuspend_delay = 2;           /* Default delay value, in seconds */
module_param_named(autosuspend, usb_autosuspend_delay, uint, 0644);
MODULE_PARM_DESC(autosuspend, "default autosuspend delay");
```
从中可知`nousb`是个模块参数。关于模块参数，我们都知道可以在加载模块的时候可以指定，但是如何在内核启动的时候指定？
打开系统的grub文件，然后找到kernel行，比如：
```
kernel  /boot/vmlinuz-2.6.18-kdb root=/dev/sda1 ro splash=silent vga=0x314
```
其中的`ro`，`splash`，`vga`等都表示内核参数。当某一模块被编译进内核的时候，它的模块参数便需要在kernel行来指定，格式为“<span style="color:red;">模块名.参数=值</span>”，比如：
```bash
modprobe usbcore autosuspend=2
```
对应到kernel行，即为 ：
```bash
usbcore.autosuspend=2
```
通过命令`modinfo -p ${modulename}`可以得知一个模块有哪些参数可以使用。同时，对于已经加载到内核里的模块，它们的模块参数会列举在`/sys/module/${modulename}/parameters/`目录下面，可以使用`echo -n ${value} > /sys/module/${modulename}/parameters/${parm}`这样的命令去修改。

### 可变参数宏

关于`usb_init`函数，第三个问题是，`pr_info`如何实现与使用？

`pr_info`只是一个打印信息的可辨参数宏，`printk`的变体，在include/linux/kernel.h里定义：
```c
#define pr_info(fmt,arg...) /
        printk(KERN_INFO fmt,##arg)
```
99年的ISO C标准里规定了可变参数宏，和函数语法类似，比如
```c
#define debug(format, ...) fprintf (stderr, format, __VA_ARGS__)
```
里面的“`…`”就表示可变参数，调用时，它们就会替代宏体里的`__VA_ARGS__`。GCC总是会显得特立独行一些，它支持更复杂的形式，可以给可变参数取个名字，比如
```c
#define debug(format, args...) fprintf (stderr, format, args)
```
有了名字总是会容易交流一些。是不是与`pr_info`比较接近了？除了‘`##`’，它主要是针对空参数的情况。既然说是可变参数，那传递空参数也总是可以的，空即是多，多即是空，股市里的哲理这里同样也是适合的。如果没有‘`##`’，传递空参数的时候，比如
```c
debug ("A message");
```
展开后，里面的字符串后面会多个多余的逗号。这个逗号你应该不会喜欢，而‘`##`’则会使预处理器去掉这个多余的逗号。

关于`usb_init`函数，上面的三个问题之外，余下的代码分别完成usb各部分的初始化，接下来就需要围绕它们分别进行深入分析。这里只是演示如何入手分析，所以就不做深入分析了。
