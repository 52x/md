---
title: 驱动模型之设备驱动
tags:
  - 设备模型
categories:
  - Kernel
  - 内核文档
date: 2014-03-05 14:18:28
---

翻译原文链接：<http://blog.chinaunix.net/uid-20522771-id-3449546.html>
内核文档： Documentation/driver-model/driver.txt

<!--more-->
### 定义

查阅 kerneldoc 以了解 struct device_driver.

### 内存分配

设备驱动应该是静态分配的结构体。虽然系统中一个 driver 可能会支持多个设备，但`struct device_driver`仍将 driver 做为一个单独的整体（而不是某个设备实例的一部分）。

### 初始化

driver 至少应该初始化两个字段：`name` 和 `bus`。当然也应该初始化 `devclass` 字段，这样它可以在内部获得适当的链接。同时它也应该尽可能多的初始化回调函数，尽管这些回调函数是可选的。

### 声明

如上所述，`struct device_driver` 对象是静态分配的。以下是一个 `eepro100 driver` 的声明的小例子。这个声明仅仅是个假设，它依赖于被完全转换成新模型的 driver。
```c
    static struct device_driver eepro100_driver = {
        .name = "eepro100",
        .bus = &pci_bus_type,

        .probe = eepro100_probe,
        .remove = eepro100_remove,
        .suspend = eepro100_suspend,
        .resume = eepro100_resume,
    };
```

但大多数 driver 不能够被完全转换成新模型，这是因为这些 driver 所依附的总线都有一个该总线特定的（bus-specific）字段，而这些字段不能一概而论。

最常见的例子就是设备 ID 结构。通常一个驱动会定义一组它支持的设备 ID。而这些结构的数据格式以及与设备 ID 进行比较的算法都完全是总线特定的。因为将它们定义成总线特定的实体会牺牲类型安全，所以我们将保留总线特定的数据结构。

总线特定的（bus-specific）驱动应该包含一个通用的结构体 `struct device_driver`，就像这样：
```c
    struct pci_driver {
        const struct pci_device_id *id_table;
        struct device_driver driver;
    };
```

一个包含了总线特定字段的驱动应该像这样定义（还是用 eepro100 driver）：
```c
    static struct pci_driver eepro100_driver = {
        .id_table = eepro100_pci_tbl,
        .driver = {
            .name = "eepro100",
            .bus = &pci_bus_type,
            .probe = eepro100_probe,
            .remove = eepro100_remove,
            .suspend = eepro100_suspend,
            .resume = eepro100_resume,
        },
    };
```

可能有些人会觉得这样的内嵌结构体有些别扭甚至比较丑陋。但目前为止，这是能够满足我们需要的最好的方法了...

### 注册

```c
    int driver_register(struct device_driver * drv);
```
驱动程序将会在启动时注册该结构体。对于那些没有总线特定字段的 driver（即没有一个总线特定的 driver 结构体），应该使用`driver_register`并传递给这个函数指向该驱动的`struct device_driver`对象的指针。

然而大部分 driver 拥有一个总线特定结构并需要一个像`pci_driver_register`这样的函数完成驱动在总线上的注册。

重要的是驱动程序应该尽早的注册他们的 driver 结构。在向核心注册的过程中，`struct device_driver`对象中的几个字段会被初始化，这些字段包括引用计数(reference count)和锁(lock)，因为设备模型核心或总线驱动假设上述字段总是可用的。

### 转换到总线驱动

定义一个封装函数，可以更容易的实现驱动到新模型的转换。driver 可以完全忽略通用结构并让总线封装函数去填充这些字段。对于那些回调函数，总线则可以定义一组通用回调函数用来指向 driver 的总线特定回调函数。

该解决方案只是暂时的。为了得到 driver 里的 `class` 信息，这些 driver 必须能够被修改。因为从 driver 到新模型的转换应当减少基础结构的复杂性和代码规模，所以建议添加 它们到 `class` 信息的转换。

### 访问

一旦该对象已被注册，它就可以访问对象的公共字段，比如锁(lock)和设备列表。
```c
    int driver_for_each_dev(struct device_driver * drv, void * data,
                            int (*callback)(struct device * dev, void * data));
```

`devices` 字段就是一组已经绑定到 driver 的设备。LDM 核心提供了一个辅助函数操作 driver 所控制的所有设备。这个辅助函数将在访问每个节点时锁定driver，并且会在访问每个设备时设置正确的引用计数。

### sysfs

当一个 driver 注册之后，就会在 sysfs 中它所属总线目录下创建一个目录。在这个目录中，driver 可以在全局范围内向用户空间导出驱动的控制接口，例如切换驱动程序中的 debug 输出。

这个目录的一个未来的特性就是它本身将是一个‘`devices`’目录。该目录将会包含它所支持的那些设备的目录符号链接。

### 回调

```c
    int (*probe)(struct device * dev);
```

`probe()` 将在总线的 `rwsem` 锁定的情况下在进程上下文中调用，然后 driver 将部分地与设备绑定。在`probe()`或其他例程中，driver 通常使用`container_of()`实现"`dev`"到总线特定类型的转换。该类型往往会提供设备的资源数据，就像`pci_dev.resource[]`或`platform_device.resources`，这些资源用于除了`dev->platform_data`以外的 driver 初始化。

这个回调包含了绑定该 driver 到给定设备的驱动特定的逻辑。这包括了验证该设备是否存在、该版本能否被处理、能否分配该 driver 的数据结构并初始化以及哪些硬件能被初始化。driver 通常会使用`dev_set_drvdata()`保存一个指向它的`state`的指针。当一个 driver 成功将自己绑定到设备时，`probe()`会返回0，然后驱动模型代码将会完成它这部分的 driver 到设备的绑定。

driver 的`probe()`函数可能返回负的`errno`值表示该 driver 没有被绑定到设备，这种情况下它必须释放所有已经成功分配的资源。
```c
    int (*remove)(struct device * dev);
```
调用 `remove` 来解除驱动(driver)和设备(device)的绑定。如果从系统中物理移除一个设备，或者驱动模块已经被卸载，或者在一个 `reboot` 时序中等等，这个 `remove` 都会被调用。

这都取决于这个 driver 判定该设备存在与否。它应该释放为这个设备而专门分配的资源，比如 `driver_data` 字段中的一切。

如果该设备仍然存在，它应该暂停该设备并将其置于能够支持的低功耗状态。
```c
    int (*suspend)(struct device * dev, pm_message_t state);
```
`suspend` 将设备置于在低功耗状态。

```c
    int (*resume)(struct device * dev);
```
`resume` 则用于把设备从低功耗状态中恢复回来。

### 属性

```c
    struct driver_attribute {
        struct attribute attr;
        ssize_t (*show)(struct device_driver *driver, char *buf);
        ssize_t (*store)(struct device_driver *, const char * buf, size_t count);
    };
```

设备驱动程序可以通过它们的 sysfs 目录将属性导出。可以使用宏 `DRIVER_ATTR` 定义属性，这个宏工作原理和 `DEVICE_ATTR` 是一样的。

示例：
```c
    DRIVER_ATTR(debug,0644,show_debug,store_debug);
```

这实际上就是定义一个这样的东西：
```c
    struct driver_attribute driver_attr_debug;
```

稍后就可以使用以下函数在 driver 的目录中添加或删除属性。
```c
    int driver_create_file(struct device_driver *, const struct driver_attribute *);
    void driver_remove_file(struct device_driver *, const struct driver_attribute *);
```
