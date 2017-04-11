---
title: Linux Platform Device and Driver
date: 2014-02-25 10:52:33
tags:
categories:
  - Kernel
  - 设备驱动
---

原文：<http://blog.csdn.net/yili_xie/article/details/5187014>

从 Linux 2.6 起引入了一套新的驱动管理和注册机制: `platform_device` 和 `platform_driver` 。  
Linux 中大部分的设备驱动，都可以使用这套机制 , 设备用 `platform_device` 表示，驱动用 `platform_driver` 进行注册。

Linux platform driver 机制和传统的 device driver 机制 ( 通过 `driver_register` 函数进行注册 ) 相比，一个十分明显的优势在于 platform 机制将设备本身的资源注册进内核，由内核统一管理，在驱动程序中使用这些资源时通过 platform device 提供的标准接口进行申请并使用。这样提高了驱动和资源管理的独立性，并且拥有较好的可移植性和安全性 ( 这些标准接口是安全的 ) 。

Platform 机制的本身使用并不复杂，由两部分组成： `platform_device` 和 `platform_driver`。  
通过 Platform 机制开发发底层驱动的大致流程为:  
1）定义 `platform_device`  
2）注册 `platform_device`  
3）定义 `platform_driver`  
4）注册 `platform_driver`  
<!--more-->

首先要确认的就是设备的资源信息，例如设备的地址，中断号等。  
在 2.6 内核中 platform 设备用结构体 `platform_device` 来描述，该结构体定义在 `kernel/include/linux/platform_device.h` 中，
```c
struct platform_device {
    const char *name;
    u32  id;
    struct device dev;
    u32  num_resources;
    struct resource *resource;
};
```
该结构一个重要的元素是 `resource` ，该元素存入了最为重要的设备资源信息，定义在 `kernel/include/linux/ioport.h` 中，
```c
struct resource {
    const char *name;
    unsigned long start, end;
    unsigned long flags;
    struct resource *parent, *sibling, *child;
};
```
下面举 s3c2410 平台的 i2c 驱动作为例子来说明：
```c
/* arch/arm/mach-s3c2410/devs.c */
/* I2C */
static struct resource s3c_i2c_resource[ ] = {
         [0] = {
                   .start = S3C24XX_PA_IIC,
                   .end = S3C24XX_PA_IIC + S3C24XX_SZ_IIC - 1,
                   .flags = IORESOURCE_MEM,
         } ,
         [1] = {
                   .start = IRQ_IIC, //S3C2410_IRQ(27)
                   .end = IRQ_IIC,
                   .flags = IORESOURCE_IRQ,
         }
};
```
这里定义了两组 `resource` ，它描述了一个 I2C 设备的资源，第 1 组描述了这个 I2C 设备所占用的总线地址范围， `IORESOURCE_MEM` 表示第 1 组描述的是内存类型的资源信息，第 2 组描述了这个 I2C 设备的中断号， `IORESOURCE_IRQ` 表示第 2 组描述的是中断资源信息。设备驱动会根据 `flags` 来获取相应的资源信息。

有了 `resource` 信息，就可以定义 `platform_device` 了：
```c
struct platform_device s3c_device_i2c = {
         .name = "s3c2410-i2c",
         .id = -1,
         .num_resources = ARRAY_SIZE(s3c_i2c_resource),
         .resource = s3c_i2c_resource,
};
```
定义好了 `platform_device` 结构体后就可以调用函数 `platform_add_devices` 向系统中添加该设备了，之后可以调用 `platform_driver_register()` 进行设备注册。要注意的是，这里的 `platform_device` 设备的注册过程必须在相应设备驱动加载之前被调用，即执行 `platform_driver_register` 之前 , 原因是因为驱动注册时需要匹配内核中所以已注册的设备名。

`s3c2410-i2c` 的 `platform_device` 是在系统启动时，在 `cpu.c` 里的 `s3c_arch_init()` 函数里进行注册的，这个函数申明为 `arch_initcall(s3c_arch_init);` 会在系统初始化阶段被调用。  
`arch_initcall` 的优先级高于 `module_init` 。所以会在 Platform 驱动注册之前调用。 (详细参考 `include/linux/init.h`)

`s3c_arch_init` 函数如下：
```c
/* arch/arm/mach-3sc2410/cpu.c */
static int __init s3c_arch_init( void )
{
    int ret;
    ……
/* 这里board指针指向在mach-smdk2410.c里的定义的smdk2410_board，里面包含了预先定义的I2C Platform_device等. */
    if ( board != NULL ) {
        struct platform_device **ptr = board->devices;
        int i;

        for ( i = 0; i < board->devices_count; i++ , ptr++ ) {
            ret = platform_device_register(*ptr) ;      //在这里进行注册

            if (ret) {
                printk( KERN_ERR "s3c24xx: failed to add board device %s (%d) @%p/n" , (*ptr)->name,
ret, *ptr) ;
            }
        }
         /* mask any error, we may not need all these board
         * devices */
        ret = 0;
    }
    return ret;
}
```
同时被注册还有很多其他平台的 `platform_device` ，详细查看 `arch/arm/mach-s3c2410/mach-smdk2410.c` 里的 `smdk2410_devices` 结构体。

驱动程序需要实现结构体 `struct platform_driver` ，参考 `drivers/i2c/busses`
```c
/* device driver for platform bus bits */
static struct platform_driver s3c2410_i2c_driver = {
         .probe = s3c24xx_i2c_probe,
         .remove = s3c24xx_i2c_remove,
         .resume = s3c24xx_i2c_resume,
         .driver = {
                   .owner = THIS_MODULE,
                   .name = "s3c2410-i2c" ,
         } ,
};
```
在驱动初始化函数中调用函数 `platform_driver_register()` 注册 `platform_driver` ，需要注意的是 `s3c_device_i2c` 结构中 `name` 元素和 `s3c2410_i2c_driver` 结构中 `driver.name` 必须是相同的，这样在 `platform_driver_register()` 注册时会对所有已注册的所有 `platform_device` 中的 `name` 和当前注册的 `platform_driver` 的 `driver.name` 进行比较，只有找到相同的名称的 `platform_device` 才能注册成功，当注册成功时会调用 `platform_driver` 结构元素 `probe` 函数指针，这里就是` s3c24xx_i2c_probe`, 当进入 `probe` 函数后，需要获取设备的资源信息，常用获取资源的函数主要是：
```c
struct resource *platform_get_resource(struct platform_device *dev, unsigned int type, unsigned int num);
```
根据参数 `type` 所指定类型，例如 `IORESOURCE_MEM` ，来获取指定的资源。
```c
struct int platform_get_irq(struct platform_device *dev, unsigned int num);
```
获取资源中的中断号。

下面举 `s3c24xx_i2c_probe` 函数分析 , 看看这些接口是怎么用的。  
前面已经讲了， `s3c2410_i2c_driver` 注册成功后会调用 `s3c24xx_i2c_probe` 执行，下面看代码：
```c
/* drivers/i2c/busses/i2c-s3c2410.c */

static int s3c24xx_i2c_probe(struct platform_device *pdev)
{
    struct s3c24xx_i2c *i2c = &s3c24xx_i2c;
    struct resource *res;
    int ret;

    /* find the clock and enable it */

    i2c->dev = &pdev-> dev;
    i2c->clk = clk_get( &pdev->dev, "i2c" ) ;
    if (IS_ERR( i2c-> clk) ) {
     dev_err(&pdev->dev, "cannot get clock/n" ) ;
     ret = -ENOENT;
     goto out;
    }

    dev_dbg(&pdev->dev, "clock source %p/n" , i2c->clk) ;
    clk_enable(i2c->clk) ;

    /* map the registers */
    res = platform_get_resource(pdev, IORESOURCE_MEM, 0) ; /* 获取设备的IO资源地址 */
    if ( res == NULL ) {
     dev_err(&pdev->dev, "cannot find IO resource/n") ;
     ret = -ENOENT;
     goto out;
    }

    i2c->ioarea = request_mem_region(res->start, (res->end- res->start) + 1, pdev->name) ; /* 申请这块IO Region */

    if ( i2c->ioarea == NULL ) {
     dev_err(&pdev->dev, "cannot request IO/n" ) ;
     ret = -ENXIO;
     goto out;
    }

    i2c->regs = ioremap(res->start, (res->end- res->start) + 1) ; /* 映射至内核虚拟空间 */

    if ( i2c->regs == NULL ) {
     dev_err( &pdev->dev, "cannot map IO/n" ) ;
     ret = -ENXIO;
     goto out;
    }

    dev_dbg(&pdev->dev, "registers %p (%p, %p)/n" , i2c->regs, i2c->ioarea, res) ;

    /* setup info block for the i2c core */
    i2c->adap.algo_data = i2c;
    i2c->adap.dev.parent = &pdev->dev;

    /* initialise the i2c controller */
    ret = s3c24xx_i2c_init(i2c) ;
    if (ret != 0)
     goto out;

    /* find the IRQ for this unit (note, this relies on the init call to ensure no current IRQs pending */

    res = platform_get_resource(pdev, IORESOURCE_IRQ, 0) ; /* 获取设备IRQ中断号 */

    if ( res == NULL ) {
     dev_err( &pdev->dev, "cannot find IRQ/n" ) ;
     ret = -ENOENT;
     goto out;
    }

    ret = request_irq(res->start, s3c24xx_i2c_irq, IRQF_DISABLED, /* 申请IRQ */
     pdev->name, i2c) ;

    ……

    return ret;
}
```
**小思考：**  
那什么情况可以使用 platform driver 机制编写驱动呢？  
我的理解是只要和内核本身运行依赖性不大的外围设备 ( 换句话说只要不在内核运行所需的一个最小系统之内的设备 ), 相对独立的 , 拥有各自独自的资源 (addresses and IRQs) ， 都可以用 `platform_driver` 实现。如： lcd,usb,uart 等，都可以用 `platform_driver` 写，而 timer,irq 等最小系统之内的设备则最好不用 `platform_driver` 机制，实际上内核实现也是这样的。
