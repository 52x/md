title: 基于Shadowsocks透明代理的实现
date: 2015-02-04 01:21:44
tags: [proxy,linux]

---
**本文只是提供了一个透明代理实现的思路，具体代码等待后续补充。**


<!--more-->
程序员们最在意的，可能就是网络跟设备配置了。

做一个透明代理，能够做到：

	自动翻墙，本地无需配置。

## 网络拓扑图
![透明代理网络拓扑图](http://harchiko.qiniudn.com/Auto-Proxy/autoproxy_map.png)

## 需求

1. 一台外网VPS（能被中国大陆访问到，能够访问到Facebook等网站）
2. 一台本地Linux服务器(此处可以用智能路由器代替)。

## 方案设想

1. 本地的Linux服务器作为网关， 处理本地流量和需要代理的流量。
2. 本地服务器将需要代理的流量转发给Shadowsocks并通过Shadowsocks来实现代理功能。
3. 流量转发到Cow中，然后通过Cow来判断需要自动翻墙的网址。

## 具体方案

1. 在VPS中安装Shadowsocks，并在本地客户端测试是否成功。
2. 在本地Linux服务器中部署Cow，通过Cow来进行ShadowSocks代理。
3. 使用iptables将流量转发到Cow本地监控的端口。
