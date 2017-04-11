---
title: 驱动模型之设备结构
tags:
  - 设备模型
categories:
  - Kernel
  - 内核文档
date: 2014-03-05 13:10:52
---

翻译原文链接：<http://blog.chinaunix.net/uid-20522771-id-3447322.html>
内核文档：Documentation/driver-model/device.txt
<!--more-->

### 基本设备结构体

查阅 kerneldoc 以了解 `struct device`。

### 编程接口

那些发现了设备的总线驱动使用以下接口将这个设备注册到核心：
```c
    int device_register(struct device * dev);
```

总线应该初始化该设备的下列字段：

*   parent
*   name
*   bus_id
*   bus

当一个设备的引用计数减到 0 时，则从核心中移除该设备。可以使用以下接口改变一个设备的引用计数。
```c
    struct device * get_device(struct device * dev);
    void put_device(struct device * dev);
```

如果 `device` 的引用计数不是 0（已经在移除的过程中），`get_device()` 会返回一个指向传递给它的`struct device`的指针。

一个 `driver` 可以使用以下接口访问 `device` 结构体中的`lock`：
```c
    void lock_device(struct device * dev);
    void unlock_device(struct device * dev);
```

### 属性

```c
    struct device_attribute {
        struct attribute    attr;
        ssize_t (*show)(struct device *dev, struct device_attribute *attr,
                        char *buf);
        ssize_t (*store)(struct device *dev, struct device_attribute *attr,
                         const char *buf, size_t count);
    };
```

设备驱动程序可以通过 sysfs 导出设备的属性。

请参考 Documentation/filesystems/sysfs.txt 来了解 sysfs 如何工作。

正如 Documentation/kobject.txt 中所述，设备属性的创建必须在`KOBJ_ADD`
uevent 发生之前。实现这个的唯一途径就是定义一个属性组。

定义属性可以使用宏函数`DEVICE_ATTR`。
```c
    #define DEVICE_ATTR(name,mode,show,store)
```

示例：
```c
    static DEVICE_ATTR(type, 0444, show_type, NULL);
    static DEVICE_ATTR(power, 0644, show_power, store_power);
```
这将定义两个拥有各自名称的 `struct device_attribute` 类型的结构体，'`dev_attr_type`'
和'`dev_attr_power`'。这两个属性可以象这样组织到一块：
```c
    static struct attribute *dev_attrs[] = {
        &dev_attr_type.attr,
        &dev_attr_power.attr,
        NULL,
    };

    static struct attribute_group dev_attr_group = {
        .attrs = dev_attrs,
    };

    static const struct attribute_group *dev_attr_groups[] = {
        &dev_attr_group,
        NULL,
    };
```

这些数组随后可以通过设置 `groups` 指针，在 `device_register()` 被调用之前关联到一个 `device` 上：
```c
    dev->groups = dev_attr_groups;
    device_register(dev);
```

`device_register()` 函数将使用‘`groups`‘指针去创建该设备的属性，而
`device_unregister()` 函数也将使用‘`groups`‘指针去移除该设备的属性。

提醒一句：尽管 kernel 允许在任何时候都可以在一个设备上调用 `device_create_file()` 和 `device_remove_file()`，但用户空间还是强烈期望能够得知在属性何时被创建。当一个新的设备注册进 kernel 时，就会产生一个 `uevent` 事件并通知用户空间该设备可用了。但如果设备的属性是在设备注册之后才添加的话，那么用户空间将得不到任何通知，对这个新的属性也将一无所知。

这对于那些需要在驱动探测（driver probe）时为设备发布附加属性的设备驱动程序来说是很重要的。如果设备驱动程序只是简单的传递 `device` 结构体来调用`device_create_file()`的话，用户空间就永远也得不到关于新属性的通知。
