---
title: linux设备模型
tags:
  - 设备模型
categories:
  - Kernel
  - 内核修炼之道
date: 2014-04-17 14:17:28
---

参考原文：
[《Linux内核修炼之道》精华分享与讨论（11）——设备模型（上） ](http://blog.csdn.net/fudan_abc/article/details/5410486)
[《Linux内核修炼之道》精华分享与讨论（12）——设备模型（下） ](http://blog.csdn.net/fudan_abc/article/details/5417879)

对于驱动开发来说，设备模型的理解是根本，毫不夸张得说，理解了设备模型，再去看那些五花八门的驱动程序，你会发现自己站在了另一个高度，从而有了一种俯视的感觉。
顾名而思义，设备模型是关于设备的模型，对咱们写驱动的和不写驱动的人来说，设备的概念就是总线和与其相连的各种设备。电脑城的IT工作者都会知道设备是通过总线连到计算机上的，而且还需要对应的驱动才能用，可是总线是如何发现设备的，设备又是如何和驱动对应起来的，它们经过怎样的艰辛才找到命里注定的那个他，它们的关系如何，白头偕老型的还是朝三暮四型的，这些问题就不是他们关心的了，而是咱们需要关心的。这些疑问的中心思想中心词汇就是总线、设备和驱动，没错，它们就是咱们这里要聊的Linux设备模型的名角。
<!--more-->
总线、设备、驱动，也就是bus、device、driver，既然是名角，在内核里都会有它们自己专属的结构，在include/linux/device.h里定义。
```c
struct bus_type {
  const char              * name;
    struct module           * owner;

     struct kset             subsys;
     struct kset             drivers;
     struct kset             devices;
     struct klist            klist_devices;
     struct klist            klist_drivers;

     struct blocking_notifier_head bus_notifier;

     struct bus_attribute    * bus_attrs;
     struct device_attribute * dev_attrs;
     struct driver_attribute * drv_attrs;
     struct bus_attribute drivers_autoprobe_attr;
     struct bus_attribute drivers_probe_attr;

     int (*match)(struct device * dev, struct device_driver * drv);
     int (*uevent)(struct device *dev, char **envp,
     int num_envp, char *buffer, int buffer_size);
     int             (*probe)(struct device * dev);
     int             (*remove)(struct device * dev);
     void            (*shutdown)(struct device * dev);

     int (*suspend)(struct device * dev, pm_message_t state);
     int (*suspend_late)(struct device * dev, pm_message_t state);
     int (*resume_early)(struct device * dev);
     int (*resume)(struct device * dev);

     unsigned int drivers_autoprobe:1;
};

struct device_driver {
    const char              * name;
     struct bus_type         * bus;

     struct kobject          kobj;
     struct klist            klist_devices;
     struct klist_node       knode_bus;

     struct module           * owner;
     const char              * mod_name;  /* used for built-in modules */
     struct module_kobject   * mkobj;

    int     (*probe)        (struct device * dev);
    int     (*remove)       (struct device * dev);
    void    (*shutdown)     (struct device * dev);
    int     (*suspend)      (struct device * dev, pm_message_t state);
    int     (*resume)       (struct device * dev);
};

struct device {
  struct klist  klist_children;
  struct klist_node knode_parent;  /* node in sibling list */
  struct klist_node knode_driver;
  struct klist_node knode_bus;
  struct device  *parent;

  struct kobject kobj;
  char bus_id[BUS_ID_SIZE]; /* position on parent bus */
  struct device_type *type;
  unsigned  is_registered:1;
  unsigned  uevent_suppress:1;

  struct semaphore sem; /* semaphore to synchronize calls to
        * its driver.
        */

  struct bus_type * bus;  /* type of bus device is on */
  struct device_driver *driver; /* which driver has allocated this
         device */
  void  *driver_data; /* data private to the driver */
  void  *platform_data; /* Platform specific data, device
         core doesn't touch it */
  struct dev_pm_info power;

#ifdef CONFIG_NUMA
  int  numa_node; /* NUMA node this device is close to */
#endif
  u64  *dma_mask; /* dma mask (if dma'able device) */
  u64  coherent_dma_mask;/* Like dma_mask, but for
           alloc_coherent mappings as
           not all hardware supports
           64 bit addresses for consistent
           allocations such descriptors. */

  struct list_head dma_pools; /* dma pools (if dma'ble) */

  struct dma_coherent_mem *dma_mem; /* internal for coherent mem
           override */
  /* arch specific additions */
  struct dev_archdata archdata;

  spinlock_t  devres_lock;
  struct list_head devres_head;

  /* class_device migration path */
  struct list_head node;
  struct class  *class;
  dev_t   devt;  /* dev_t, creates the sysfs "dev" */
  struct attribute_group **groups; /* optional groups */

  void (*release)(struct device * dev);
};
```
我知道进入了21世纪，最缺的就是耐性，房价股价都让咱们没有耐性，内核的代码也让人没有耐性。不过做为最没有耐性的一代人，还是要平心静气的扫一下上面的结构，我们会发现，`struct bus_type`中有成员`struct kset drivers`和`struct kset devices`，同时`struct device`中有两个成员`struct bus_type * bus`和`struct device_driver *driver`，`struct device_driver`中有两个成员`struct bus_type * bus`和`struct klist klist_devices`。先不说什么是`klist`、`kset`，光从成员的名字看，它们就是一个完美的三角关系。我们每个人心中是不是都有两个她？一个梦中的她，一个现实中的她。

凭一个男人的直觉，我们可以知道，`struct device`中的`bus`表示这个设备连到哪个总线上，`driver`表示这个设备的驱动是什么，`struct device_driver`中的`bus`表示这个驱动属于哪个总线，`klist_devices`表示这个驱动都支持哪些设备，因为这里`device`是复数，又是`list`，更因为一个驱动可以支持多个设备，而一个设备只能绑定一个驱动。当然，`struct
bus_type`中的`drivers`和`devices`分别表示了这个总线拥有哪些设备和哪些驱动。

单凭直觉，张钰红不了。我们还需要看看什么是`klist`、`kset`。还有上面`device`和`
driver`结构里出现的`kobject`结构是什么？作为一个五星红旗下长大的孩子，我可以肯定的告诉你，`kobject`和`kset`都是Linux设备模型中最基本的元素，总线、设备、驱动是西瓜，`kobjcet`、`kset`是种瓜的人，没有幕后种瓜人的汗水不会有清爽解渴的西瓜，我们不能光知道西瓜的的甜，还要知道种瓜人的辛苦。`kobject`和`kset`不会在意自己的得失，它们存在的意义在于把总线、设备和驱动这样的对象连接到设备模型上。种瓜的人也不会在意自己的汗水，在意的只是能不能送出甜蜜的西瓜。

一般来说应该这么理解，整个Linux的设备模型是一个OO的体系结构，总线、设备和驱动都是其中鲜活存在的对象，`kobject`是它们的基类，所实现的只是一些公共的接口，`kset`是同种类型`kobject`对象的集合，也可以说是对象的容器。只是因为C里不可能会有C++里类的class继承、组合等的概念，只有通过`kobject`嵌入到对象结构里来实现。这样，内核使用`kobject`将各个对象连接起来组成了一个分层的结构体系，就好像马列主义将我们13亿人也连接成了一个分层的社会体系一样。`kobject`结构里包含了`parent`成员，指向了另一个`kobject`结构，也就是这个分层结构的上一层结点。而`kset`是通过链表来实现的，这样就可以明白，`struct bus_type`结构中的成员`drivers`和`devices`表示了一条总线拥有两条链表，一条是设备链表，一条是驱动链表。我们知道了总线对应的数据结构，就可以找到这条总线关联了多少设备，又有哪些驱动来支持这类设备。

那么`klist`呢？其实它就包含了一个链表和一个自旋锁，我们暂且把它看成链表也无妨，本来在2.6.11内核里，`struct device_driver`结构的`devices`成员就是一个链表类型。这么一说，咱们上面的直觉都是正确的，如果买股票，摸彩票时直觉都这么管用，就不会有咱们这被压扁的一代了。

现在的人都知道，三角关系很难处。那么总线、设备和驱动之间是如何和谐共处那？先说说总线中的那两条链表是怎么形成的。内核要求每次出现一个设备就要向总线汇报，或者说注册，每次出现一个驱动，也要向总线汇报，或者说注册。比如系统初始化的时候，会扫描连接了哪些设备，并为每一个设备建立起一个`struct device`的变量，每一次有一个驱动程序，就要准备一个`struct device_driver`结构的变量。把这些变量统统加入相应的链表，`device`插入`devices`链表，`driver`插入`drivers`链表。这样通过总线就能找到每一个设备，每一个驱动。然而，假如计算机里只有设备却没有对应的驱动，那么设备无法工作。反过来，倘若只有驱动却没有设备，驱动也起不了任何作用。在他们遇见彼此之前，双方都如同路埂的野草，一个飘啊飘，一个摇啊摇，谁也不知道未来在哪里，只能在生命的风里飘摇。于是总线上的两张表里慢慢的就挂上了那许多孤单的灵魂。`devices`开始多了，`drivers`开始多了，他们像是来自两个世界，`devices`们彼此取暖，`drivers`们一起狂欢，但他们有一点是相同的，都只是在等待属于自己的那个另一半。

现在，总线上的两条链表已经有了，这个三角关系三个边已经有了两个，剩下的那个呢？链表里的设备和驱动又是如何联系呢？先有设备还是先有驱动？很久很久以前，在那激情燃烧的岁月里，先有的是设备，每一个要用的设备在计算机启动之前就已经插好了，插放在它应该在的位置上，然后计算机启动，然后操作系统开始初始化，总线开始扫描设备，每找到一个设备，就为其申请一个`struct device`结构，并且挂入总线中的`devices`链表中来，然后每一个驱动程序开始初始化，开始注册其`struct device_driver`结构，然后它去总线的`devices`链表中去寻找(遍历)，去寻找每一个还没有绑定驱动的设备，即`struct device`中的`struct device_driver`指针仍为空的设备，然后它会去观察这种设备的特征，看是否是他所支持的设备，如果是，那么调用一个叫做`device_bind_driver`的函数，然后他们就结为了秦晋之好。换句话说，把`struct device`中的`struct device_driver driver`指向这个驱动，而`struct device_driver driver`把`struct device`加入他的那张`struct klist klist_devices`链表中来。就这样，`bus`、`device`和`driver`，这三者之间或者说他们中的两两之间，就给联系上了。知道其中之一，就能找到另外两个。一荣俱荣，一损俱损。

但现在情况变了，在这红莲绽放的日子里，在这樱花伤逝的日子里，出现了一种新的名词，叫热插拔。设备可以在计算机启动以后在插入或者拔出计算机了。因此，很难再说是先有设备还是先有驱动了。因为都有可能。设备可以在任何时刻出现，而驱动也可以在任何时刻被加载，所以，出现的情况就是，每当一个`struct device`诞生，它就会去`bus`的`drivers`链表中寻找自己的另一半，反之，每当一个`struct device_driver`诞生，它就去`bus`的`devices`链表中寻找它的那些设备。如果找到了合适的，那么OK，和之前那种情况一样，调用`device_bind_driver`绑定好。如果找不到，没有关系，等待吧，等到昙花再开，等到风景看透，心中相信，这世界上总有一个人是你所等的，只是还没有遇到而已。


设备模型拍得再玄幻，它也只是个模型，必须得落实在具体的子系统，否则就只能抱着个最佳技术奖空遗恨。既然前面已经以USB子系统的实现分析示例了分析内核源码应该如何入手，那么这里就仍然以USB子系统为例，看看设备模型是如何软着陆的。

## 内核中USB子系统的结构

我们已经知道了USB子系统的代码都位于drivers/usb目录下面，也认识了一个很重要的目录——core子目录。现在，我们再来看一个很重要的模块 —— usbcore。

core就是核心，基本上你要在你的电脑里用USB设备，那么两个模块是必须的：一个是usbcore，这就是核心模块；另一个是主机控制器的驱动程序，比如ehci_hcd和uhci_hcd。你的USB设备要工作，合适的USB主机控制器模块也是必不可少的。

usbcore负责实现一些核心的功能，为别的设备驱动程序提供服务，提供一个用于访问和控制USB硬件的接口，而不用去考虑系统当前存在哪种主机控制器。至于core、主机控制器和USB驱动三者之间的关系，如下图所示。

{% asset_img usb_subsystem.png %}

USB驱动和主机控制器就像core的两个保镖，协议里也说了，主机控制器的驱动（HCD）必须位于USB软件的最下一层。HCD提供主机控制器硬件的抽象，隐藏硬件的细节，在主机控制器之下是物理的USB及所有与之连接的USB设备。而HCD只有一个客户，对一个人负责，就是usbcore。usbcore将用户的请求映射到相关的HCD，用户不能直接访问HCD。

core为咱们完成了大部分的工作，因此咱们写USB驱动的时候，只能调用core的接口，core会将咱们的请求发送给相应的HCD。

## USB子系统与设备模型

关于设备模型，最主要的问题就是，bus、device、driver是如何建立联系的？换言之，这三个数据结构中的指针是如何被赋值的？绝对不可能发生的事情是，一旦为一条总线申请了一个`struct bus_type`的数据结构之后，它就知道它的`devices`链表和`drivers`链表会包含哪些东西，这些东西一定不会是先天就有的，只能是后天填进来的。

具体到USB子系统，完成这个工作的就是USB core。USB core的代码会进行整个USB系统的初始化，比如申请`struct bus_type usb_bus_type`，然后会扫描USB总线，看线上连接了哪些USB设备，或者说Root Hub上连了哪些USB设备，比如说连了一个USB键盘，那么就为它准备一个`struct device`，根据它的实际情况，为这个`struct device`赋值，并插入`devices`链表中来。

又比如Root Hub上连了一个普通的Hub，那么除了要为这个Hub本身准备一个`struct device`以外，还得继续扫描看这个Hub上是否又连了别的设备，有的话继续重复之前的事情，这样一直进行下去，直到完成整个扫描，最终就把`usb_bus_type`中的`devices`链表给建立了起来。

那么`drivers`链表呢？这个就不用bus方面主动了，而该由每一个driver本身去bus上面登记，或者说挂牌。具体到USB子系统，每一个USB设备的驱动程序都会对应一个`struct usb_driver`结构，其中有一个`struct device_driver driver`成员，USB core为每一个设备驱动准备了一个函数，让它把自己的这个`struct device_driver driver`插入到`usb_bus_type`中的`drivers`链表中去。而这个函数正是我们此前看到的`usb_registeri`。而与之对应的`usb_deregister`所从事的正是与之相反的工作，把这个结构体从`drivers`链表中删除。

而`struct bus_type`结构的match函数在USB子系统里就是`usb_device_match`函数，它充当了一个红娘的角色，在USB总线的USB设备和USB驱动之间牵线搭桥，类似于交大BBS上的鹊桥版，虽然它们上面的条件都琳琅满目的，但明显这里match的条件不是那么的苛刻，要更为实际些。

可以说，USB core的确是用心良苦，为每一个USB设备驱动做足了功课，正因为如此，作为一个实际的USB设备驱动，它在初始化阶段所要做的事情就很少、很简单了，直接调用`usb_register`即可。事实上，没有人是理所当然应该为你做什么的，但USB core这么做了。所以每一个写USB设备驱动的人应该铭记，USB设备驱动绝不是一个人在工作，在他身后，是USB core所提供的默默无闻又不可或缺的支持。
