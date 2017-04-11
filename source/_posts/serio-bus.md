---
title: serio总线
date: 2014-03-03 14:10:35
tags:
categories:
  - Kernel
  - 设备驱动
---

原文链接：<http://blog.chinaunix.net/uid-20543183-id-1930815.html>

### 前言

serio总线同之前分析的platform总线一样，也是一种虚拟总线。它是Serial I/O的缩写，表示串行的输入输出设备，很多输入输出设备都是以此为基础的。同前面几篇笔记一样，下面的代码分析是基于linux kernel 2.6.25版本。
<!--more-->

### serio总线的初始化

serio总线的初始化是在`linux2.6.25/drivers/input/serio/serio.c`中完成的。代码如下：
```c
static int __init serio_init(void)
{
     int error;

     error = bus_register(&serio_bus);
     if (error) {
         printk(KERN_ERR "serio: failed to register serio bus, error: %d\n", error);
         return error;
     }

     serio_task = kthread_run(serio_thread, NULL, "kseriod");
     if (IS_ERR(serio_task)) {
         bus_unregister(&serio_bus);
         error = PTR_ERR(serio_task);

         printk(KERN_ERR "serio: Failed to start kseriod, error: %d\n", error);
         return error;
     }

     return 0;
}
```
在这里，创建了对应`bus_type`为`serio_bus`的总线，还创建了一个名为“`kseriod`”的内核线程。我们暂且不管它是做什么的，等以后的代码涉及到再来进行分析。

### serio设备注册

serio设备对应的数据结构为`struct serio`, 对应的注册接口为：`serio_register_port()`。代码如下：
```c
static inline void serio_register_port(struct serio *serio)
{
     __serio_register_port(serio, THIS_MODULE);
}
```
它是`__serio_register_port()`的一个封装函数。代码如下：
```c
void __serio_register_port(struct serio *serio, struct module *owner)
{
     serio_init_port(serio);
     serio_queue_event(serio, owner, SERIO_REGISTER_PORT);
}
```
它先初始化一个serio设备，在`serio_init_port()`中，它指定了设备的总线类型为`serio_bus`。之后，调用`serio_queue_event()`。看这个函数的名称好像是产生了什么事件，来看一下它的代码：
```c
static int serio_queue_event(void *object, struct module *owner,
                   enum serio_event_type event_type)
{
     unsigned long flags;
     struct serio_event *event;
     int retval = 0;

     spin_lock_irqsave(&serio_event_lock, flags);

     /*
      * Scan event list for the other events for the same serio port,
      * starting with the most recent one. If event is the same we
      * do not need add new one. If event is of different type we
      * need to add this event and should not look further because
      * we need to preseve sequence of distinct events.
      */

     list_for_each_entry_reverse(event, &serio_event_list, node) {
         if (event->object == object) {
              if (event->type == event_type)
                   goto out;
              break;
         }
     }

     event = kmalloc(sizeof(struct serio_event), GFP_ATOMIC);
     if (!event) {
         printk(KERN_ERR
              "serio: Not enough memory to queue event %d\n",
              event_type);
         retval = -ENOMEM;
         goto out;
     }

     if (!try_module_get(owner)) {
         printk(KERN_WARNING
              "serio: Can't get module reference, dropping event %d\n",
              event_type);
         kfree(event);
         retval = -EINVAL;
         goto out;
     }

     event->type = event_type;
     event->object = object;
     event->owner = owner;

     list_add_tail(&event->node, &serio_event_list);
     wake_up(&serio_wait);
out:
     spin_unlock_irqrestore(&serio_event_lock, flags);
     return retval;
}
```
这个函数比较简单，就是根据参数信息生成了一个`struct serio_event`结构，再将此结构链接至`serio_event_list`末尾。

接着就要来寻找`serio_event_list`这个链表的处理。这个就是`serio_thread`的工作了，还记得初始化的时候所创建的线程么，不错，就是它。Kernel可能认为对serio的操作比较频繁，所以对一些操作事件化，将它以多线程处理。  
`serio_thread()`代码如下：
```c
static int serio_thread(void *nothing)
{
     set_freezable();
     do {
         serio_handle_event();
         wait_event_freezable(serio_wait,
              kthread_should_stop() || !list_empty(&serio_event_list));
     } while (!kthread_should_stop());

     printk(KERN_DEBUG "serio: kseriod exiting\n");
     return 0;
}
```
这个函数的核心处理是`serio_handle_event()`, 代码如下：
```c
static void serio_handle_event(void)
{
     struct serio_event *event;
     mutex_lock(&serio_mutex);

     /*
      * Note that we handle only one event here to give swsusp
      * a chance to freeze kseriod thread. Serio events should
      * be pretty rare so we are not concerned about taking
      * performance hit.
      */
     if ((event = serio_get_event())) {
         switch (event->type) {
              case SERIO_REGISTER_PORT:
                   serio_add_port(event->object);
                   break;
              case SERIO_RECONNECT_PORT:
                   serio_reconnect_port(event->object);
                   break;
              case SERIO_RESCAN_PORT:
                   serio_disconnect_port(event->object);
                   serio_find_driver(event->object);
                   break;
              case SERIO_ATTACH_DRIVER:
                   serio_attach_driver(event->object);
                   break;
              default:
                   break;
         }

         serio_remove_duplicate_events(event);
         serio_free_event(event);
     }

     mutex_unlock(&serio_mutex);
}
```
这个函数的流程大致是这样的：  
调用`serio_get_event()`从链表中取出`struct serio_event`元素，然后对这个元素的事件类型做不同的处理，处理完了之后，调用`serio_remove_duplicate_events()`在链表中删除相同请求的`event`.
对应之前的`serio_register_port()`函数，它产生的事件类型是`SERIO_REGISTER_PORT`。也就是说，对于注册serio设备来说，流程会转入`serio_add_port()`。
代码如下：
```c
static void serio_add_port(struct serio *serio)
{
     int error;

     if (serio->parent) {
         serio_pause_rx(serio->parent);
         serio->parent->child = serio;
         serio_continue_rx(serio->parent);
     }

     list_add_tail(&serio->node, &serio_list);
     if (serio->start)
         serio->start(serio);
     error = device_add(&serio->dev);
     if (error)
         printk(KERN_ERR
              "serio: device_add() failed for %s (%s), error: %d\n",
              serio->phys, serio->name, error);
     else {
         serio->registered = 1;

         error = sysfs_create_group(&serio->dev.kobj, &serio_id_attr_group);
         if (error)
              printk(KERN_ERR
                   "serio: sysfs_create_group() failed for %s (%s), error: %d\n",
                   serio->phys, serio->name, error);
     }
}
```
我们终于看到serio device注册的庐山真面目了，它会调用设备的`start()`函数，然后调用`device_add()`将设备注册到总线上。

同platform总线一样，这里的serio device注册的时候也会产生一个hotplug事件，对应就会调用总线的`uenvent`函数，`serio_bus`的`event`接口为：
```c
static int serio_uevent(struct device *dev, struct kobj_uevent_env *env)
{
     struct serio *serio;

     if (!dev)
         return -ENODEV;

     serio = to_serio_port(dev);

     SERIO_ADD_UEVENT_VAR("SERIO_TYPE=%02x", serio->id.type);
     SERIO_ADD_UEVENT_VAR("SERIO_PROTO=%02x", serio->id.proto);
     SERIO_ADD_UEVENT_VAR("SERIO_ID=%02x", serio->id.id);
     SERIO_ADD_UEVENT_VAR("SERIO_EXTRA=%02x", serio->id.extra);
     SERIO_ADD_UEVENT_VAR("MODALIAS=serio:ty%02Xpr%02Xid%02Xex%02X",
                   serio->id.type, serio->id.proto, serio->id.id, serio->id.extra);

     return 0;
}
```
可见, 会在hotplug的环境变量中添加几项值。记得我们之前分析设备驱动模型的时候，在总线下面的设备都有一个属性文件，这个文件的内容就是对应`bus`和`kset`所添加的环境变量。在`/sys`文件系统中就可以看到这此环境变量。做个测试：
```bash
[root@localhost serio0]# cat /sys/bus/serio/devices/serio0/uevent
DRIVER=atkbd
PHYSDEVBUS=serio
PHYSDEVDRIVER=atkbd
SERIO_TYPE=06
SERIO_PROTO=00
SERIO_ID=00
SERIO_EXTRA=00
MODALIAS=serio:ty06pr00id00ex00
```
### serio driver的注册

对应serio driver注册的接口`serio_register_driver()`。代码如下：
```c
static inline int serio_register_driver(struct serio_driver *drv)
{
     return __serio_register_driver(drv, THIS_MODULE, KBUILD_MODNAME);
}
```
它是`__serio_register_driver()`的封装函数，代码如下：
```c
int __serio_register_driver(struct serio_driver *drv, struct module *owner, const char *mod_name)
{
     int manual_bind = drv->manual_bind;
     int error;

     drv->driver.bus = &serio_bus;
     drv->driver.owner = owner;
     drv->driver.mod_name = mod_name;

     /*
      * Temporarily disable automatic binding because probing
      * takes long time and we are better off doing it in kseriod
      */
     drv->manual_bind = 1;

     error = driver_register(&drv->driver);
     if (error) {
         printk(KERN_ERR
              "serio: driver_register() failed for %s, error: %d\n",
              drv->driver.name, error);
         return error;
     }

     /*
      * Restore original bind mode and let kseriod bind the
      * driver to free ports
      */
     if (!manual_bind) {
         drv->manual_bind = 0;

         error = serio_queue_event(drv, NULL, SERIO_ATTACH_DRIVER);
         if (error) {
              driver_unregister(&drv->driver);
              return error;
         }
     }

     return 0;
}
```
上面这段代码比较简单，在注册驱动的时候，将驱动的总线指定为`serio_bus`，然后调用`driver_register()`将驱动注册到总线。如果`drv->manual_bind`不为`1`，还会产生一个`SERIO_ATTACH_DRIVER`事件。`drv->manual_bind`成员的含义应该是要手动进行驱动与设备的绑定。

在注册驱动的时候，会产生一次驱动与设备的匹配过程，这过程会调用`bus->match --> bus->probe`，看下serio总线是怎么样处理的。
`serio_bus.match`的函数如下：
```c
static int serio_bus_match(struct device *dev, struct device_driver *drv)
{
     struct serio *serio = to_serio_port(dev);
     struct serio_driver *serio_drv = to_serio_driver(drv);

     if (serio->manual_bind || serio_drv->manual_bind)
         return 0;

     return serio_match_port(serio_drv->id_table, serio);
}
```
如果驱动或者设备指定了手动绑定，那么这次绑定是不成功的，否则调用`serio_match_port()`进行更细致的判断。代码如下：
```c
static int serio_match_port(const struct serio_device_id *ids, struct serio *serio)
{
     while (ids->type || ids->proto) {
         if ((ids->type == SERIO_ANY || ids->type == serio->id.type) &&
             (ids->proto == SERIO_ANY || ids->proto == serio->id.proto) &&
             (ids->extra == SERIO_ANY || ids->extra == serio->id.extra) &&
             (ids->id == SERIO_ANY || ids->id == serio->id.id))
              return 1;
         ids++;
     }
     return 0;
}
```
由此看出，只有serio device信息与serio driver的`id_table`中的信息匹配的时候，才会将设备和驱动绑定起来。

`serio_bus.probe`的接口函数如下示：
```c
static int serio_driver_probe(struct device *dev)
{
     struct serio *serio = to_serio_port(dev);
     struct serio_driver *drv = to_serio_driver(dev->driver);

     return serio_connect_driver(serio, drv);
}

static int serio_connect_driver(struct serio *serio, struct serio_driver *drv)
{
     int retval;

     mutex_lock(&serio->drv_mutex);
     retval = drv->connect(serio, drv);
     mutex_unlock(&serio->drv_mutex);

     return retval;
}
```
即会调用设备驱动的`connect()`函数。

在注册serio driver的时候，还会产生`SERIO_ATTACH_DRIVER`事件，这个事件的处理是在`serio_handle_event()`中完成的。相应的处理接口为：
```c
static void serio_attach_driver(struct serio_driver *drv)
{
     int error;

     error = driver_attach(&drv->driver);
     if (error)
         printk(KERN_WARNING
              "serio: driver_attach() failed for %s with error %d\n",
              drv->driver.name, error);
}
```
可以看出，这里也是执行一次设备与驱动的匹配过程。

### serio 的中断处理函数分析

`serio_interrupt()`在serio bus构造的驱动也是一个常用的接口，这个接口用来处理serio 设备的中断。我们来看下它的代码：
```c
irqreturn_t serio_interrupt(struct serio *serio,
                   unsigned char data, unsigned int dfl)
{
         unsigned long flags;
         irqreturn_t ret = IRQ_NONE;

         spin_lock_irqsave(&serio->lock, flags);

         if (likely(serio->drv)) {
                ret = serio->drv->interrupt(serio, data, dfl);
         } else if (!dfl && serio->registered) {
                   serio_rescan(serio);
                   ret = IRQ_HANDLED;
         }

         spin_unlock_irqrestore(&serio->lock, flags);
         return ret;
}
```
首先，先判断当前设备是否已经关联到了驱动程序。如果已经被关联了，那么调用驱动的中断处理函数。如果没有，就会转入`serio_rescan()`中执行。该函数代码如下：
```c
void serio_rescan(struct serio *serio)
{
         serio_queue_event(serio, NULL, SERIO_RESCAN_PORT);
}
```
由此可见，它是执行了一个`SERIO_RESCAN_PORT`动作。同样，这个动作是在`serio_handle_event()`中处理的。相应的处理接口为：
```c
case SERIO_RESCAN_PORT:
        serio_disconnect_port(event->object);
        serio_find_driver(event->object);
        break;
```
首先调用`serio_disconnect_port()`解除serio设备与驱动绑定，然后调用`serio_find_driver()`重新执行一次设备与驱动的匹配过程。  
`serio_find_driver()` 代码如下：
```c
static void serio_find_driver(struct serio *serio)
{
	int error;

	error = device_attach(&serio->dev);
	if (error < 0)
		printk(KERN_WARNING
			"serio: device_attach() failed for %s (%s), error: %d\n",
			serio->phys, serio->name, error);
}
```
如果有合适的驱动，就会将之与设备关联起来。

### 小结

来小结一下，在serio总线中，注册一个serio设备时，除了将其注册到所属的`serio_bus`上，还会调用设备的`start()`函数。在中断处理的时候，如果serio设备已经关联到驱动，则调用驱动的`interrupt`函数。如果没有关联，则重新匹配驱动并将其注册到总线。  
总的说来，serio 总线的代码很简单。到这里，我们已经分析过platform, serio两种虚拟总线，应该结合这两个虚拟总线的例子好好体会一下结构封装的技巧。
