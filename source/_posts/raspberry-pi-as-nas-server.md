title: 从零开始将树莓派打造成家用下载机【一】 网络配置篇
date: 2016-04-17 1:28 PM
categories: "raspberry"
tags: [linux,pi]
---

本篇讲介绍一下树莓派基本的网络配置，使其能够在公网正常访问。
<!--more-->

![http://harchiko.qiniudn.com/Pi_3_Model_B.png](http://harchiko.qiniudn.com/Pi_3_Model_B.png)

### 烧录micro sd卡

具体操作可查看官方[https://www.raspberrypi.org/documentation/installation/installing-images/README.md](https://www.raspberrypi.org/documentation/installation/installing-images/README.md)，比较简单，不详述。

### 配置IP地址

烧录好之后插上SD卡，接好网线启动树莓派，看到树莓派灯正常亮系统应该就算启动完成了。

1. 进入路由器后台,打开DHCP服务，查看树莓派的ip地址，将其修改为固定ip地址后重启路由器(我的为192.168.0.169)。
2. 使用 `ssh pi@192.168.0.169 `登录树莓派（Mac/Linux 下直接使用 Terminal, Windows下可使用 Xshell）

### 配置公网访问


### 路由器
* DMZ主机配置
在路由器中，配置中找到DMZ主机选项，将其设置为树莓派的ip。

### SSH配置
#### 密码修改
    使用命令 `passwd pi`修改 pi 的密码。建议密码修改强度足够强，因为这台主机会暴露在公网。
#### SSH端口修改
    出于同样的安全考虑，修改ssh端口

* 修改配置文件 `vi /etc/ssh/sshd_config` 将ssh端口修改为 xxxx(你希望的端口),这里我设定的是1213。
* 重启SSH服务，`sudo service ssh restart`

#### 查看本地IP是否能从公网访问

> 在国内，因为ipv4地址不够用或者一些其他原因，你所使用的ip可能是电信分配的局域网的ip，这时候你就需要找电信客服说一下你需要公网ip让他们帮你改回来。

这里通过 telnet 服务看一下是否当前分配的是公网ip
1. **查看本机的公网ip**
使用命令 `curl ifconfig.me`,或者直接在百度中搜索ip。
2. **本地查看SSH服务端口号是否更改过来 ** `telnet 192.168.0.169 1213` 
(`#xxxx` 指的是端口号)
3. **通过公网查看** 通过第一步查看到的ip来从公网访问`telnet xxx.xxx.xxx 1213`
(这里使用公网 ip）如果能够正常运行的话则证明此树莓派已经运行在公网。

这样 ssh 登录命令变为 `ssh pi@xxx.xxx.xxx -p 1213` （指定端口）


#### 将本地ip与域名绑定

因为家庭宽带的ip是不固定的，如果使用 ip 进行连接的话每次都要先获取到机器的当前ip才能进行ssh登录，这里我们将ip地址绑定到二级域名上。

按照此repo: [https://github.com/zhaochunqi/change_my_ip_on_dnspod](https://github.com/zhaochunqi/change_my_ip_on_dnspod) 中README进行相应配置。

1. clone 这个 repo， 在Dnspod中获取到 APIKEY，以及响应的二级域名需要的参数。
2. 修改库中的 change_my_ip_on_dnspod.sh `Shell`脚本。
3. 使用命令`chmod +x change_my_ip_on_dnspod.sh`添加脚本的可执行权限。
4. 运行 ./change_my_ip_on_dnspod.sh 查看返回值,得到如下图的结果即为成功。![http://harchiko.qiniudn.com/Screen%20Shot%202016-04-18%20at%2012.13.26%20AM.png](http://harchiko.qiniudn.com/Screen%20Shot%202016-04-18%20at%2012.13.26%20AM.png)
3. 登录树莓派系统 `crontab -e`在末尾添加 
```bash
# every 20 mins
*/20 * * * * ./path/to/change_ip_4_pi.sh
```
`path/to/change_ip_4_pi.sh` 替换为你的真实地址。

至此，你能从你刚刚绑定的二级域名直接访问到你的树莓派:

```bash
ssh pi@haha.mydomain.com -p 1213
```
