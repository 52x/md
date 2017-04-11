title: Reactive Cocoa Tutorial [4] = 只取所需的Filters
date: 2014-04-19 09:30:28
tags: Reactive Cocoa Tutorial
---

#我是前言
这是[Reactive Cocoa Tutorial系列](http://blog.sunnyxx.com/tags/Reactive%20Cocoa%20Tutorial/)其中的一篇，[上一篇](http://blog.sunnyxx.com/2014/03/06/rac_3_racsignal/)简单介绍了RAC中最重要的`RACSignal`，下面几篇文章将主要从它的`Operations`下手，这也是工程中使用RAC的重点。从简到难，本篇文章先介绍RAC消息流的`过滤器-Filters`类别的相关方法。

<!--more-->

-----

#RAC中的Filters

##画个范围
一个Signal源可以产生一系列next值，但并非所有值都是需要的，具体的Subscriber可以选择在原有Signal上套用Filter操作来过滤掉不需要的值。  
我的定义：RAC中如果一个`Operation`将处理后的值集合是处理前值集合的`子集`，我们就可以把它归为`Filter`类型。  

当然通过之前介绍的基础操作完全可以自己拼出个想要的filter来，RAC为了方便使用已经实现了几个常用的filter，经过总结，这些filter大概可以分成两类：`next值过滤类型`和`起止点过滤类型`

##值过滤类型Filters
### - filter: (BOOL (^)(id value))
RAC中的filter同名方法`- filter:(BOOL (^)(id value))`，简单明了，将一个value用block做test，返回YES的才会通过，它的内部实现使用了`- flattenMap:`，将原来的`Signal`经过过滤转化成只返回过滤值的`Signal`，用法也不难理解：    

```
[[self.inputTextField.rac_textSignal filter:^BOOL(NSString *value) {
    return [value hasPrefix:@"sunny"];
}] subscribeNext:^(NSString *value) {
    NSLog(@"This value has prefix `sunny` : %@", value);
}];
```

此外，还有几个这个方法的衍生方法：   

### - ignore: (id)
忽略给定的值，注意，这里忽略的既可以是地址相同的对象，也可以是`- isEqual:`结果相同的值，也就是说自己写的Model对象可以通过重写`- isEqual:`方法来使`- ignore:`生效。常用的值的判断没有问题，如下：      


```
[[self.inputTextField.rac_textSignal ignore:@"sunny"] subscribeNext:^(NSString *value) {
    NSLog(@"`sunny` could never appear : %@", value);
}];
```

### - ignoreValues
这个比较极端，忽略所有值，只关心Signal结束，也就是只取`Comletion`和`Error`两个消息，中间所有值都丢弃。   
注意，这个操作应该出现在Signal有终止条件的的情况下，如`rac_textSignal`这样除`dealloc`外没有终止条件的Signal上就不太可能用到。   


### - distinctUntilChanged
也是一个**相当常用**的Filter（但它不是- filter:的衍生方法），它将这一次的值与上一次做比较，当相同时（也包括`- isEqual:`）被忽略掉。   
比如UI上一个Label绑定了一个值，根据值更新显示的内容:

```
RAC(self.label, text) = [RACObserve(self.user, username) distinctUntilChanged];
self.user.username = @"sunnyxx"; // 1st
self.user.username = @"sunnyxx"; // 2nd
self.user.username = @"sunnyxx"; // 3rd
```

如果不增加`distinctUntilChanged`的话对于连续的相同的输入值就会有不必要的处理，这个栗子只是简单的UI刷新，但遇到如写数据库，发网络请求的情况时，代价就不能购忽略了。  

所以，对于相同值可以忽略的情况，果断加上它吧。   



##起止点过滤类型
除了被动的当next值来的时候做判断，也可以主动的提前选择开始和结束条件，分为两种类型：
`take型（取）`和`skip型(跳)`

###- take: (NSUInteger)
从开始一共取N次的next值，不包括`Competion`和`Error`，如：   

```
[[[RACSignal createSignal:^RACDisposable *(id<RACSubscriber> subscriber) {
    [subscriber sendNext:@"1"];
    [subscriber sendNext:@"2"];
    [subscriber sendNext:@"3"];
    [subscriber sendCompleted];
    return nil;
}] take:2] subscribeNext:^(id x) {
    NSLog(@"only 1 and 2 will be print: %@", x);
}];
```

###- takeLast: (NSUInteger)
取最后N次的next值，注意，由于一开始不能知道这个Signal将有多少个next值，所以RAC实现它的方法是将所有next值都存起来，然后**原Signal完成时**再将后N个**依次**发送给接收者，但Error发生时依然是立刻发送的。 
###- takeUntil:(RACSignal *)
当给定的signal完成前一直取值。最简单的栗子就是`UITextField`的`rac_textSignal`的实现（删减版本）: 

```
- (RACSignal *)rac_textSignal {
	@weakify(self);
	return [[[[[RACSignal
		concat:[self rac_signalForControlEvents:UIControlEventEditingChanged]]
		map:^(UITextField *x) {
			return x.text;
		}]
		takeUntil:self.rac_willDeallocSignal] // bingo!
}
```  
也就是这个Signal一直到textField执行`dealloc`时才停止
   
###- takeUntilBlock:(BOOL (^)(id x))  
对于每个next值，运行block，当block返回YES时停止取值，如： 
  
```
[[self.inputTextField.rac_textSignal takeUntilBlock:^BOOL(NSString *value) {
    return [value isEqualToString:@"stop"];
}] subscribeNext:^(NSString *value) {
    NSLog(@"current value is not `stop`: %@", value);
}];
```

###- takeWhileBlock:(BOOL (^)(id x))
上面的反向逻辑，对于每个next值，block返回	YES时才取值


###- skip:(NSUInteger)   
从开始跳过N次的next值，简单的栗子：  

```
[[[RACSignal createSignal:^RACDisposable *(id<RACSubscriber> subscriber) {
    [subscriber sendNext:@"1"];
    [subscriber sendNext:@"2"];
    [subscriber sendNext:@"3"];
    [subscriber sendCompleted];
    return nil;
}] skip:1] subscribeNext:^(id x) {
    NSLog(@"only 2 and 3 will be print: %@", x);
}];
```
###- skipUntilBlock:(BOOL (^)(id x))  
和`- takeUntilBlock:`同理，一直跳，直到block为YES
###- skipWhileBlock:(BOOL (^)(id x)) 
和`- takeWhileBlock:`同理，一直跳，直到block为NO

-----

#总结
本章介绍了RAC中Filter类型的Operation，总结一下：

- 适用场景：需要一个next值集合的`子集`时
- Filter类型：值过滤型和起止点过滤型
- 值过滤型常用方法： `-filter:`，`-ignore:`，`-distinctUnitlChanged`
- 起止点过滤型常用方法：`take`系列和`skip`系列



#References

https://github.com/ReactiveCocoa/ReactiveCocoa/blob/master/Documentation/BasicOperators.md#filtering

---
原创文章，转载请注明源地址，[blog.sunnyxx.com](blog.sunnyxx.com)