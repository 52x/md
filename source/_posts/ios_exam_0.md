title: ios程序员6级考试
date: 2014-03-06 23:10:58
tags: ios6级考试
---
## 前言

ios面试题看过来
题目多来源于项目中遇到的错误和平时的误区，要是都能了如指掌，恭喜你，6级过了- -。  
考点大概是对ios框架、objc语言基础的理解，以看代码为主（那种“谈谈xxxx的理解的题就算了吧”）  

不断总结中...

 > It's examing time...  

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
<!--more-->

##2. 下面的代码报错？警告？还是正常输出什么？
```
Father *father = [Father new];
BOOL b1 = [father responseToSelector:@selector(responseToSelector:)];
BOOL b2 = [Father responseToSelector:@selector(responseToSelector:)];
NSLog(@"%d, %d", b1, b2);
```

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


-----
# 答案和解答
[请戳我，我是传送门](http://blog.sunnyxx.com/2014/03/06/ios_exam_0_key/)
-----
原创文章，转载请注明源地址，[blog.sunnyxx.com](http://blog.sunnyxx.com)
