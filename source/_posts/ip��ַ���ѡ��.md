title: ip地址库的选择
date: 2015-12-27 12:49:13
tags: [ip, golang]
---
目前市面上常用的ip地址库，有以下几种

* 1，新浪的api接口（限制未知）http://int.dpool.sina.com.cn/iplookup/iplookup.php?format=js&ip=218.192.3.42
* 2，淘宝的api接口 （10QPS） http://ip.taobao.com/
* 3，百度的api接口 (100W/天) 
* 4， 纯真ip(国内，数据提供模式导致数据可能不准确)
* 5，ipip.net （国内）
* 6， maxmid  (国外)


####如何选择
1，web客户端
直接选择新浪的请求连接（http://int.dpool.sina.com.cn/iplookup/iplookup.php?format=js）就可以了，简单方便省事

2，在服务器端：

 * 因为服务器端获取第三方api请求，不可控因素过多，建议是选择服务器端查询数据库的方式来走，这里可选的有三种：纯真ip，ipip.net和国外的maxmid。

* 从精度和准确度来讲应该是： 纯真ip < ipip.et = maxmid

* 语言支持上，他们也大差不差，各个主流语言的程序SDK都已经给出。

* 产品使用度上，他们都有自己的不少的用户。但是貌似ipip.net更专业点，携程等等都是他们的用户。

因为maxmid是国外的，所以建议现在使用 ipip.net

这里给出项目使用到的SDK：

* nodejs ：https://github.com/ChiChou/node-ipip

* golang ：https://github.com/wangtuanjie/ip17mon