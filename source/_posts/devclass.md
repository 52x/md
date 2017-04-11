---
title: 驱动模型之设备类
tags:
categories:
  - Kernel
  - 内核文档
date: 2014-03-05 15:28:09
---

翻译原文链接：<http://m.blog.csdn.net/blog/guoshaobei/4927177>
内核文档：Documentation/driver-model/class.txt
<!--more-->

### 介绍

一个设备类描述了一类的设备，例如语音设备或者网络设备。下面是已定义的设备类:

<Insert List of Device Classes Here>

每个设备类定义了一套语法和设备遵循的编程接口。设备驱动就是为特定总线上的特定设备而完成的这套编程接口实现。

对于一个设备驻足在哪个总线上，设备类是不可知的。

### 编程接口

设备类的数据结构如下：
```c
typedef int (*devclass_add)(struct device *);
typedef void (*devclass_remove)(struct device *);

struct device_class {
    char            * name;
    rwlock_t        lock;
    u32            devnum;
    struct list_head    node;

    struct list_head    drivers;
    struct list_head    intf_list;

    struct driver_dir_entry    dir;
    struct driver_dir_entry    device_dir;
    struct driver_dir_entry    driver_dir;

    devclass_add        add_device;
    devclass_remove        remove_device;
};
```
一个典型的设备类类似如下定义：
```c
struct device_class input_devclass = {
    .name          = "input",
    .add_device    = input_add_device,
    .remove_device = input_remove_device,
};
```
每个设备类数据结构放在一个头文件里导出，因此它能够被驱动，扩展程序和接口使用。

内核里设备类的注册和注销函数接口如下所示：
```c
int devclass_register(struct device_class * cls);
void devclass_unregister(struct device_class * cls);
```

### 设备

当设备绑定到驱动上，它就会被添加到驱动所属的设备类里。在驱动模型问世之前，这个过程在驱动的`probe()`回调函数里实现；当有了驱动模型后，这个过程在`probe()`回调函数被调用后实现。

设备在设备类里会被枚举。当一个设备被加入到一个设备类里，设备类的`devnum`域就会做加操作，分配给这个设备。这个域从来不会做减操作，所以当设备从设备类里移除，重新添加时，它会获得一个不同的枚举值。

设备类被允许为设备创建一个类相关的数据结构，并将其保存到设备的`class_data`指针。

设备类里没有设备链表。每个驱动有一个它支持的设备链表。为了能够访问到设备类里的所有的设备，将会迭代设备类里的每个驱动的设备链表。

### 设备驱动

当设备驱动被注册到内核里时，它们会被加入到设备类里。一个驱动通过设备`struct device_driver::devclass`域来说明它属于哪个设备类。

### sysfs目录结构

在sysfs的顶层目录里，有个名为'`class`'的目录。

每个设备类在`class`目录里获得一个自己的目录，以及两个默认的子目录：
```
        class/
        `-- input
            |-- devices
            `-- drivers
```
注册到设备类里的驱动会在`drivers/`目录里获得一个指向实际`drivers`目录的符号链接（实际`drivers`目录在`bus`目录下）：
```
   class/
   `-- input
       |-- devices
       `-- drivers
           `-- usb:usb_mouse -> ../../../bus/drivers/usb_mouse/
```

每一个设备在`devices/`目录里获得一个指向实际驻扎的设备目录的符号链接：
```
   class/
   `-- input
       |-- devices
       |   `-- 1 -> ../../../root/pci0/00:1f.0/usb_bus/00:1f.2-1:0/
       `-- drivers
```

### 导出属性

```c
struct devclass_attribute {
        struct attribute        attr;
        ssize_t (*show)(struct device_class *, char * buf, size_t count, loff_t off);
        ssize_t (*store)(struct device_class *, const char * buf, size_t count, loff_t off);
};
```
设备类驱动使用`DEVCLASS_ATTR`宏导出属性，类似`devices`提供的`DEVICE_ATTR`宏。
例如，一个如下定义：
```c
static DEVCLASS_ATTR(debug,0644,show_debug,store_debug);
```
等价于如下声明：
```c
static devclass_attribute devclass_attr_debug;
```
总线驱动可以从设备类的sysfs目录里添加/删除一个属性，使用如下函数接口：
```c
int devclass_create_file(struct device_class *, struct devclass_attribute *);
void devclass_remove_file(struct device_class *, struct devclass_attribute *);
```
在上面的例子里，放置在设备类目录里的文件会被命名为'`debug`'。

### 接口

也许存在多个访问同一设备类下的同一设备的机制。设备接口(Device interfaces)就是用来描述这样的机制的。

当一个设备被加入到一个设备类里，内核尝试将它添加到设备类的每一个被注册过的接口里。
