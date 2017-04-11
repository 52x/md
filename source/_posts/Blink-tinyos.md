title: Tinyos 编程入门
date: 2015/06/10
tags: [tinyos, Blink]

---
简要的说下 Tinyos 的背景：它是 UC Berkeley（加州大学伯克利分校）开发的开放源代码操作系统，专为嵌入式无线传感网络设计，操作系统基于构件（component-based）的架构使得快速的更新成为可能，而这又减小了受传感网络存储器限制的代码长度。在物联网如火如荼的今天，这个系统大有作为。它是嵌入式，轻量级。可以对无线传感器节点，SmartNode 节点等各种设备进行编程监控环境等状况，物联网必将渗透在生活的各个角落。

<!-- more -->

该系统中程序的开发是利用 Nesc 语言，该语言是的语法是类似于 C 语言，但是与 C 语言也有些区别。有 C 语言基础的稍微花几天时间就可以进行 Nesc 编程了。

对于该系统的更多背景了解和一些配置教程请点击[我](http://pan.baidu.com/s/1jGEe07O)。

###Nesc 中接口(interface)

NesC 程序主要由各式组件（component）构成，组件和组件之间通过特定的接口（interface）互相沟通。一个接口内声明了提供相关服务的方法。例如数据读取接口（Read）内就包含了读取（read）、读取结束（readDone）函数。接口只是制定了组件之间交流的规范，也就是通过某一个接口，只能通过该接口提供的方法实现两个组件之间的交流。但是接口终归只是接口，只是一组函数的声明，并未包含对接口的实现。例如以下便是读取接口的代码：

```
 interface Read<val_t> {
  
  command error_t read();

  
  event void readDone( error_t result, val_t val );
}
```

只有函数的声明，但是没有函数体，所以需要有一个组件来实现（implementation）这个接口。实现某一个接口的组件，称之为提供者（provider），而使用该接口进行通信的，称之为用户(user)。

接口内的函数分两类，一类为命令（command），另一类为事件（event）。用户可以调用某一组件提供的接口命令，然后等待相应的事件被触发。也就是说命令是有外部主动的被调用，而事件是被动的执行。简单的假设就是：组件 A 提供了 Read 接口以便其他组件与之对话，组件 B 调用组件 A 的 Read 接口的 read 命令来读取某一个数据，例如温度，然后等温度读取完毕之后，系统返回一个 readDone（读取结束）的事件给组件 B。

###Nesc 中的组件(component)

NesC 程序由组件构成。组件内主要是包含了对各类接口的使用（uses）和提供（provides）。例如组件 A 提供了Read 接口，那 A 就需要负责实现 Read 接口内的 read 命令，也就是 read 命令的函数体，即“具体这个值是如何读取出来的”。因为命令（command）是由接口的提供者（provider）负责实现的。如果组件 B 使用了 A 提供的Read 接口，那在读取数据结束以后，系统会返回给 B 一个“读取结束”的事件，而B则需要负责处理这个事件，即“数据读取完毕以后，我用这个数据干什么”，将值返回给计算机，或者是通过无线发送给其他传感器等等，所以事件（event）是由接口的使用者（user）来负责实现的。

组件分为两类。分别是模块(module)和配置(configuration)。

模块内包含了程序的主要逻辑实现代码，也就是对各类命令和事件的实现，是 NesC 程序的可执行代码的主体。而配置则是负责将各个模块，通过特定的接口连接起来，其本身并不负责实现任何特定的命令或者事件。

以 TinyOS 附带的 Blink（闪烁发光二极管）程序为例，

模块组件的代码如下：

```
 // BlinkC.nc
#include "Timer.h"

//模块组件
module BlinkC @safe()
{
  //声明使用的接口
  uses interface Timer<TMilli> as Timer0;
  uses interface Timer<TMilli> as Timer1;
  uses interface Timer<TMilli> as Timer2;
  uses interface Leds;
  uses interface Boot;
}
implementation
{
  //event时间必须在使用方中实现
  event void Boot.booted()
  {
    call Timer0.startPeriodic( 250 );
    call Timer1.startPeriodic( 500 );
    call Timer2.startPeriodic( 1000 );
  }

  event void Timer0.fired()
  {
    dbg("BlinkC", "Timer 0 fired @ %s.\n", sim_time_string());
    call Leds.led0Toggle();
  }
 
  event void Timer1.fired()
  {
    dbg("BlinkC", "Timer 1 fired @ %s \n", sim_time_string());
    call Leds.led1Toggle();
  }
 
  event void Timer2.fired()
  {
    dbg("BlinkC", "Timer 2 fired @ %s.\n", sim_time_string());
    call Leds.led2Toggle();
  }
}
```

配置组件的代码如下：

```
 代码
//BlinkAppC.nc
configuration BlinkAppC
{
}
implementation
{
components MainC, BlinkC, LedsC;
components new TimerMilliC() as Timer0;
components new TimerMilliC() as Timer1;
components new TimerMilliC() as Timer2;

//接口的使用方和提供方声明
BlinkC -> MainC.Boot;

BlinkC.Timer0 -> Timer0;
BlinkC.Timer1 -> Timer1;
BlinkC.Timer2 -> Timer2;
BlinkC.Leds -> LedsC;
}
```

Blink 程序由两个组件构成。BlinkC.nc 为模块，BlinkAppC.nc 为配置。

在模块 BlinkC 的声明内（module BlinkC {...}）内表明了该程序需要用到的全部接口。因为 Blink 程序的主要目的是将 TelosB 传感器上的三盏 LED 发光二极管以不同的频率闪烁。所以我们需要三个精度为毫秒（TMilli）的计时器接口（Timer），分辨使用as关键字重命名为 Timer0，Timer1 和 Timer2。既然需要点亮发光二极管，自然需要一个操控发光二极管的接口，也就是 Leds，最后就是程序启动负责初始化的接口 Boot。

接下去在实现部分（implementation {...}）。在实现部分需要实现所有我们用到的接口的事件，在这个程序里面，我们只是使用了接口，而作为这些接口的用户，我们只需要负责去实现他们的事件。这些接口内的命令，则由接口的提供者负责实现。
这里主要是两个事件，一个是 Boot 接口的 booted 事件，另一个是计时器被触发的 fired 事件。在 booted 事件中，也就是程序启动以后，我们的主要任务就一个，启动三个计时器：

```
call Timer0.startPeriodic( 250 );
call Timer1.startPeriodic( 500 );
call Timer2.startPeriodic( 1000 );
```

0号 Led 灯的频率为 4Hz，1 号 Led 灯的频率为 2Hz，2 号 Led 灯的频率为 1Hz。这里 startPeriodic 是一个启动计时器的命令，呼叫命令需要使用 call 关键字。同样，因为是命令，所以它们由接口的提供者负责实现，我们只负责使用就可以了。另一个需要我们处理的事件就是计时器的触发，因为有三个计时器，所以需要书写三个触发事件：

```
event void Timer0.fired()
{
dbg("BlinkC", "Timer 0 fired @ %s.\n", sim_time_string());
call Leds.led0Toggle();
}

event void Timer1.fired()
{
dbg("BlinkC", "Timer 1 fired @ %s \n", sim_time_string());
call Leds.led1Toggle();
}

event void Timer2.fired()
{
dbg("BlinkC", "Timer 2 fired @ %s.\n", sim_time_string());
call Leds.led2Toggle();
}
```

先不关心 dbg 的那一行。我们可以看到 0 号计时器触发的时候，我们切换 0 号发光二极管的状态（如果是亮的则熄灭，如果是灭的则点亮）；1 号计时器触发时则切换 1 号发光二极管；3 号计时器同理。同样的道理，led0Toggle，led1Toggle 和 led2Toggle 属于 Leds 接口的三个命令，只管用 call 调用使用便可。

接下去是看配置 BlinkAppC。这个组件本身并不使用或者提供任何接口，所以在其声明部分为空（configuration BlinkAppC{}）。而在其实现（implementation）部分则需要实现对组件的连接。因为 BlinkC 模块使用了 Boot、Leds 和 Timer接口，所以必须指明这些接口都是由其他哪些组件提供的。所以：

```
components MainC, BlinkC, LedsC;
components new TimerMilliC() as Timer0;
components new TimerMilliC() as Timer1;
components new TimerMilliC() as Timer2;

BlinkC -> MainC.Boot;

BlinkC.Timer0 -> Timer0;
BlinkC.Timer1 -> Timer1;
BlinkC.Timer2 -> Timer2;
BlinkC.Leds -> LedsC;
```

先使用 component 关键字标明，这个程序当中，总共要用到哪几个组件。其中包括我们自己编写的 BlinkC 模块。还有负责提供 Boot 接口的 MainC 组件，负责提供 Leds 接口的 LedsC 组件。还有提供 Timer 接口的TimerMilliC，其属于泛型（generic）配置，支持被实例化。这里先不细说，因为我们需要用到三个计时器，所以需要使用 new 关键字创建三个计时器的实例，然后分别用 as 被重命名为 Timer0、Timer1 和 Timer2。 
 
再往下就是组件之间的连接了。BlinkC 使用了 Boot 接口，而 MainC 正好提供了 BlinkC 所需的 Boot 接口，所以我们将他们进行连接。箭头所指方向为从使用者指向提供者。

```
BlinkC->MainC.Boot
// 或者像下面这样也是可以的。
MainC.Boot<-BlinkC
```

因为 BlinkC 内部就使用了一个 Boot 接口，所以 BlinkC 后面的 Boot 被省略了。完整的书写格式为：

```
// 意为：Blink组件内使用的Boot接口由MainC组件提供。
BlinkC.Boot->Mainc.Boot
```

接着是控制发光二极管的 Leds 接口，由 LedsC 组件提供。这里也进行了简写，完整的书写格式为：

```
BlinkC.Leds->LedsC.Leds
```

计数器的连接同理。
通过 Blink 程序，可以帮助我们理解 NesC 程序的构成和编程思路。这理解当然还有很多其他的技巧。

