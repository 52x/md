title: mongodb分片
date: 2012-07-25 17:02:27
tags: mongodb
---


		副本集实现了网站的安全备份和故障的无缝转移，但是并不能实现数据的大容量存储，毕竟物理硬件是有极限的，这个时候就需要做分布式部署，把数据保存到其他机器上。Mongodb的分片技术就很完美的实现了这个需求。

## 理解Mongodb的分片技术即Sharding架构

什么是Sharding？说白了就是把海量数据水平扩展的集群系统，数据分表存储在Sharding的各个节点上。

Mongodb的数据分开分为chunk，每个chunk都是collection中的一段连续的数据记录，一般为200MB，超出则生成新的数据块。

## 构建Sharding需要三种角色，

- shard服务器(Shard Server)：Shard服务器是存储实际数据的分片，每个Shard可以是一个mongod实例，也可以是一组mongod实例构成的Replica Sets，为了实现每个Shard内部的故障自动切换，MongoDB官方建议每个Shard为一组Replica Sets。

- 配置服务器(Config Server)：为了将一个特定的collection存储在多个Shard中，需要为该collection指定一个Shard key，决定该条记录属于那个chunk，配置服务器可以存储以下信息：

 > 所有Shard节点的配置信息

 > 每个chunk的Shard key范围

 > chunk在各Shard的分布情况

 > 集群中所有DB和collection的Shard配置信息

         
 - 路由进程(Route Process):一个前端路由，客户端由此接入，首先询问配置服务器需要到那个Shard上查询或保存记录，然后连接相应Shard执行操作，最后将结果返回客户端。客户端只需 将原本发给mongod的查询活更新请求原封不动的发给路由器进程，而不必关心所操作的记录存储在那个Shard上。

 

 ## 构建Sharding

 由上面分析可得出，构建一个Sharding至少需要4个mongodb进程，两个Shard Server(做分片),一个Config Server，一个Route Process，然后安排如下


进程|端口|文件目录
----|---|---
Shard Server1|2000|mongodb5
Shard Server2|2001|mongodb6
Config Server|30000|mongodb7
Route Process|40000|mongodb8

## 开始配置：

- 启动Shard Server


![](/mdimg/2012072517165248.png)

![](/mdimg/2012072517185193.png)

这里启动只多了一个命令：shardsvr，用这个命令就表示这个进程是Shard进程。


- 启动Config Server

启动Config Server用的是configsvr命令

![](/mdimg/2012072517354465.png)

- 启动Route Process

![](/mdimg/2012072517351687.png)

这里设置chunk大小为1M，方便测试分片效果

- 配置Sharding

所有进程都启动好以后，剩余的就是把他们串成串儿了

 新开个cmd，然后连接到路由器进程中，使用addshard添加到路由器中

![](/mdimg/2012072517383769.png)

通过上面两次操作，整个架构已经串成了一串，但是，别着急，架构还不知道分片的数据库和片键呢

![](/mdimg/2012072517422797.png)

指定分片的数据库是Friends，然后指定按照表FriendUserAttach中的_id分片。

至此整个系统配置完毕。

## 分片验证

验证分片情况，我是用程序插入的数据，因为表是我实际所用的表，在cmd里插入就太麻烦了，这里我用客户端驱动插入10000条数据

![](/mdimg/2012072517550440.png)

用use命令切换到Friends数据库，然后stats查看当前状态

字段说明：sharded为true，说明此表是经过分片处理的

shards部分有两个Shard Server分别是："shard0000" 和 "shard0001"。"shard0000"的字段count为1016，表明此Shard Server上分布的数据量是1016条，size表示此Shard Server上分布的数据库大小，单位为b。