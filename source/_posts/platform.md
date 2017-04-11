---
title: 驱动模型之平台设备
tags:
categories:
  - Kernel
  - 内核文档
date: 2014-03-05 16:08:08
---

翻译原文链接：<http://m.blog.csdn.net/blog/guoshaobei/4971881>
内核文档：Documentation/driver-model/platform.txt
<!--more-->

## 平台设备与驱动

查看<linux/platform_device.h>中关于平台总线的驱动模型接口：平台设备和平台驱动。这个伪总线用于连接具有最小基础设施的总线上的设备，例如SOC上的用于集成外设的总线，或者是老式的PC互联总线，但是不包括大型的有规范定义的总线，比如PCI或者USB。

### 平台设备

平台设备是系统中一些典型的自治设备(autonomous entities)。包括老式的基于端口的设备，外设总线上的北桥(host bridge)，以及SOC上的绝大多数控制器。罕见地，一个平台设备通过一段其他类型的总线连接到系统，而它的寄存器仍然是直接寻址的。

平台设备有一个'`name`'域，用于驱动绑定；以及一个`resource`链表，诸如地址和IRQs。
```c
struct platform_device {
    const char    *name;
    u32        id;
    struct device    dev;
    u32        num_resources;
    struct resource    *resource;
};
```

### 平台驱动

平台驱动遵循标准驱动模型惯例，检测/枚举在驱动之外处理，而驱动提供`probe()`和`remove()`方法。它们使用标准惯例接口来支持电源管理和关闭通知。
```c
struct platform_driver {
    int (*probe)(struct platform_device *);
    int (*remove)(struct platform_device *);
    void (*shutdown)(struct platform_device *);
    int (*suspend)(struct platform_device *, pm_message_t state);
    int (*suspend_late)(struct platform_device *, pm_message_t state);
    int (*resume_early)(struct platform_device *);
    int (*resume)(struct platform_device *);
    struct device_driver driver;
};
```
记住`probe()`函数应当验证指定的硬件设备的确存在；有时平台设置代码不能确认这一点。`probe()`能够使用设备的资源，包括时钟和设备平台数据(device platform_data)。

平台驱动用下面正常的方式来注册它们自己：
```c
    int platform_driver_register(struct platform_driver *drv);
```
或者，在设备没有热插拔功能的情况下，`probe()`函数可以放置到init section，以减少驱动在运行时的内存过度占用：
```c
    int platform_driver_probe(struct platform_driver *drv,
              int (*probe)(struct platform_device *))
```

## 设备枚举

一般来说，平台相关(以及板极相关)的设置代码使用下列函数接口注册平台设备：
```c
    int platform_device_register(struct platform_device *pdev);
    int platform_add_devices(struct platform_device **pdevs, int ndev);
```
这个通用规则仅注册那些实际存在的设备，但在一些案例里其他的设备也可以注册。例如，内核也许需要配置来驱动一个额外的网络适配器工作，但是这个适配器并不是在所有的目标板上实际存在；类似地，内核也许需要配置来驱动一个集成的控制器，但是这个控制器在一些目标板上并没有挂载任何实际外设。

在一些案例里，启动固件会导出一个表(tables)，来描述在一个给定的目标板上实际存在的设备。没有这样的表的话，通常系统设置代码来设置设备的唯一办法就是为特定的目标板编译一个特定内核。在嵌入式以及定制的系统开发里，编译针对某个特定目标板的内核是常用的方法。

在许多案例中，与设备相关的内存和IRQ资源并不足以让设备驱动工作。板极设置代码将使用设备的 `platform_data`域来提供一些其他的重要的设备信息。

嵌入式系统中的平台设备经常需要一个或者多个时钟，这些时钟除了在实际被用到的时候，通常保持关闭状态(节省电源)。系统也会设置这些与设备关联的时钟，通过调用`clk_get(&pdev->dev, clock_name)`来返回所需要的时钟。

## 老式驱动：设备检测

一些驱动并没有完全转换到新的设备驱动模型，因为他们还承担着一个驱动之外的职责：由驱动注册自己的平台设备，而不由系统基础设施完成。这样的驱动不能被热插拔或者冷插拔，因为热/冷插拔机制要求设备的创建放置在一些不同的系统组件里而不是驱动里。

这种做法唯一'好'的理由是用于处理一些老式的系统设计，比如像以前的IBM PC机，依赖于容易出错的"probe-the-hardware"模型来进行硬件配置。新的系统设计已经取缔这种模式，更倾向于总线级的动态配置支持（PCI，USB），或者是由启动固件提供的设备列表（例如PNPACPI on x86）。关于什么东西出现在哪儿有太多冲突的可能,即使由操作系统来做有根据的猜测, 也难免因频繁出错而惹麻烦.

这种驱动模型是不鼓励使用的。如果你在更新这样的驱动，请试图将设备枚举移植到一个更适合的地方，驱动之外。这样的据测可以使驱动净化，一开始处于正常的驱动模型，设备节点由PNP或者平台设备设置代码来完成。

尽管如此，内核依然保留一些APIs来支持老式的驱动。尽量少用这些调用，除非你的驱动缺乏热插拔支持。
```c
    struct platform_device *platform_device_alloc(
            const char *name, int id);
```
你可以使用`platform_device_alloc()`来动态地分配一个设备，用它进行资源初始化和`platform_device_register()`调用。
通常一个更好的解决方案是：
```c
    struct platform_device *platform_device_register_simple(
            const char *name, int id,
            struct resource *res, unsigned int nres);
```
你可以使用`platform_device_register_simple()`调用，一步完成分配和注册一个设备。

## 设备命名和驱动绑定

`platform_device.dev.bus_id`是设备的真正名称。它由两部分组成：

*   `platform_device.name` ... 用于驱动匹配。
*   `platform_device.id` ... 设备实例化号码，置为"-1“表示仅有一个设备。

这两部分是连接在一起的，所以`name/id` "`serial/0`"表示了总线的`bus_id`是"`serial.0`"，"`serial/3`"表示了总线的`bus_id`是"`serial.3`"；他们都使用了名称为"`serial`"的平台驱动。而"`my_rtc/-1`"表示的`bus_id`是"`my_rtc`"(没有用id做后缀)，它使用了名称为"`my_rtc`"的平台驱动。

驱动绑定由驱动核心(driver core)自动执行，当在设备和驱动之间查找到匹配时会调用驱动的`probe()`函数。如果`probe()`调用成功，驱动和设备绑定成功。有如下三种不同的方法来查找匹配：

*   当一个设备注册后，总线的驱动就会被检测是否匹配。平台设备应当在系统启动的早期注册。
*   当一个驱动使用`platform_driver_register()`注册后，总线上的所有的未绑定设备都会被检测是否匹配。驱动通常会在系统启动晚期注册，或者通过模块加载。
*   使用`platform_driver_probe()`注册驱动与使用`platform_driver_register()`注册驱动差不多，唯一例外的是使用前者注册驱动后，如果之后有其他设备注册，该驱动不会被检测是否配。(这是可以的，因为这种接口仅用于没有热插拔功能的设备上)
