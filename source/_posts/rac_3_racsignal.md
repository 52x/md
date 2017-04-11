title: Reactive Cocoa Tutorial [3] = RACSignal的巧克力工厂
date: 2014-03-06 22:45:43
tags: Reactive Cocoa Tutorial
---
Reactive Cocoa Tutorial 系列，转载请注明该文源地址 http://blog.sunnyxx.com/2014/03/06/rac_3_racsignal/  -- by sunnyxx

##Overview

　　上一篇介绍了函数式编程和`RACStream`，使得函数得以串联起来，而它的具体子类，也是RAC编程中最重要的部分，`RACSignal`就是使得算式得以逐步运算并使其有意义的关键所在，本节主要介绍`RACSignal`的机理，具体的使用放到接下来的几节。


<img src="http://pic.jschina.com.cn/0/12/03/96/12039600_602173.jpg" width="400px"/>


##巧克力工厂的运作模式

　　RACStream实现了一个嵌套函数的结构，如f(x) = f1(f2(f3(x)))，但好像是考试卷子上的一道题，没有人去做它，没得出个结果的话这道题是没有意义的。

<!--more-->

　　OK，现在起将这个事儿都比喻成一个巧克力工厂，f(x)的结果是一块巧克力，f1,f2,f3代表巧克力生产的几个步骤，如果这个工厂不开工，它是没有意义的。

　　再说RACSignal，引用RAC doc的描述：
　　
> “A signal, represented by the RACSignal class, is a push-driven
> stream.”

　　我觉得这个`push-driven`要想解释清楚，需要和RACSequence的`pull-driven`放在一起来看。在巧克力工厂，push-driven是“生产一个吃一个”，而pull-driven是“吃完一个才生产下一个”，对于工厂来说前者是主动模式：生产了巧克力就“push”给各个供销商，后者是被动模式：各个供销商过来“pull”产品时才给你现做巧克力。

###Status

　　所以，对于RACSigna的push-driven的生产模式，首先，当工厂发现没有供销商签合同准备要巧克力的时候，工厂当然没有必要开动生产；只要当有一个以上准备收货的经销商时，工厂才开动生产。这就是RACSignal的休眠（cold）和激活（hot）状态，也就是所谓的冷信号和热信号。一般情况下，一个RACSignal创建之后都处于cold状态，有人去subscribe才被激活。

###Event

　　RACSignal能产生且只能产生三种事件：next、completed，error。

　　next表示这个Signal产生了一个值（成功生产了一块巧克力）

　　completed表示Signal结束，结束信号只标志成功结束，不带值（一个批次的订单完成了）

　　error表示Signal中出现错误，立刻结束（一个机器坏了，生产线立刻停止运转）

　　工厂厂长存了所有供销商的QQ，每当发生上面三件事情的一件时，都用QQ挨个儿发消息告诉他们，于是供销商就能根据生产状态决定要做点什么。当订单完成或者失败后，厂长就会把这个供销商的QQ删了，以后发消息的时候也就没必要通知他了。

###Side Effects

　　RACSignal在被subscribe的时候可能会产生副作用，先举个官方的栗子：

```
__block int aNumber = 0;

// Signal that will have the side effect of incrementing `aNumber` block
// variable for each subscription before sending it.
RACSignal *aSignal = [RACSignal createSignal:^ RACDisposable * (id<RACSubscriber> subscriber) {
    aNumber++;
    [subscriber sendNext:@(aNumber)];
    [subscriber sendCompleted];
    return nil;
}];

// This will print "subscriber one: 1"
[aSignal subscribeNext:^(id x) {
    NSLog(@"subscriber one: %@", x);
}];

// This will print "subscriber two: 2"
[aSignal subscribeNext:^(id x) {
    NSLog(@"subscriber two: %@", x);
}];
```
　　上面的signal在作用域外部引用了一个int变量，同时在signal的运算过程中作为`next`事件的值返回，这就造成了所谓的`副作用`，因为第二个订阅者的订阅而影响了输出值。

　　我的理解来看，这个事儿做的就不太地道，一个正经的函数式编程中的函数是不应该因为进行了运算而导致后面运算的值不统一的。但对于实际应用的情况来看也到无可厚非，比如用户点击了“登录”按钮，编程时把登录这个业务写为一个login的RACSignal，当然，第一次调用登录和再点一次第二次调用登录的结果肯定不一样了。所以说RAC式编程减少了大部分对临时状态值的定义，但不是全部哦。

　　怎么办呢？我觉得最好的办法就是“约定”，RAC design guide里面介绍了对于一个signal的命名法则：

    Hot signals without side effects 最好使用property，如“textChanged”，不太理解什么情况用到这个，权当做一个静态的属性来看就行。
    Cold signals without side effects 使用名词类型的方法名，如“-currentText”，“currentModels”，同时表明了返回值是个啥（这个尤其得注意，RACSignal的next值是id类型，所以全得是靠约定才知道具体返回类型）
    Signals with side effects 这种就是像login一样有副作用的了，推荐使用动词类型的方法名，用对动词基本就能知道是不是有副作用了，比如“-loginSignal”和“-saveToFile”大概就知道前面一个很可能有副作用，后面一个多存几次文件应该没副作用

　　当然，也可以`multicast`一个event，使得某些特殊的情况来共享一个副作用，后面再具体讲，先一个官方的简单的栗子：

```
// This signal starts a new request on each subscription.
RACSignal *networkRequest = [RACSignal createSignal:^(id<RACSubscriber> subscriber) {
    AFHTTPRequestOperation *operation = [client
        HTTPRequestOperationWithRequest:request
        success:^(AFHTTPRequestOperation *operation, id response) {
            [subscriber sendNext:response];
            [subscriber sendCompleted];
        }
        failure:^(AFHTTPRequestOperation *operation, NSError *error) {
            [subscriber sendError:error];
        }];

    [client enqueueHTTPRequestOperation:operation];
    return [RACDisposable disposableWithBlock:^{
        [operation cancel];
    }];
}];

// Starts a single request, no matter how many subscriptions `connection.signal`
// gets. This is equivalent to the -replay operator, or similar to
// +startEagerlyWithScheduler:block:.
RACMulticastConnection *connection = [networkRequest multicast:[RACReplaySubject subject]];
[connection connect];

[connection.signal subscribeNext:^(id response) {
    NSLog(@"subscriber one: %@", response);
}];

[connection.signal subscribeNext:^(id response) {
    NSLog(@"subscriber two: %@", response);
}];
```
　　当地一个订阅者subscribeNext的时候触发了AFNetworkingOperation的创建和执行，开始网络请求，此时又来了个订阅者订阅这个Signal，按理说这个网络请求会被“副作用”，重新发一遍，但做了上面的处理之后，这两个订阅者接收到了同样的一个请求的内容。

###RACScheduler - 生产线

　　RACScheduler是RAC里面对线程的简单封装，事件可以在指定的scheduler上分发和执行，不特殊指定的话，事件的分发和执行都在一个默认的后台线程里面做，大多数情况也就不用动了，有一些特殊的signal必须在主线程调用，使用-deliverOn：可以切换调用的线程。

　　但值得特殊了解的事实是：

    However, RAC guarantees that no two signal events will ever arrive concurrently. While an event is being processed, no other events will be delivered. The senders of any other events will be forced to wait until the current event has been handled.

　　意思是订阅者执行时的block一定非并发执行，也就是说不会执行到一半被另一个线程进入，也意味着写subscribeXXX block的时候没必要做加锁处理了。

###巧克力的生产工艺

　　RACSignal的厂子建好了，运行的模式也都想好了，剩下的就是巧克力的加工工艺了。

　　有了RACStream的嵌套和组装的基础，RACSignal得以使用组件化的工艺来一步步的加工巧克力，从可可，牛奶，糖等原料，混合到这种巧克力适用的液态巧克力，过滤，提纯，冷却，夹心，压模，再到包装，一个巧克力就产出了。对于不同种类的巧克力，比如酒心巧克力，也不过是把其中的某个组件替换成注入酒心罢了。

　　RACSignal的生产组件，也就是它的各式各样的operation，一个具体业务逻辑的实现，其实也就是选择合适operation按合适的顺序组合起来。

　　还举那个用户在textFiled输入并显示到上面的label中的栗子:
```
RAC(self.outputLabel, text) = self.inputTextField.rac_textSignal;
```
　　现在需求变成“用户输入3个字母以上才输出到label，当不足3个时显示提示”，OK，好办：
```
RAC(self.outputLabel, text) = [[self.inputTextField.rac_textSignal
    startWith:@"key is >3"] /* startWith 一开始返回的初始值 */
    filter:^BOOL(NSString *value) {
        return value.length > 3; /* filter使满足条件的值才能传出 */
}];

```
　　需求又增加成“**当输入sunny时显示输入正确**”

```
RAC(self.outputLabel, text) = [[self.inputTextField.rac_textSignal
    startWith:@"key is >3"] // startWith 一开始返回的初始值
    filter:^BOOL(NSString *value) { // filter使满足条件的值才能传出
        return value.length > 3;
    }]
    map:(NSString *value) {　// map将一个值转化为另一个值输出
        return [value isEqualToString:@"sunny"] ? @"bingo!" : value;
    }];

```
　　可以看出，基本上一个业务逻辑经过分析后可以拆解成一个个小RACSignal的组合，也就像生产巧克力的一道道工艺了。上面的栗子慢慢感觉就像了一个简陋的输答案的框了。

###然后呢？

　　接下来的几节就具体介绍一下RACSignal的operation方法，RAC提供了很多操作方法，大概总结为几大类：过滤型、XXX型、XXX型，后面再慢慢道来。
