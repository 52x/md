title: ios程序员6级考试（答案和解释）
date: 2014-03-06 23:10:58
tags: ios6级考试
---
# 我是前言
上次发了个[ios程序员6级考试题](http://blog.sunnyxx.com/2014/03/06/ios_exam_0/) ，还在不断补充中，开个帖子配套写答案和解释。

------

##1. 下面的代码分别输出什么？  


```
@implementation Son : Father
- (id)init
{
    self = [super init];
    if (self)
    {
        NSLog(@"%@", NSStringFromClass([self class]));
        NSLog(@"%@", NSStringFromClass([super class]));
    }
    return self;
}
@end
```

答案：都输出"Son"  
解释：objc中`super`是编译器标示符，并不像`self`一样是一个对象，遇到向`super`发的方法时会转译成`objc_msgSendSuper(...)`，而参数中的对象还是`self`，于是从父类开始沿继承链寻找`- class`这个方法，最后在`NSObject`中找到（若无override），此时，`[self class]`和[super class]已经等价了。

<!--more-->

##2. 下面的代码报错？警告？还是正常输出什么？
```
Father *father = [Father new];
BOOL b1 = [father responseToSelector:@selector(responseToSelector:)];
BOOL b2 = [Father responseToSelector:@selector(responseToSelector:)];
NSLog(@"%d, %d", b1, b2);
```

答案：都输出"1"(YES)  
解释：objc中：  
- 不论是实例对象还是Class，都是id类型的对象（Class同样是对象）
- 实例对象的isa指向它的Class（储存所有减号方法）,Class对象的isa指向元类（储存所有加号方法）
- 向一个对象（id类型）发送消息时，都是从这个对象的`isa`指针指向的Class中寻找方法

回到题目，当像`Father`类发送一个实例方法（`- responseToSelector`）消息时：
1. 会从它的`isa`，也就是Father元类对象中寻找，由于元类中的方法都是类方法，所以自然找不到
2. 于是沿继承链去父类NSObject元类中寻找，依然没有
3. 由于objc对这块的设计是，NSObject的元类的父类是NSObject类（也就是我们熟悉的NSObject类），其中有所有的实例方法，因此找到了`- responseToSelector`

补充：NSObject类中的所有实例方法`很可能`都对应实现了一个类方法（至少从开源的代码中可以看出来），如`+ resonseToSelector`，但并非公开的API，如果真的是这样，上面到第2步就可以找到这个方法。  
再补充： 非NSObject的selector这样做无效。

##3. 请求很快就执行完成，但是completionBlock很久之后才设置，还能否执行呢？

```
...
// 当前在主线程

[request startAsync]; // 后台线程异步调用，完成后会在主线程调用completionBlock
sleep(100); // sleep主线程，使得下面的代码在后台线程完成后才能执行
[request setCompletionBlock:^{
    NSLog(@"Can I be printed?");
}];
...
```

答案：可以（有条件）  
解释：为了方便解释，我们将其考虑成`gcd`的两个线性`queue`：main queue 和 back queue  

当代码执行到`sleep(100)`时，这两个queue要执行的顺序看起来是这样的：  
- main: *--- sleep -------------------------> | ---setCompletionBlock--->
- back: *--- network ---->

于是网络请求很快回来，回调函数一般要执行如：
```
// 回到主线程执行回调
dispatch_async(dispatch_get_main_queue(), ^{
  if (self.completionBlock) self.completionBlock();
});
```
于是成了这样：  

- main: *----sleep----> | ---setCompletionBlock---> | ---invoke completionBlock---->
- back: *
 
所以，当sleep结束后，主线程保持了调用顺序：  
- main: *---setCompletionBlock---> | ---invoke completionBlock---->  

此时，`completionBlock`的执行是在`setCompletionBlock`，之后的，所以可以正常回调。  

注：这个解释有一个有限制条件，如果用下面的方法回调，则情况就会不同了：  
```
// 回到主线程执行回调
if (self.completionBlock) {
  dispatch_async(dispatch_get_main_queue(), ^{
    self.completionBlock();
});

```


##4. 不使用IB时，下面这样做有问题么？
```
- (void)viewDidLoad
{
  [super viewDidLoad];
  CGRect frame = CGRectMake(0, 0, self.view.bounds.size.width * 0.5, self.bounds.size.height * 0.5);
  UIView *view = [[UIView alloc] initWithFrame:frame];
  [self.view addSubview:view];
}
```

解释：不使用IB手动创建ViewController时，在`viewDidLoad`中并未进行位置的初始化，原来遇到过不少次这个小坑，当外部创建这个vc时：  
```
TestViewController *vc = [[TestViewController alloc] init];
vc.view.frame = CGRectMake(0, 0, 100, 100);
//...
```

我们知道，`ViewController`的view初始化大概流程是：  
```
- (UIView *)view
{
  if (!_view) {
    [self loadView];
    [self viewDidLoad]; // Edit: 这句话移动到括号内，感谢@change2hao的提醒
  }
}
```

所以在外部执行到`vc.view.frame = CGRectMake(0, 0, 100, 100);`这句话时，在赋值操作执行前，`viewDidLoad`就已经被调用，因而在`viewDidLoad`中对view frame的取值都是默认值（window的大小），而非设定值。  

注： 使用IB加载时如上情况也会发生，只是一般在IB就已经有一个预设值了。  

-----

# 后续题目继续补充中

-----

原创文章，转载请注明源地址，[blog.sunnyxx.com](blog.sunnyxx.com)

