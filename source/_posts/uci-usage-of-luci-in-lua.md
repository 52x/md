---
title: LuCI的uci使用（lua）
date: 2016-04-08 13:36:39
tags:
 - LuCI
 - uci
 - lua
categories:
 - uci
---
## add ##

```
local uci = require("luci.model.uci").cursor()
uci:add("wireless", "wifi-iface")
uci:commit("wireless")
```

actual result：

`config wifi-iface` 一句会添加至 `/etc/config/wireless` 的末尾。


## delete ##

sample `cat /etc/config/network`:

```
config interface 'lan'
    option ipaddr '192.168.1.1'
```

codes:

```
local uci = require("luci.model.uci").cursor()
uci:delete("network", "lan", "ipaddr") -- 如果不加第三个参数（ipaddr），则会delete掉lan的所有options，包括config interface 'lan'一句。
uci:commit("network")
```


## delete_all ##

```
local uci = require("luci.model.uci").cursor()
uci:delete_all("firewall", "redirect")
uci:commit("firewall")
```


## foreach ##

sample `cat /etc/config/wireless`:

```
config wifi-device 'radio0'
    option country 'CN'
    option disabled '0'

config wifi-iface
    option ssid 'aaa'
    option hidden '0'

config wifi-iface
    option ssid 'bbb'
    option hidden '0'
```

codes:

```
local uci = require("luci.model.uci").cursor()
uci:foreach("wireless", "wifi-iface", function(s)
    print(s[".type"], s[".name"], s[".index"]) -- 三个是默认自带有的
    print(s.ssid)

    -- 注意此处set之后不能commit，必须在遍历外面commit
    uci:set("wireless", s[".name"], "hidden", "1")
end)

-- 没有set则无需commit。
uci:commit("wireless")
```


## get ##

```
local uci = require("luci.model.uci").cursor()
local p = uci:get("network", "lan", "ipaddr")
print(p)
```


## get_all ##

```
local LuciUtil = require("luci.util")
local uci = require("luci.model.uci").cursor()
local p = uci:get_all("network", "lan")
LuciUtil.dumptable(p)
```


## get_list ##

sample `cat /etc/config/network`:

```
config interface 'wan'
    option proto 'static'
    option ipaddr '10.1.1.2'
    option netmask '255.255.255.0'
    option gateway '10.1.1.1'
    list dns '2.2.2.2'
    list dns '1.1.1.1'
```

codes:

```
local uci = require("luci.model.uci").cursor()
local LuciUtil = require("luci.util")
local p = uci:get_list("network", "wan", "dns")
LuciUtil.dumptable(p)
```


## section ##

```
local uci = require("luci.model.uci").cursor()
local options = {
    ["src"]       = "wan",
    ["src_dport"] = "8080",
    ["proto"]     = "tcpudp",
    ["target"]    = "DNAT",
    ["dest"]      = "lan",
    ["dest_port"] = "80",
    ["dest_ip"]   = "192.168.1.100",
    ["name"]      = "aaa"
}
uci:section("firewall", "redirect", sectionName, options)
uci:commit("firewall")
```

actual result（端口转发的例子）：

```
# 'abc'取决于sectionName，sectionName = nil则没有。
config redirect 'abc'
    # options = nil则下面部分都没有
    option src 'wan'
    option src_dport '8080'
    option proto 'tcpudp'
    option target 'DNAT'
    option dest 'lan'
    option dest_port '80'
    option dest_ip '192.168.1.100'
    option name 'aaa'
```


## set ##

```
local uci = require("luci.model.uci").cursor()
uci:set("network", "lan", "ipaddr", "value_u_wanna_set")
uci:commit("network")

```
> set有个特点，如果要设置的值跟目标文件中的值一样，则uci不会修改配置文件，相当于没改。
> 可查看文件修改日期发现：`ls -le /etc/config/network`


## set_confdir ##

```
function test()
    local uci = require("luci.model.uci").cursor()
    uci:set_confdir("/tmp/etc/config")
    -- set_confdir之后，此uci都会去操作/tmp/etc/config目录
    -- 作用域为本函数内
end
```


## set_list ##

```
local uci = require("luci.model.uci").cursor()
local tbl = {
    [1] = "3.3.3.3",
    [2] = "4.4.4.4"
}

uci:set_list("network", "wan", "dns", tbl)
或者
uci:set_list("network", "wan", "dns", "5.5.5.5")

uci:commit("network")
```

> 注意`set_list`会先删掉文件中的list项再添加进去。


更多LuCI API在此：[https://github.com/openwrt/luci/tree/master/documentation/api](https://github.com/openwrt/luci/tree/master/documentation/api "https://github.com/openwrt/luci/tree/master/documentation/api")
