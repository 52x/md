title: .bash_profile vs .bashrc
date: 2015-12-18 5:32 PM
categories: 
tags: [bash,linux,mac]
---
Bash的配置文件位置包括`.bash_profile`和 `.bashrc`，那这两个配置文件有什么区别呢？

<!--more-->
![](http://harchiko.qiniudn.com/54002458_p0.jpg)

根据[http://linux.die.net/man/1/bash](http://linux.die.net/man/1/bash)的描述，`.bash_profile` is executed for login shells, while `.bashrc` is executed for interactive non-login shells.

## 什么是 login 和 non-login shell ?

当通过console**登录**时（不管是通过物理机器还是远程ssh），`.bash_profile`在初始化命令弹出之前就已经执行。

但是，当你**已经登录进机器**，不管是通过Gnome还是KDE的xterm，那么`.bashrc`在窗口弹出之前执行。而且，`.bashrc`在使用`/bin/bash`在Terminal调用时同样会执行。


## 为什么是两个不同的文件？

`.bash_profile`会在每次登录之前打印一长串的诊断信息，如果你只想在启动时看到，使用`.bash_profile`,如果放在`.bashrc`中，你会在每一次新启动一个Terminal时候看到一次。

## Mac OS X -- 一个例外

每一次新启动一个terminal窗口时，调用`.bash_profile`而不是`.bashrc`,其它GUI 终端模拟器也可能这样做，但是多数不是这样做的。

## 建议

多数情况下，分开管理不是很方便，**当你想要设定一个环境变量的时候，你想要两个都生效**,你可以通过在`.bash_profile` 中执行命令 `source .bashrc`，然后在`.bashrc`中写入PATH或者其他的设定。

添加如下命令到`.bash_profile`中

```bash
if [ -f ~/.bashrc ]; then
   source ~/.bashrc
fi
```

这样当你登录到console时，`.bashrc`会被调用。

## 参考链接

[http://www.joshstaiger.org/archives/2005/07/bash_profile_vs.html](http://www.joshstaiger.org/archives/2005/07/bash_profile_vs.html)