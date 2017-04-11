title: 使用 supervisor 来管理 linux 程序
date: 2015-03-04 05:45:39
tags: [linux,supervisor,python]

---

## `supervisor`简介
supervisor 是使用python语言编写的可以监控和控制系统类unix进程的程序，类似于ruby 的 god 。

<!--more-->

## 安装

`supervisor`是python编写，安装十分简便，只需：

```bash
$: easy_install supervisor
```

## 使用

supervisor 可以使用配置文件来启动，指定参数 -c , 默认位置使用
-help 查看。

建立`supervisord.conf`配置文件。

```
[program:cow]
command = nohup  /home/harchiko/cow/cow &
autostart = true
autorestart = true
[supervisord]
```

这是用来监控`cow`的文件。

如果提示

`Error: could not find config file supervisord.`

那是因为缺少 `[supervisord]` 模块，如上述配置文件中添加即可。

## 启动
```bash
supervisord -c supervisord.conf
```

如是启动即可。

注意，如果想要自动重启务必加上 `autorestart = true`

另外，supervisor 还有交互式命令 supervisorcli , 输入 help 查看即可了解如何操作。