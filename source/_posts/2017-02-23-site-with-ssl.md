---
title: 网站正式启用https
tags: [https,ssl,vps]
date: 2017-02-23 10:19:37
updated: 2017-02-23 10:19:37
categories: [Vps]
keywords: [https,vps,ssl]
description: 终于挂上了绿色的安全锁
---

# 喜大普奔

### 2017年2月23日

本站支持https啦，看着绿色的安全锁还是比较欣喜的，哈哈。

证书是从[qcloud][qcloud]上申请的DV证书，很方便，一年期的，免费的。申请方便，使用方便，不错，不错！



### 2017年2月24日更新

加了两条跳转指令

```
RewriteCond %{HTTP_HOST} ^liuxuan.net
RewriteRule (.*) https://www.liuxuan.net/$1 [R=301,L]
RewriteCond %{SERVER_PORT} 80
RewriteRule ^(.*)$ https://www.liuxuan.net/$1 [R,L]
```





[qcloud]: https://console.qcloud.com/ssl