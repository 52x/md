---
title: Linux LCD 驱动学习
date: 2015-04-04 22:45:09
tags:
categories:
  - Kernel
  - 设备驱动
---

原文：[S3C2410 LCD驱动学习心得](http://blog.csdn.net/gzliu_hit/article/details/6719457)

## 实验内容简要描述

**1．实验目的**  
学会驱动程序的编写方法，配置S3C2410的LCD驱动，以及在LCD屏上显示包括bmp和jpeg两种格式的图片  
**2．实验内容**  
（1）分析S3c2410实验箱LCD以及LCD控制器的硬件原理，据此找出相应的硬件设置参数，参考xcale实验箱关于lcd的设置，完成s3c2410实验箱LCD的设置  
（2）在LCD上显示一张BMP图片或JPEG图片  
**3．实验条件（软硬件环境）**  
PC机、S3C2410开发板、PXA255开发板
<!--more-->

## 实验原理

### S3C2410内置LCD控制器分析

#### S3C2410 LCD控制器

一块LCD屏显示图像，不但需要LCD驱动器，还需要有相应的LCD控制器。通常LCD驱动器会以COF/COG的形式与LCD玻璃基板制作在一起，而LCD控制器则由外部电路来实现。而S3C2410内部已经集成了LCD控制器，因此可以很方便地去控制各种类型的LCD屏，例如：STN和TFT屏。S3C2410 LCD控制器的特性如下：  
1. STN屏  
   支持3种扫描方式：4bit单扫、4位双扫和8位单扫  
   支持单色、4级灰度和16级灰度屏  
   支持256色和4096色彩色STN屏（CSTN）  
   支持分辩率为640*480、320*240、160*160以及其它规格的多种LCD
2. TFT屏  
   支持单色、4级灰度、256色的调色板显示模式  
   支持64K和16M色非调色板显示模式  
   支持分辩率为640*480，320*240及其它多种规格的LCD  
   对于控制TFT屏来说，除了要给它送视频资料（`VD[23:0]`）以外，还有以下一些信号是必不可少的，分别是：  
   `VSYNC(VFRAME)`：帧同步信号  
   `HSYNC(VLINE)`：行同步信号  
   `VCLK`：像数时钟信号  
   `VDEN(VM)`：数据有效标志信号

由于本项目所用的S3C2410上的LCD是TFT屏，并且TFT屏将是今后应用的主流，因此接下来，重点围绕TFT屏的控制来进行。  
图1.1 是S3C2410内部的LCD控制器的逻辑示意图：

{% asset_img lcd1.jpg 图1.1 %}

`REGBANK` 是LCD控制器的寄存器组，用来对LCD控制器的各项参数进行设置。而 `LCDCDMA` 则是LCD控制器专用的DMA信道，负责将视频资料从系统总线（System Bus）上取来，通过 `VIDPRCS` 从`VD[23:0]`发送给LCD屏。同时 `TIMEGEN` 和 `LPC3600` 负责产生 LCD屏所需要的控制时序，例如`VSYNC`、`HSYNC`、`VCLK`、`VDEN`，然后从 `VIDEO MUX` 送给LCD屏。

#### TFT屏时序分析

图1.2 是TFT屏的典型时序。其中`VSYNC`是帧同步信号，`VSYNC`每发出1个脉冲，都意味着新的1屏视频资料开始发送。而`HSYNC`为行同步信号，每个`HSYNC`脉冲都表明新的1行视频资料开始发送。而`VDEN`则用来标明视频资料的有效，`VCLK`是用来锁存视频资料的像数时钟。  
在帧同步以及行同步的头尾都必须留有回扫时间，例如对于`VSYNC`来说前回扫时间就是`(VSPW+1)＋(VBPD+1)`，后回扫时间就是`(VFPD +1)`；`HSYNC`亦类同。这样的时序要求是当初CRT显示器由于电子枪偏转需要时间，但后来成了实际上的工业标准，乃至于后来出现的TFT屏为了在时序上与CRT兼容，也采用了这样的控制时序。

{% asset_img lcd2.jpg 图1.2 %}

S3C2410实验箱上的LCD是一款3.5寸TFT真彩LCD屏，分辩率为240*320，下图为该屏的时序要求。

{% asset_img lcd3.jpg 图1.3 %}

通过对比图1.2和图1.3，我们不难看出：
```
VSPW+1=2 => VSPW=1
VBPD+1=2 => VBPD=1
LINVAL+1=320 => LINVAL=319
VFPD+1=3 => VFPD=2
HSPW+1=4 => HSPW=3
HBPD+1=7 => HBPW=6
HOZVAL+1=240 => HOZVAL=239
HFPD+1=31 => HFPD=30
```
以上各参数，除了`LINVAL`和`HOZVAL`直接和屏的分辩率有关，其它的参数在实际操作过程中应以上面的为参考，不应偏差太多。

#### LCD控制器主要寄存器功能详解

{% asset_img lcd4.jpg 图1.4 %}

`LINECNT`：当前行扫描计数器值，标明当前扫描到了多少行。  
`CLKVAL`：决定VCLK的分频比。LCD控制器输出的VCLK是直接由系统总线（AHB）的工作频率HCLK直接分频得到的。作为240*320的TFT屏，应保证得出的VCLK在5~10MHz之间。  
`MMODE`：VM信号的触发模式（仅对STN屏有效，对TFT屏无意义）。  
`PNRMODE`：选择当前的显示模式，对于TFT屏而言，应选择`[11]`，即TFT LCD panel。  
`BPPMODE`：选择色彩模式，对于真彩显示而言，选择16bpp（64K色）即可满足要求。  
`ENVID`：使能LCD信号输出。

{% asset_img lcd5.jpg 图1.5 %}

`VBPD`， `LINEVAL`， `VFPD`， `VSPW` 的各项含义已经在前面的时序图中得到体现。

{% asset_img lcd6.jpg 图1.6 %}

`HBPD`， `HOZVAL`， `HFPD` 的各项含义已经在前面的时序图中得到体现。

{% asset_img lcd7.jpg 图1.7 %}

`HSPW` 的含义已经在前面的时序图中得到体现。  
`MVAL` 只对STN屏有效，对TFT屏无意义。  
`HSPW` 的含义已经在前面的时序图中得到体现，这里不再赘述。  
`MVAL` 只对STN屏有效，对TFT屏无意义。

{% asset_img lcd8.jpg 图1.8 %}

`VSTATUS`：当前`VSYNC`信号扫描状态，指明当前`VSYNC`同步信号处于何种扫描阶段。  
`HSTATUS`：当前`HSYNC`信号扫描状态，指明当前`HSYNC`同步信号处于何种扫描阶段。  
`BPP24BL`：设定`24bpp`显示模式时，视频资料在显示缓冲区中的排列顺序（即低位有效还是高位有效）。对于`16bpp`的64K色显示模式，该设置位无意义。  
`FRM565`：对于`16bpp`显示模式，有2种形式，一种是`RGB = 5:5:5:1`，另一种是`5:6:5`。后一种模式最为常用，它的含义是表示64K种色彩的16bit RGB资料中，红色（R）占了5bit，绿色（G）占了6bit，蓝色（B）占了5bit  
`INVVCLK`，`INVLINE`，`INVFRAME`，`INVVD`：通过前面的时序图，我们知道，CPU的LCD控制器输出的时序默认是正脉冲，而LCD需要`VSYNC(VFRAME)`、`VLINE(HSYNC)`均为负脉冲，因此 `INVLINE` 和 `INVFRAME` 必须设为“1”，即选择反相输出。

{% asset_img 图1.9 %}

`INVVDEN`，`INVPWREN`，`INVLEND` 的功能同前面的类似。  
`PWREN` 为LCD电源使能控制。在CPU LCD控制器的输出信号中，有一个电源使能管脚`LCD_PWREN`，用来做为LCD屏电源的开关信号。  
`ENLEND` 对普通的TFT屏无效，可以不考虑。  
`BSWP` 和 `HWSWP` 为字节（Byte）或半字（Half-Word）交换使能。由于不同的GUI对FrameBuffer（显示缓冲区）的管理不同，必要时需要通过调整 `BSWP` 和 `HWSWP` 来适应GUI。

### Linux 驱动

#### FrameBuffer

Linux是工作在保护模式下，所以用户态进程是无法像DOS那样使用显卡BIOS里提供的中断调用来实现直接写屏，Linux仿显卡的功能，将显存抽象出FrameBuffer这个设备来供用户态进程实现直接写屏。Framebuffer机制将硬件结构抽象掉，可以通过Framebuffer的读写直接对显存进行操作。用户可以将Framebuffer看成是显示内存的一个映像，将其映射到进程地址空间之后，就可以直接进行读写操作，而写操作可以立即反应在屏幕上。这种操作是抽象的，统一的。用户不必关心物理显存的位置、换页机制等等具体细节。这些都是由Framebuffer设备驱动来完成的。

在Linux系统下，FrameBuffer的主要的结构如图所示。Linux为了开发FrameBuffer程序的方便，使用了分层结构。`fbmem.c` 处于Framebuffer设备驱动技术的中心位置。它为上层应用程序提供系统调用，也为下一层的特定硬件驱动提供接口；那些底层硬件驱动需要用到这儿的接口来向系统内核注册它们自己。

{% asset_img lcd10.jpg 图2.1 %}

`fbmem.c` 为所有支持FrameBuffer的设备驱动提供了通用的接口，避免重复工作。下将介绍`fbmem.c`主要的一些数据结构。

#### 数据结构

##### Linux FrameBuffer的数据结构

在FrameBuffer中，`fb_info`可以说是最重要的一个结构体，它是Linux为帧缓冲设备定义的驱动层接口。它不仅包含了底层函数，而且还有记录设备状态的数据。每个帧缓冲设备都与一个`fb_info`结构相对应。`fb_info`的主要成员如下
```c
struct fb_info {
    int node;
    struct fb_var_screeninfo var; /* Current var */
    struct fb_fix_screeninfo fix; /* Current fix */
    struct fb_videomode *mode;    /* current mode */

    struct fb_ops *fbops;
    struct device *device; /* This is the parent */
    struct device *dev;    /* This is this fb device */

    char __iomem *screen_base; /* Virtual address */
    unsigned long screen_size; /* Amount of ioremapped VRAM or 0 */
    …………
};
```
其中`node`成员域标示了特定的FrameBuffer，实际上也就是一个FrameBuffer设备的次设备号。`fb_var_screeninfo`结构体成员记录用户可修改的显示控制器参数，包括屏幕分辨率和每个像素点的比特数。`fb_var_screeninfo`中的`xres`定义屏幕一行有多少个点, `yres`定义屏幕一列有多少个点, `bits_per_pixel`定义每个点用多少个字节表示。其他域见以下代码注释。
```c
struct fb_var_screeninfo {
    __u32 xres;   /* visible resolution */
    __u32 yres;
    __u32 xoffset;  /* offset from virtual to visible */
    __u32 yoffset;  /* resolution */
    __u32 bits_per_pixel; /* bits/pixel */
    __u32 pixclock; /* pixel clock in ps (pico seconds) */
    __u32 left_margin;  /* time from sync to picture */
    __u32 right_margin; /* time from picture to sync */
    __u32 hsync_len;  /* length of horizontal sync */
    __u32 vsync_len;  /* length of vertical sync */
    …………
};
```
在`fb_info`结构体中，`fb_fix_screeninfo`中记录用户不能修改的显示控制器的参数，如屏幕缓冲区的物理地址，长度。当对帧缓冲设备进行映射操作的时候，就是从`fb_fix_screeninfo`中取得缓冲区物理地址的。
```c
struct fb_fix_screeninfo {
    char id[16];        /* identification string eg "TT Builtin" */
    unsigned long smem_start;    /* Start of frame buffer mem (physical address) */
    __u32 smem_len;     /* Length of frame buffer mem */
    unsigned long mmio_start;    /* Start of Mem Mapped I/O(physical address) */
    __u32 mmio_len;      /* Length of Memory Mapped I/O  */
    …………
};
```
`fb_info`还有一个很重要的域就是`fb_ops`。它是提供给底层设备驱动的一个接口。通常我们编写字符驱动的时候，要填写一个`file_operations`结构体，并使用`register_chrdev()`注册之，以告诉Linux如何操控驱动。当我们编写一个FrameBuffer的时候，就要依照Linux FrameBuffer编程的套路，填写`fb_ops`结构体。这个`fb_ops`也就相当于通常的`file_operations`结构体。
```c
struct fb_ops {
    int (*fb_open)(struct fb_info *info, int user);
    int (*fb_release)(struct fb_info *info, int user);
    ssize_t (*fb_read)(struct file *file, char __user *buf, size_t count, loff_t *ppos);
    ssize_t (*fb_write)(struct file *file, const char __user *buf, size_t count, loff_t *ppos);
    int (*fb_set_par)(struct fb_info *info);
    int (*fb_setcolreg)(unsigned regno, unsigned red, unsigned green,
        unsigned blue, unsigned transp, struct fb_info *info);
    int (*fb_setcmap)(struct fb_cmap *cmap, struct fb_info *info)
    int (*fb_mmap)(struct fb_info *info, struct vm_area_struct *vma);
    ……………
}
```
上面的结构体，根据函数的名字就可以看出它的作用，这里不在一一说明。下图给出了Linux FrameBuffer的总体结构，作为这一部分的总结。

{% asset_img lcd11.jpg 图2.2 %}

##### S3C2410中LCD的数据结构

在S3C2410的LCD设备驱动中，定义了`s3c2410fb_info`来标识一个LCD设备，结构体如下：
```c
struct s3c2410fb_info {
    struct fb_info  *fb;
    struct device  *dev;
    struct s3c2410fb_mach_info *mach_info;
    struct s3c2410fb_hw regs;  /* LCD Hardware Regs */
    dma_addr_t  map_dma;  /* physical */
    u_char *   map_cpu;   /* virtual */
    u_int   map_size;
    /* addresses of pieces placed in raw buffer */
    u_char *   screen_cpu;  /* virtual address of buffer */
    dma_addr_t  screen_dma; /* physical address of buffer */
    …………
};
```
成员变量fb指向我们上面所说明的`fb_info`结构体，代表了一个FrameBuffer。`dev`则表示了这个LCD设备。`map_dma`，`map_cpu`，`map_size`这三个域指向了开辟给LCD DMA使用的内存地址。`screen_cpu`，`screen_dma`指向了LCD控制器映射的内存地址。另外regs标识了LCD控制器的寄存器。
```c
struct s3c2410fb_hw {
    unsigned long lcdcon1;
    unsigned long lcdcon2;
    unsigned long lcdcon3;
    unsigned long lcdcon4;
    unsigned long lcdcon5;
};
```
这个寄存器和硬件的寄存器一一对应，主要作为实际寄存器的映像，以便程序使用。  
这个`s3c2410fb_info`中还有一个`s3c2410fb_mach_info`成员域。它存放了和体系结构相关的一些信息，如时钟、LCD设备的GPIO口等等。这个结构体定义为
```c
struct s3c2410fb_mach_info {
    unsigned char fixed_syncs; /* do not update sync/border */
    int type;      /* LCD types */
    int width;     /* Screen size */
    int height;
    struct s3c2410fb_val xres;  /* Screen info */
    struct s3c2410fb_val yres;
    struct s3c2410fb_val bpp;
    struct s3c2410fb_hw  regs;  /* lcd configuration registers */
    /* GPIOs */
    unsigned long gpcup;
    unsigned long gpcup_mask;
    unsigned long gpccon;
    unsigned long gpccon_mask;
    …………
};
```
{% asset_img lcd12.jpg 图2.3 %}

上图表示了S3C2410驱动的整体结构，反映了结构体之间的相互关系

#### 主要代码结构以及关键代码分析

##### FrameBuffer驱动的统一管理

`fbmem.c`实现了Linux FrameBuffer的中间层，任何一个FrameBuffer驱动，在系统初始化时，必须向`fbmem.c`注册，即需要调用`register_framebuffer()`函数，在这个过程中，设备驱动的信息将会存放入名称为`registered_fb`数组中，这个数组定义为
```c
struct fb_info *registered_fb[FB_MAX];
int num_registered_fb;
```
它是类型为`fb_info`的数组，另外`num_register_fb`则存放了注册过的设备数量。  
我们分析一下`register_framebuffer`的代码。
```c
int register_framebuffer(struct fb_info *fb_info)
{
    int i;
    struct fb_event event;
    struct fb_videomode mode;

    if (num_registered_fb == FB_MAX) return -ENXIO; /* 超过最大数量 */
    num_registered_fb++;
    for (i = 0 ; i < FB_MAX; i++)
        if (!registered_fb[i]) break;     /* 找到空余的数组空间 */
    fb_info->node = i;

    fb_info->dev = device_create(fb_class, fb_info->device,
        MKDEV(FB_MAJOR, i), "fb%d", i);  /* 为设备建立设备节点 */
    if (IS_ERR(fb_info->dev)) {
     …………
    } else {
        fb_init_device(fb_info);      /* 初始化改设备 */
    }
    …………
    return 0;
}
```
从上面的代码可知，当FrameBuffer驱动进行注册的时候，它将驱动的`fb_info`结构体记录到全局数组`registered_fb`中，并动态建立设备节点，进行设备的初始化。注意，这里建立的设备节点的次设备号就是该驱动信息在`registered_fb`存放的位置，即数组下标 `i`。在完成注册之后，`fbmem.c`就记录了驱动的`fb_info`。这样我们就有可能实现`fbmem.c`对全部FrameBuffer驱动的统一处理。

##### 实现消息的分派

`fbmem.c`实现了对系统全部FrameBuffer设备的统一管理。当用户尝试使用一个特定的FrameBuffer时，`fbmem.c`怎么知道该调用哪个特定的设备驱动呢？  
我们知道，Linux是通过主设备号和次设备号，对设备进行唯一标识。不同的FrameBuffer设备向`fbmem.c`注册时，程序分配给它们的主设备号是一样的，而次设备号是不一样的。于是我们就可以通过用户指明的次设备号，来决定具体该调用哪一个FrameBuffer驱动。下面通过分析`fbmem.c`的`fb_open()`函数来说明。（注：一般我们写FrameBuffer驱动不需要实现open函数，这里只是说明函数流程。）
```c
static int fb_open(struct inode *inode, struct file *file){
    int fbidx = iminor(inode);
    struct fb_info *info;
    int res;
       /* 得到真正驱动的函数指针 */
    if (!(info = registered_fb[fbidx])) return -ENODEV; 
    if (info->fbops->fb_open) {
        res = info->fbops->fb_open(info,1); //调用驱动的open()
        if (res)  module_put(info->fbops->owner);
    }
    return res;
}
```
当用户打开一个FrameBuffer设备的时，将调用这里的`fb_open()`函数。传进来的`inode`就是欲打开设备的设备号，包括主设备和次设备号。`fb_open`函数首先通过`iminor()`函数取得次设备号，然后查全局数组`registered_fb`得到设备的`fb_info`信息，而这里面存放了设备的操作函数集`fb_ops`。这样，我们就可以调用具体驱动的`fb_open()`函数，实现`open`的操作。下面给出了一个LCD驱动的`open()`函数的调用流程图，用以说明上面的步骤。

{% asset_img lcd13.jpg 图2.4 %}

##### 开发板S3C2410 LCD驱动的流程

（1）在`mach-smdk2410.c`中，定义了初始的LCD参数。注意这是个全局变量。
```c
static struct s3c2410fb_mach_info smdk2410_lcd_cfg = {
    .regs= {
        .lcdcon1 = S3C2410_LCDCON1_TFT16BPP |
            S3C2410_LCDCON1_TFT |
            S3C2410_LCDCON1_CLKVAL(7),
            ......
    },
    .width  = 240,
    .height = 320,
    .xres = {.min = 240, .max= 240, .defval = 240},
    .bpp   = {.min = 16, .max= 16, .defval = 16},
    ......
};
```
（2）内核初始化时候调用`s3c2410fb_probe`函数。下面分析这个函数的做的工作。首先先动态分配`s3c2410fb_info`空间。
```c
fbinfo = framebuffer_alloc(sizeof(struct s3c2410fb_info),&pdev->dev);
```
把域`mach_info`指向`mach-smdk2410.c`中的`smdk2410_lcd_cfg` 。
```c
info->mach_info = pdev->dev.platform_data;
```
设置`fb_info`域的`fix`，`var`，`fops`字段。
```c
fbinfo->fix.type  =  FB_TYPE_PACKED_PIXELS;
fbinfo->fix.type_aux  = 0;
fbinfo->fix.xpanstep  = 0;

fbinfo->var.nonstd    = 0;
fbinfo->var.activate  = FB_ACTIVATE_NOW;
fbinfo->var.height    = mach_info->height;
fbinfo->var.width     = mach_info->width;

fbinfo->fbops  = &s3c2410fb_ops;
……
```
该函数调用`s3c2410fb_map_video_memory()`申请DMA内存，即显存。
```c
fbi->map_size = PAGE_ALIGN(fbi->fb->fix.smem_len + PAGE_SIZE);
fbi->map_cpu  = dma_alloc_writecombine(fbi->dev, fbi->map_size,
          &fbi->map_dma, GFP_KERNEL);

fbi->map_size = fbi->fb->fix.smem_len;
……
```
设置控制寄存器，设置硬件寄存器。
```c
memcpy(&info->regs, &mach_info->regs, sizeof(info->regs));
info->regs.lcdcon1 &= ~S3C2410_LCDCON1_ENVID;
………
```
调用函数`s3c2410fb_init_registers()`，把初始值写入寄存器。
```c
writel(fbi->regs.lcdcon1, S3C2410_LCDCON1);
writel(fbi->regs.lcdcon2, S3C2410_LCDCON2);
```
（3）当用户调用`mmap()`映射内存的时候，`fbmem.c`把刚才设置好的显存区域映射给用户。
```c
start = info->fix.smem_start;
len = PAGE_ALIGN((start & ~PAGE_MASK) + info->fix.smem_len);
io_remap_pfn_range(vma, vma->vm_start, off >> PAGE_SHIFT,
    vma->vm_end - vma->vm_start, vma->vm_page_prot)；
……
```
这样就完成了驱动初始化到用户调用的整个过程。

### BMP和JPEG图形显示程序

#### 在LCD上显示BMP或JPEG图片的主流程图

首先，在程序开始前。要在`nfs/dev`目录下创建LCD的设备结点，设备名`fb0`,设备类型为字符设备，主设备号为`29`，次设备号为`0`。命令如下：
```bash
mknod fb0 c 29 0
```
在LCD上显示图象的主流程图如图3.1所示。程序一开始要调用`open`函数打开设备，然后调用`ioctl`获取设备相关信息，接下来就是读取图形文件数据，把图象的RGB值映射到显存中，这部分是图象显示的核心。对于JPEG格式的图片，要先经过JPEG解码才能得到RGB数据，本项目中直接采用现成的JPEG库进行解码。对于bmp格式的图片，则可以直接从文件里面提取其RGB数据。要从一个bmp文件里面把图片数据阵列提取出来，首先必须知道bmp文件的格式。下面来详细介绍bmp文件的格式。

{% asset_img lcd14.jpg 图3.1 %}

#### bmp位图格式分析

位图文件可看成由四个部分组成：位图文件头、位图信息头、彩色表和定义位图的字节阵列。如图3.2所示。

{% asset_img lcd15.jpg 图3.2 %}

（1）文件头中各个段的地址及其内容如图3.3。

{% asset_img lcd16.jpg 图3.3 %}

位图文件头数据结构包含BMP图象文件的类型，显示内容等信息。它的数据结构如下定义：
```c
typedef struct
{
    int  bfType；//表明位图文件的类型，必须为BM
    long bfSize；//表明位图文件的大小，以字节为单位
    int  bfReserved1；//属于保留字，必须为本0
    int  bfReserved2；//也是保留字，必须为本0
    long bfOffBits；//位图阵列的起始位置，以字节为单位
} BITMAPFILEHEADER；
```
（2）信息头中各个段的地址及其内容如图3.4所示。

{% asset_img lcd17.jpg 图3.4 %}

位图信息头的数据结构包含了有关BMP图象的宽，高，压缩方法等信息，它的C语言数据结构如下所示。
```c
typedef struct {
    long  biSize； //指出本数据结构所需要的字节数
    long  biWidth；//以象素为单位，给出BMP图象的宽度
    long  biHeight；//以象素为单位，给出BMP图象的高度
    int   biPlanes；//输出设备的位平面数，必须置为1
    int   biBitCount；//给出每个象素的位数
    long  biCompress；//给出位图的压缩类型
    long  biSizeImage；//给出图象字节数的多少
    long  biXPelsPerMeter；//图像的水平分辨率
    long  biYPelsPerMeter；//图象的垂直分辨率
    long  biClrUsed；//调色板中图象实际使用的颜色素数
    long  biClrImportant；//给出重要颜色的索引值
} BITMAPINFOHEADER；
```
（3）对于象素小于或等于16位的图片，都有一个颜色表用来给图象数据阵列提供颜色索引，其中的每块数据都以B、G、R的顺序排列，还有一个是`reserved`保留位。而在图形数据区域存放的是各个象素点的索引值。它的C语言数据结构如下所示。
```c
typedef struct {
    unsigned char b;
    unsigned char g;
    unsigned char r;
    unsigned char reserved;
} ColorTable;
```
4）对于24位和32位的图片，没有彩色表，它在图象数据区里直接存放图片的RGB数据，其中的每个象素点的数据都以B、G、R的顺序排列。每个象素点的数据结构如下所示。
```c
typedef struct {
    unsigned char b;
    unsigned char g;
    unsigned char r;
    unsigned char reserved;
} RGB;
```
（5）由于图象数据阵列中的数据是从图片的最后一行开始往上存放的，因此在显示图象时，是从图象的左下角开始逐行扫描图象，即从左到右，从下到上。  
（6）对S3C2410或PXA255开发板上的LCD来说，他们每个象素点所占的位数为16位，这16位按`B：G：R=5：6：5`的方式分，其中B在最高位，R在最低位。而从bmp图象得到的R、G、B数据则每个数据占8位，合起来一共24位，因此需要对该R、G、B数据进行移位组合成一个16位的数据。移位方法如下：
```c
b >>= 3; g >>= 2; r >>= 3;
RGBValue = ( r<<11 | g << 5 | b);
```
基于以上分析，提取各种类型的bmp图象的流程如图3.5所示

{% asset_img lcd18.jpg 图3.5 %}

#### 实现显示任意大小的图片

开发板上的LCD屏的大小是固定的，S3C2410上的LCD为：240*320，PXA255上的为：640*480。比屏幕小的图片在屏上显示当然没问题，但是如果图片比屏幕大呢？这就要求我们通过某种算法对图片进行缩放。  
缩放的基本思想是将图片分成若干个方块，对每个方块中的R、G、B数据进行取平均，得到一个新的R、G、B值，这个值就作为该方块在LCD屏幕上的映射。  
缩放的算法描述如下：  
(1)、计算图片大小与LCD屏大小的比例，以及方块的大小。为了适应各种屏幕大小，这里并不直接给lcd_width和lcd_height赋值为240和320。而是调用标准的接口来获取有关屏幕的参数。具体如下：  
```c
// Get variable screen information
if (ioctl(fbfd, FBIOGET_VSCREENINFO, &vinfo)) {
    printf("Error reading variable information.");
    exit(3);
}
unsigned int lcd_width=vinfo.xres;
unsigned int lcd_height=vinfo.yres;
```
计算比例：
```c
widthScale=bmpi->width/lcd_width;
heightScale=bmpi->height/lcd_height;
```
本程序中方块的大小以如下的方式确定：  
注：以下两行代码原文内容已缺失，待补充。
```c
unsigned int paneWidth= ;
unsigned int paneHeight= ;
```
符号 代表向上取整。  
(2)、从图片的左上角开始，以(`i * widthScale`，`j * heightScale`)位起始点，以宽`paneWidth`高`paneHeight`为一个小方块，对该方块的R、G、B数值分别取平均，得到映射点的R、G、B值，把该点作为要在LCD上显示的第（`i` , `j`）点存储起来。
这部分的程序如下：
```c
//-------------取平均--------
for(i = 0; i < lcd_height; i++)
{
    for(j = 0; j < lcd_width; j++)
    {
        color_sum_r = 0;
        color_sum_g = 0;
        color_sum_b = 0;
        for(m = i*heightScale; m < i*heightScale+paneHeight; m++)
        {
            for(n = j*widthScale; n < j*widthScale+paneWidth; n++)
            {
                color_sum_r += pointvalue[m][n].r;
                color_sum_g += pointvalue[m][n].g;
                color_sum_b += pointvalue[m][n].b;
            }
        }
         RGBvalue_256->r = div_round(color_sum_r, paneHeight*paneWidth);
         RGBvalue_256->g = div_round(color_sum_g, paneHeight*paneWidth);
         RGBvalue_256->b = div_round(color_sum_b, paneHeight*paneWidth);
    }
}
```
#### 图片数据提取及显示的总流程

通过以上的分析，整个图片数据提取及显示的总流程如图3.6所示。

{% asset_img lcd19.jpg 图3.6 %}
