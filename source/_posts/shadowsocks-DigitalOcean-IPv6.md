title: shadowsocks+DigitalOcean+IPv6
date: 2015-05-10 23:07:45
categories: Network
tags: [socks5 proxy,IPv6,VPS]
---
shadowsocks是科学上网的利器，利用DigitalOcean等VPS可部署shadowsocks服务器端。配合IPv6，不仅可以跨越GFW，也可以跳过校园计费网关，实现免流量`全网访问`。

<!-- more -->

# shadowsocks简介
[shadowsocks](http://shadowsocks.org/en/index.html)是SOCKS5代理，与常见的VPN全局代理不同，SOCKS5代理默认只针对浏览器。我对于代理的理解是：当A能直接访问B，B能直接访问C，但是A却不能直接访问C时，可以用B做“跳板”，将C中的内容通过B返回给A。由于GFW的存在，youtube等网站大陆不能直接访问。但是大陆可以访问美国的大部分VPS，这些VPS自然可以直接访问youtube。那么利用VPS做“跳板”，在上面搭建shadowsocks服务器，就可以实现科学上网。
[shadowsocks客户端](http://shadowsocks.org/en/download/clients.html)，几乎是全平台覆盖，PC的Windows、Mac OS X、Linux；手机的Android、iOS；路由器的OpenWRT都可以下载到对应版本。
shadowsocks项目开源，可以贡献代码，或者根据个人需求自行编译。

# GitHub Student pack
[Student Developer Pack](https://education.github.com/pack)是面向学生的优惠包。利用教育信息成功注册后，会获得一些开发工具的优惠包。之前可以使用*.edu.cn邮箱注册，现在GitHub关闭了.cn学生邮箱注册，可以使用上传学生证的方式注册。在优惠包中，有DigitalOcean的100$代金券，穷学生的福利！抓紧时间注册~

# DigitalOcean
云主机服务商，注册用户后，充值5$激活激活账户后，方可使用GitHub赠送的100$代金券，此时账户里会有105$。Create Droplet，![资费标准](http://digitalocean.youhuima.cc/wp-content/uploads/2014/05/12-size.jpg)如果仅用于搭建shadowsocks 5$/月的最低标准足够，账户中的余额可以使用21个月，花5$购买将近2年VPS使用权，还是很值的。

# 安装shadowsocks server
具体操作参考如下两篇文章，总之很简单，几乎是一键安装。
[Shadowsocks服务器部署](http://hceasy.com/2013/12/shadowsocks-%E6%9C%8D%E5%8A%A1%E7%AB%AF%E9%83%A8%E7%BD%B2/)
[Shadowsocks使用说明](https://github.com/shadowsocks/shadowsocks/wiki/Shadowsocks-%E4%BD%BF%E7%94%A8%E8%AF%B4%E6%98%8E)
需要注意的DigitalOcean的vps在Ubuntu系统中`apt-get install python-gevent python-pip`时会出错，重新添加源就可以。

# 下载shadowsocks客户端
在客户端中填写与服务器相对应的信息，便可以科学上网了。

# 校园网IPv6
通过之前的部署，已经可以科学上网。但是考虑到校园网的特殊性——上网按流量收费。VPS上每个月有1TB的流量，但在科学上网时同时走校园网和VPS流量，瓶颈自然在校园网。
但是，校园网的优势是IPv6，目前校园网的IPv6不仅速度快，而且不限流量。DigitalOcean的VPS支持IPv6，因此开通shadowsocks的IPv6功能，就可以不登录学校网关而访问全网！
首先将VPS开启IPv6服务[How To Enable IPv6 for DigitalOcean Droplets](https://www.digitalocean.com/community/tutorials/how-to-enable-ipv6-for-digitalocean-droplets)。
将shadowsocks的服务器端配置文件做如下修改：`"server":"::"`,此时可以同时监听IPv4和IPv6，将客户端的IP地址修改为VPS的IPv6地址即可。

需要注意的是，DigtalOcean的IPv6地址形如：public_ipv6_address/64，除了特殊说明外，IPv6地址均不包含`/64`。

# 其他
1. 不登录校园网，只使用IPv6时，需要将客户端改为“全局代理”模式，否则用“PAC模式”即可。
2. SOCKS5代理可以支持QQ，在QQ登录页面的右上角，可以设置SOCKS5代理，ip地址为127.0.0.1。
3. 浏览器访问有问题时，可以考虑手动修改浏览器SOCKS5代理，地址也是127.0.0.1。
4. PC使用shadowsocks客户端，发射wifi热点时，手机只能浏览网页，app不能使用；但是Android的shadowsocks客户端，几乎可是实现VPN的功能，微信、支付宝等app都可以使用。
5. PC端的ssh、github等命令不支持。
6. DigitalOcean+shadowsocks网速可观，youtube 720P无卡顿；优酷等国内视频网站，不涉及版权时，可正常播放。

# 后记
上周去国科大，连接他们学校的wifi后，没有登录网关，通过shadowsocks直接全网访问！以后游览全国各地校园，只要wifi支持IPv6，便能享受免费上网的福利啦~