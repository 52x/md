---
title: 驱动模型之总线类型
tags:
  - 设备模型
categories:
  - Kernel
  - 内核文档
date: 2014-03-05 11:59:27
---

翻译原文链接：<http://blog.chinaunix.net/uid-20522771-id-3452028.html>
内核文档： Documentation/driver-model/bus.txt
<!--more-->
### 定义

查阅 kerneldoc 以了解 `struct bus_type`。
```c
    int bus_register(struct bus_type * bus);
```

### 声明

内核中每个总线类型（PCI、USB 等等）都应该声明一个此类型的静态对象。它们必须初始化该对象的 `name` 字段，然后可选的初始化 `match` 回调函数。
```c
    struct bus_type pci_bus_type = {
        .name = "pci",
        .match = pci_bus_match,
    };
```

这个结构体应该在头文件中向驱动程序导出：
```c
    extern struct bus_type pci_bus_type;
```

### 注册

当初始化一个总线驱动时，将会调用` bus_register`。这时这个总线对象剩下的字段将被初始化，然后这个对象会被插入到总线类型的一个全局列表里去。一旦完成一个总线对象的注册，那么对于总线驱动来说它里面的字段就已经可用了。

### 回调函数

`match()`：给设备加载驱动

设备 ID 的结构和比较这些 ID 的语义本质上讲都是总线定制的。驱动通常会定义一个它所支持的设备 ID 的数组，并将这个数组驻留在总线定制的驱动结构中。

回调函数 `match` 的目的就是在不牺牲总线定制功能或类型安全的情况下，通过比较驱动所支持的 ID 与特定设备的 ID，给总线提供一个确定特定驱动是否支持特定设备的机会。

当一个驱动注册到总线后，将会遍历总线的设备列表，然后会为每个还没有和任何一个驱动关联的设备调用回调函数`match`。

### 设备和驱动列表

设备和驱动列表是为了取代那些总线自己保存的本地列表。它们分别是 `struct devices` 和 `struct device_drivers` 的列表。总线驱动可以随意的使用这些列表，但有可能需要将其转换成总线定制类型。

LDM 核心提供了遍历各个列表的辅助函数。
```c
    int bus_for_each_dev(struct bus_type * bus, struct device * start, void * data,
                         int (*fn)(struct device *, void *));

    int bus_for_each_drv(struct bus_type * bus, struct device_driver * start,
                         void * data, int (*fn)(struct device_driver *, void *));
```
这些辅助函数迭代各个列表，然后为这些列表里的设备或驱动调用回调函数。所有列表的访问都是通过取得总线锁（实时读取）而保持同步的。列表中每个对象的引用计数都会在调用回调函数前递增；并且会在取得下一个对象之后递减。总线锁并不在回调函数调用期间继续保持。

### sysfs

这里有一个顶层目录：‘`bus`’。

每个总线在 `bus` 目录下都有自己的目录，其中有两个默认目录：
```
    /sys/bus/pci/
    |-- devices
    `-- drivers
```

在总线注册的驱动会得到一个该总线 `drivers` 目录的子目录：
```
    /sys/bus/pci/
    |-- devices
    `-- drivers
       |-- Intel ICH
       |-- Intel ICH Joystick
       |-- agpgart
       `-- e100
```

在这种类型总线上发现的每个设备都会在总线的 `deivces` 目录下获得一个符号链接，该链接指向这个设备在物理层次中的目录。
```
    /sys/bus/pci/
    |-- devices
    |  |-- 00:00.0 -> ../../../root/pci0/00:00.0
    |  |-- 00:01.0 -> ../../../root/pci0/00:01.0
    |  `-- 00:02.0 -> ../../../root/pci0/00:02.0
    `-- drivers
```

### 属性导出

```c
    struct bus_attribute {
        struct attribute    attr;
        ssize_t (*show)(struct bus_type *, char * buf);
        ssize_t (*store)(struct bus_type *, const char * buf, size_t count);
    };
```
总线驱动可以使用宏 `BUS_ATTR` 导出属性，它的工作方式就像 `device` 里的 `DEVICE_ATTR` 一样。例如，像这样定义：
```c
    static BUS_ATTR(debug,0644,show_debug,store_debug);
```

这等于是定义一个这样的东西：
```c
    static bus_attribute bus_attr_debug;
```

使用以下函数，可以用来从总线的 sysfs 目录中添加或删除属性：
```c
    int bus_create_file(struct bus_type *, struct bus_attribute *);
    void bus_remove_file(struct bus_type *, struct bus_attribute *);
```
