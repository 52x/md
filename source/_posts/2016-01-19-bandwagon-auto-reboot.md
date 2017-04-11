---
title: 搬瓦工科学上网的VPS设定定时重启办法
tags: [vps,bwg]
date: 2016-01-19 11:54:05
updated: 2017-02-22 11:54:05
categories: [Vps]
keywords: [bwg,ss,vps,crond,reboot,搬瓦工,科学上网]
description:
---

搬瓦工的VPS，$5.99 够便宜的，不过性能也就只能挂挂ss了,嘿嘿,正好满足了科学上网的需求。

但是正常使用中也曾经出现过，VPS死掉，SS怎么也连不上的情况，不过重启一下就好了。决定配置一个每天自动重启的功能。

搬瓦工是支持Crontab的，因此不需要配置了。

如果需要配置的话：
`#安装Crontabyum install vixie-cron crontabs#设置开机启动Crontabchkconfig crond on#启动Crontabservice crond start`

先列一下现有的Crontab看看：
`crontab -l`
提示为 no crontab for root

果然啥也没有,那就添加一条吧：

编辑命令：
`crontab –e`

按i进入编辑模式,输入

`30 4 * * * root /sbin/reboot`

表示 每天早上4：30分重启设备。按ESC 进入命令模式，输入：wq 保存退出。
最后重启crontab，使重启功能生效
`service crond restart`

完事 收工！

补记： 忽然反应到，重启时间错了，应该是按照我们的睡觉时间04：30。
从服务器上执行date，获取服务器时间，发现差了11个小时。
改为了`30 15 * * * root /sbin/reboot`
重启，这回可以收工了。

> 补充：Crontab基本格式：
>
> \*　 \*　 \*　 \*　　\*　　command
>
> 分　时　日　月　 周　 命令
>
>  
>
> 第1列表示分钟1～59 每分钟用*或者 */1表示
>
> 第2列表示小时1～23（0表示0点）
>
> 第3列表示日期1～31
>
> 第4列表示月份1～12
>
> 第5列标识号星期0～6（0表示星期天）
>
> 第6列要运行的命令