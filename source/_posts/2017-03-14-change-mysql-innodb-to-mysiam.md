---
title: 将数据库innodb的表转换为MYSIAM
tags: [mysql,innodb,mysiam]
date: 2017-03-14 14:53:14
updated: 2017-03-14 14:53:14
categories: [Mysql]
keywords: [mysql,innodb,mysiam]
description:
---
服务器：Debian 8
使用apt-get安装的mysql
1. 转换数据库表引擎
执行 `/usr/bin/mysql_convert_table_format testDB  --user=root --password=password`
其中testDB 是数据库名
root是用户
password是密码
要把所有的表都转换掉
2. 修改my.cnf 配置

在mysqld断，增加
```
default-storage-engine=MYISAM
innodb=OFF
```

3. 重启mysql即可