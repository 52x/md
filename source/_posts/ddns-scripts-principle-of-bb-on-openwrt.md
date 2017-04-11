---
title: OpenWRT的ddns-scripts原理（BB版）
date: 2016-04-26 15:59:30
tags:
 - ddns-scripts
 - openwrt
categories:
 - openwrt
---

## 更新的url ##

更新的url全部都记录在`/usr/lib/ddns/services`

花生壳的更新url（网上找来）：

```
http://[USERNAME]:[PASSWORD]@ddns.oray.com/ph/update?hostname=[DOMAIN]&myip=[IP]
```

公云的更新url（公云API有提供：[http://www.pubyun.com/wiki/%E5%B8%AE%E5%8A%A9:api](http://www.pubyun.com/wiki/%E5%B8%AE%E5%8A%A9:api "http://www.pubyun.com/wiki/%E5%B8%AE%E5%8A%A9:api")

```
http://[USERNAME]:[PASSWORD]@members.3322.org/dyndns/update?system=dyndns&hostname=[DOMAIN]&myip=[IP]
```

> 如需增加新的ddns服务商，则需要把该服务商的更新url填于此处。

## 上报流程 ##

启动`/etc/init.d/ddns start`。

拿出`/usr/lib/ddns/services`里面的对应更新url，并存于`update_url`变量中。

然后将`update_url`里面的`USERNAME`、`PASSWORD`和`DOMAIN`分别用配置文件`/etc/config/ddns`中的`username`、`password`和`domain`替换（以上面的花生壳更新url为例）。

`update_url`中的`IP`用当前路由器wan口ip替换掉，最终存于`final_url`变量中。

然后通过`nslookup [DOMAIN]`命令查找到域名的公网IP，存于`registered_ip`变量中。

然后`registered_ip`与路由器的wan口IP进行比较：

- 相同的话则不上报
- 不相同的话，则通过`/usr/bin/wget -O - ${final_url}`上报给ddns服务商。

## 相关配置参数 ##

`cat /etc/config/ddns`

```
config service 'oray'
    option enabled        '1'
    option service_name   'oray.com'

    option domain         '' # 注册的域名
    option username       '' # 账号名
    option password       '' # 密码

    option ip_source      'network'
    option ip_network     'wan'

    option check_interval '5'
    option check_unit     'minutes'

    option force_interval '12'
    option force_unit     'hours'
```

`ip_source`和`ip_network`的解释如下：

```
# If "ip_source" is "network" you specify a network section in your 
# /etc/network config file (e.g. "wan", which is the default) with
# the "ip_network" option.  If you specify "wan", you will update
# with whatever the ip for your wan is.
# 
# If "ip_source" is "interface" you specify a hardware interface 
# (e.g. "eth1") and whatever the current ip of this interface is
# will be associated with the domain when an update is performed.
#
# If "ip_source" is "script" you specify a script to obtain ip address.
# The "ip_script" option should contain path to your script.
#
# The last possibility is that "ip_source" is "web", which means
# that in order to obtain our ip address we will connect to a 
# website, and the first valid ip address listed on that page
# will be assumed to be ours.  If you are behind another firewall
# this is the best option since none of the local networks or 
# interfaces will have the external ip.  The website to connect
# to is specified by the "ip_url" option.  You may specify multiple
# urls in the option, separated by whitespace.
```

配合`ddns`的默认配置可看出，`ip_source`和`ip_network`是成对出现的：

```
#option ip_source  "network" 
#option ip_network "wan"
```

```
#option ip_source  "interface" 
#option ip_network "eth0.1"
```

```
#option ip_source  "script" 
#option ip_network "path to your script"
```

按照上面的`ddns`配置，如果你的路由是二级路由，那么wan口ip是局域网地址时，也照样会把本wan口的ip上报给ddns服务商的。

`ip_url`参数：

此配置没有填写`ip_url`参数，因为`ddns`脚本获取当前wan口ip时，先判断`ip_source`参数

- `ip_source = "network"`，则最终通过`ubus call network.interface dump`拿到wan口ip。
- `ip_source = "interface"`，则通过`ifconfig`拿到对应网口ip。
- `ip_source = "script"`，则执行自定义脚本。
- 以上都不是（相当于ip_source = "web"），则通过命令`wget -O - ${ip_url}`拿到公网ip，即访问`${ip_url}`地址来得到公网ip。如果还拿不到那就访问默认的`http://checkip.dyndns.org`网址拿公网ip。 
