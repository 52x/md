---
title: platform总线
tags:
categories:
  - Kernel
  - 设备驱动
date: 2014-03-02 22:54:37
---

原文链接：<http://blog.chinaunix.net/uid-20543183-id-1930814.html>

## 前言

`platform`总线是kernel中最近加入的一种虚拟总线.在近版的2.6kernel中,很多驱动都用`platform`改写了.只有在分析完`platform`总线之后,才能继续深入下去分析.在分析完`sysfs`和设备驱动模型之后,这部份应该很简单了.闲言少叙.步入正题.GO.GO!以下的源代码分析是基于2.6.25的.
<!--more-->

## platform概貌

在分析源代码之前,先在内核代码中找一个`platform`架构的驱动程序.下面以`i8042`芯片的驱动为例进行分析.  
在`linux-2.6.25/drivers/input/serio/i8042.c`的`intel 8042`的初始化入口中,有以下代码分段:
```c
static int __init i8042_init(void)
{
     ……

  err = platform_driver_register(&i8042_driver);
  if (err)
       goto err_platform_exit;

  i8042_platform_device = platform_device_alloc("i8042", -1);
     if (!i8042_platform_device) {
         err = -ENOMEM;
         goto err_unregister_driver;
     }

     err = platform_device_add(i8042_platform_device);
     if (err)
         goto err_free_device;
     ……
}
```
我们在上面的程序片段中看到,驱动程序先注册了一个`platform driver`,然后又添加了一个`platform device`.这里就涉及到了`platform`的两个最主要的操作：一个设备驱动注册，一个设备注册。

要了解`platform`总线的来龙去脉.得从它的初始化开始.

## platform初始化

`platform`总线的初始化是在`linux-2.6.25/drivers/base/platform.c`中的`platform_bus_init()`完成的,代码如下:
```c
int __init platform_bus_init(void)
{
     int error;

     error = device_register(&platform_bus);
     if (error)
         return error;

     error =  bus_register(&platform_bus_type);
     if (error)
         device_unregister(&platform_bus);
     return error;
}
```
上面代码中调用的子函数在{% post_link probe-of-linux-kernel-driver-module %}中已经分析过了。这段初始化代码创建了一个名为 “`platform`”的设备，后续`platform`的设备都会以此为`parent`。在`sysfs`中表示为：所有`platform`类型的设备都会添加在`platform_bus`所代表的目录下，即 `/sys/devices/platform` 下面.  
接着,这段初始化代码又创建了一个名为“`platform`”的总线.  
`platform_bus_type`的定义如下:
```c
struct bus_type platform_bus_type = {
     .name         = "platform",
     .dev_attrs    = platform_dev_attrs,
     .match        = platform_match,
     .uevent       = platform_uevent,
     .suspend      = platform_suspend,
     .suspend_late = platform_suspend_late,
     .resume_early = platform_resume_early,
     .resume       = platform_resume,
};
```
我们知道，在`bus_type`中包含了诸如设备与驱动匹配，`hotplug`事件等很多重要的操作。这些操作在分析`platform`设备注册与`platform`驱动注册的时候依次分析。

## platform device注册

在`intel 8042`的驱动代码中,我们看到注册一个`platform device`分为了两部分,一部份是创建一个`platform device`结构,另一部份是将其注册到总线中.先来看第一个接口.
```c
struct platform_device *platform_device_alloc(const char *name, int id)
{
     struct platform_object *pa;

     pa = kzalloc(sizeof(struct platform_object) + strlen(name), GFP_KERNEL);
     if (pa) {
         strcpy(pa->name, name);
         pa->pdev.name = pa->name;
         pa->pdev.id = id;
         device_initialize(&pa->pdev.dev);

         pa->pdev.dev.release = platform_device_release;
     }

     return pa ? &pa->pdev : NULL;
}
```
这段代码主要初始化了封装在`struct platform_object`中的`struct device`.

`struct platform_object`结构定义如下:
```c
struct platform_object {
     struct platform_device pdev;
     char name[1];
};
```
在定义中,定义`name`长度为`1`,所以,才有了`platform_device_alloc()`中分配`struct platform_object`的时候多加了名称的长度:
```c
pa = kzalloc(sizeof(struct platform_object) + strlen(name), GFP_KERNEL);
```
`struct device`结构如下:
```c
struct platform_device {
     const char    * name;
     int      id;
     struct device dev;
     u32      num_resources;
     struct resource    * resource;
};
```
在这个结构里封装了`struct device`、`struct resource`，`struct resource`这个结构在此之前没有接触过,这个结构表示设备所拥有的资源，即I/O端口或者是I/O映射内存。

`platform_device_add()`代码分段分析如下:
```c
int platform_device_add(struct platform_device *pdev)
{
     int i, ret = 0;

     if (!pdev)
         return -EINVAL;

     if (!pdev->dev.parent)
         pdev->dev.parent = &platform_bus;

     pdev->dev.bus = &platform_bus_type;
```
初始化设备的`parent`为`platform_bus`, 初始化驱备的总线为`platform_bus_type`. 回忆`platform`初始化的时候, 这两个东东这里终于派上用场了.
```c
     if (pdev->id != -1)
         snprintf(pdev->dev.bus_id, BUS_ID_SIZE, "%s.%d", pdev->name,
               pdev->id);
     else
         strlcpy(pdev->dev.bus_id, pdev->name, BUS_ID_SIZE);
```
设置设备`struct device`的`bus_id`成员, 留心这个地方, 在以后还需要用到这个的.
```c
     for (i = 0; i < pdev->num_resources; i++) {
         struct resource *p, *r = &pdev->resource[i];

         if (r->name == NULL)
              r->name = pdev->dev.bus_id;

         p = r->parent;
         if (!p) {
              if (r->flags & IORESOURCE_MEM)
                   p = &iomem_resource;
              else if (r->flags & IORESOURCE_IO)
                   p = &ioport_resource;
         }

         if (p && insert_resource(p, r)) {
              printk(KERN_ERR
                     "%s: failed to claim resource %d\n",
                     pdev->dev.bus_id, i);
              ret = -EBUSY;
              goto failed;
         }
     }
```
如果设备指定了它所拥有的资源, 将这些资源分配给它. 如果分配失败, 则失败退出. 从代码中可以看出, 如果`struct resource`的`flags`域被指定为`IORESOURCE_MEM`, 则所表示的资源为I/O映射内存; 如果指定为`IORESOURCE_IO`, 则所表示的资源为I/O端口.
```c
     pr_debug("Registering platform device '%s'. Parent at %s\n",
          pdev->dev.bus_id, pdev->dev.parent->bus_id);

     ret = device_add(&pdev->dev);
     if (ret == 0)
         return ret;

 failed:
     while (--i >= 0)
         if (pdev->resource[i].flags & (IORESOURCE_MEM|IORESOURCE_IO))
              release_resource(&pdev->resource[i]);
     return ret;
}
```
`device_add()`已经很熟悉了吧, 没错, 它就是将设备注册到指定的`bus_type`.

在分析linux设备模型的时候, 曾说过, 调用`device_add()`会产生一个`hotplug`事件. `platform device`的`hotplug`与一般的`device`事件相比, 它还要它所属`bus_type`的`uevent()`.对这个流程不熟悉的可以参照{% post_link probe-of-linux-kernel-driver-module %}. `platform device`所属的`bus_type`为`platform_bus_type`, 它的`uevent()`接口代码如下:
```c
static int platform_uevent(struct device *dev, struct kobj_uevent_env *env)
{
     struct platform_device *pdev = to_platform_device(dev);

     add_uevent_var(env, "MODALIAS=platform:%s", pdev->name);
     return 0;
}
```
上面的代码很简单,它就是在环境变量中添加一项`MODALIAS`.

## platform driver的注册

在`intel 8024`的驱动代码中看到，`platform driver`注册的接口为:
```c
int platform_driver_register(struct platform_driver *drv)
{
     drv->driver.bus = &platform_bus_type;
     if (drv->probe)
         drv->driver.probe = platform_drv_probe;
     if (drv->remove)
         drv->driver.remove = platform_drv_remove;
     if (drv->shutdown)
         drv->driver.shutdown = platform_drv_shutdown;
     if (drv->suspend)
         drv->driver.suspend = platform_drv_suspend;
     if (drv->resume)
         drv->driver.resume = platform_drv_resume;

     return driver_register(&drv->driver);
}
```
`struct platform_driver`主要封装了`struct device_driver`结构, 如下:
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
在这个接口里, 它指定`platform driver`的所属总线. 如果在`struct platform_driver`指定了各项接口的操作, 就会为`struct device_driver`中的相应接口赋值.

不要被上面的`platform_drv_XXX`吓倒了, 它们其实很简单, 就是将`struct device`转换为`struct platform_device`和`struct platform_driver`, 然后调用`platform_driver`中的相应接口函数.

最后, `platform_driver_register()`将驱动注册到总线上.  
这样, 总线上有设备, 又有驱动, 就会进行设备与驱动匹配的过程, 调用的相应接口为：  
`bus ->match --- > bus->probe/driver->probe` (如果总线的`probe`操作不存在,就会调用设备的`probe`接口)。  
这样，我们又回到`platform_bus_type`中的各项操作了。  
对应的`match`接口如下:
```c
static int platform_match(struct device *dev, struct device_driver *drv)
{
     struct platform_device *pdev;

     pdev = container_of(dev, struct platform_device, dev);
     return (strncmp(pdev->name, drv->name, BUS_ID_SIZE) == 0);
}
```
从代码中看出，如果要匹配成功，那么设备与驱动的名称必须要一致。

`platform_bus_type` 中没有指定`probe`接口，所以，会转至驱动的`probe`接口。在`platform_driver_register()`中，将其`probe`接口指定为了`platform_drv_probe`，这个函数又会将操作回溯到`struct platform_driver`的`probe`接口中。

到这里，`platform driver`的注册过程就会析完了。

## 小结

在本小节里，我们以linux设备模型为基础，分析虚拟总线`platform`的架构，对其`hotplug`事件，以及设备与驱动的匹配过程做了非常详细的分析，这部份知识是我们深入理解具体设备驱动的基础.
