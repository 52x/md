---
title: platform_get_resource 函数实现细节
tags:
categories:
  - Kernel
  - 设备驱动
date: 2014-02-25 10:58:26
---

参考原文：<http://bbs.ednchina.com/BLOG_ARTICLE_3008325.HTM>

`platform_get_resource` 函数源码如下：  
1）新内核实现
```c
struct resource *platform_get_resource(struct platform_device *dev,
                                   unsigned int type, unsigned int num)
{
       int i;

       for (i = 0; i < dev->num_resources; i++) {
              struct resource *r = &dev->resource[i];
              if (type == resource_type(r) && num-- == 0)
                     return r;
       }
       return NULL;
}
EXPORT_SYMBOL_GPL(platform_get_resource);
```
2）旧内核实现：
```c
/**
 *      platform_get_resource - get a resource for a device
 *      @dev: platform device
 *      @type: resource type
 *      @num: resource index
 */
struct resource *
platform_get_resource(struct platform_device *dev, unsigned int type,
                      unsigned int num)
{
        int i;

        for (i = 0; i < dev->num_resources; i++) {
                struct resource *r = &dev->resource[i];

                if ((r->flags & (IORESOURCE_IO|IORESOURCE_MEM|
                                 IORESOURCE_IRQ|IORESOURCE_DMA))
                    == type)
                        if (num-- == 0)
                                return r;
        }
        return NULL;
}
EXPORT_SYMBOL_GPL(platform_get_resource);
```
<!--more-->

**函数分析：**
```c
struct resource *r = &dev->resource[i];
```
这行代码使得不管你是想获取哪一份资源都从第一份资源开始搜索。
```c
if (type == resource_type(r) && num-- == 0)
```
这行代码首先通过 `type == resource_type(r)` 判断当前这份资源的类型是否匹配，如果匹配则再通过 `num-- == 0` 判断是否是你要的，如果不匹配重新提取下一份资源而不会执行 `num-- == 0` 这一句代码。

通过以上两步就能定位到你要找的资源了，接着把资源返回即可。如果都不匹配就返回`NULL`。

**实例分析：**

下面通过一个例子来看看它是如何拿到设备资源的。  
设备资源如下：
```c
static struct resource s3c_buttons_resource[] = {
       [0]={
              .start = S3C24XX_PA_GPIO,
              .end   = S3C24XX_PA_GPIO + S3C24XX_SZ_GPIO - 1,
              .flags = IORESOURCE_MEM,
       },
       [1]={
              .start = IRQ_EINT8,
              .end   = IRQ_EINT8,
              .flags = IORESOURCE_IRQ,
       },
       [2]={
              .start = IRQ_EINT11,
              .end   = IRQ_EINT11,
              .flags = IORESOURCE_IRQ,
       },
       [3]={
              .start = IRQ_EINT13,
              .end   = IRQ_EINT13,
              .flags = IORESOURCE_IRQ,
       },
       [4]={
              .start = IRQ_EINT14,
              .end   = IRQ_EINT14,
              .flags = IORESOURCE_IRQ,
       },

       [5]={
              .start = IRQ_EINT15,
              .end   = IRQ_EINT15,
              .flags = IORESOURCE_IRQ,
       },
       [6]={
              .start = IRQ_EINT19,
              .end   = IRQ_EINT19,
              .flags = IORESOURCE_IRQ,
       }
};
```
驱动中通过下面代码拿到第一份资源：
```c
struct resource *res;
res = platform_get_resource(pdev, IORESOURCE_MEM, 0);
```
函数进入 `for` 里面，`i=0`，`num_resources=7`，拿出 `resource[0]` 资源。`resource_type(r)` 提取出该份资源的资源类型并与函数传递下来的资源类型进行比较，匹配。`Num=0` (这里先判断是否等于`0`再自减`1`) 符合要求，从而返回该资源。

获取剩下资源的代码如下：
```c
for(i=0; i<6; i++){
             buttons_irq = platform_get_resource(pdev,IORESOURCE_IRQ,i);
             if(buttons_irq == NULL){
                  dev_err(dev,"no irq resource specified\n");
                   ret = -ENOENT;
                   goto err_map;
              }
              button_irqs[i] = buttons_irq->start;
}
```
分析如下：  
For第一次循环：
```c
buttons_irq = platform_get_resource(pdev,IORESOURCE_IRQ,0);
```
在拿出第一份资源进行 `resource_type(r)` 判断资源类型时不符合(此时 `num-- == 0` 这句没有执行)，进而拿出第二份资源，此时 `i=1`，`num_resources=7`，`num` 传递下来为`0`，资源类型判断时候匹配，`num` 也等于 `0`，从而确定资源并返回。

For第二次循环：
```c
buttons_irq = platform_get_resource(pdev,IORESOURCE_IRQ,1);
```
拿出第二份资源的时候 `resource_type(r)` 资源类型匹配，但是 `num` 传递下来时候为 `1`，执行 `num-- == 0` 时不符合(但 `num` 开始自减 `1`，这导致拿出第三份资源时 `num==0`)，只好拿出第三份资源。剩下的以此类推。

总结：
```c
struct resource *platform_get_resource(struct platform_device *dev,
                                   unsigned int type, unsigned int num)
```
`unsigned int type` 决定资源的类型，`unsigned int num` 决定`type`类型的第几份资源（<span style="color:red;">从`0`开始</span>）。即使同类型资源在资源数组中不是连续排放也可以定位得到该资源。  
比如第一份 `IORESOURCE_IRQ` 类型资源在 `resource[2]`，而第二份在 `resource[5]`，那
```c
platform_get_resource(pdev,IORESOURCE_IRQ,0);
```
可以定位第一份`IORESOURCE_IRQ`资源；
```c
platform_get_resource(pdev,IORESOURCE_IRQ,1);
```
可以定位第二份 `IORESOURCE_IRQ` 资源。

之所以能定位到资源，在于函数实现中的这一行代码：
```c
if (type == resource_type(r) && num-- == 0)
```
该行代码，如果没有匹配资源类型，`num-- == 0` 不会执行而重新提取下一份资源，只有资源匹配了才会寻找该类型的第几份资源，即使这些资源排放不连续。

<span style="color:red;">`num` 代表的不是 `resource` 数组的数组号，而是具有相同资源类型的序号 (从`0`开始的序号)。</span>

同类函数：  
**platform_get_irq**
```c
/**
 *      platform_get_irq - get an IRQ for a device
 *      @dev: platform device
 *      @num: IRQ number index
 */
int platform_get_irq(struct platform_device *dev, unsigned int num)
{
        struct resource *r = platform_get_resource(dev, IORESOURCE_IRQ, num);

        return r ? r->start : -ENXIO;
}
EXPORT_SYMBOL_GPL(platform_get_irq);
```
**platform_get_resource_byname**
```c
/**
 *      platform_get_resource_byname - get a resource for a device by name
 *      @dev: platform device
 *      @type: resource type
 *      @name: resource name
 */
struct resource *
platform_get_resource_byname(struct platform_device *dev, unsigned int type,
                      char *name)
{
        int i;

        for (i = 0; i < dev->num_resources; i++) {
                struct resource *r = &dev->resource[i];

                if ((r->flags & (IORESOURCE_IO|IORESOURCE_MEM|
                                 IORESOURCE_IRQ|IORESOURCE_DMA)) == type)
                        if (!strcmp(r->name, name))
                                return r;
        }
        return NULL;
}
EXPORT_SYMBOL_GPL(platform_get_resource_byname);
```
**platform_get_irq_byname**
```c
/**
 *      platform_get_irq_byname - get an IRQ for a device
 *      @dev: platform device
 *      @name: IRQ name
 */
int platform_get_irq_byname(struct platform_device *dev, char *name)
{
        struct resource *r = platform_get_resource_byname(dev, IORESOURCE_IRQ, name);

        return r ? r->start : -ENXIO;
}
EXPORT_SYMBOL_GPL(platform_get_irq_byname);
```
