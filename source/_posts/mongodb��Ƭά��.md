title: mongodb分片维护
date: 2012-07-25 18:24:03
tags: mongodb
---

Mongodb的Sharding维护也是就那几个命令，相对来说都很简单，结合实例做下演示。

- 列出所有的Shard Server

![](/mdimg/2012072518181550.png)

注意一点是：需要连接到路由的admin下。listshards的参数1是一个固定的默认值，没有特殊的意义。

- 查看Sharding的信息

![](/mdimg/2012072610260447.png)

切换到Friends数据库，使用printShardingStatus(),可以看到当前Sharding的信息。


- 对现有的表执行Sharding。

上面我们对FriendUserAttach表执行了分片，下面我们在对另外一个表FriendUser进行分片。

1, 首先我们查看下FriendUser的状态

![](/mdimg/2012072610323889.png)

 第一行 sharded=false，说明该表未被分片。然后我们连接到路由器的admin上执行分片命令

![](/mdimg/2012072610353269.png)

对数据库Friends的表FriendUser做了分片，片键是_id，我们运行命令查看下状态
![](/mdimg/2012072610384554.png)

看以看到已经成功分片。

- 新增Shard Server

新增Shard Server的用处就不在说了，这是大数据下肯定会用到的命令。下面说步骤

  首先我们在启动一个新的Mongodb ，端口号定为2002。
![](/mdimg/2012072610451996.png)

把这个新的进程添加到咱们已经做好的“串”中，注意：这点是要连接到路由的admin中
![](/mdimg/2012072610472857.png)
然后我们看下当前的分片情况
![](/mdimg/2012072610523829.png)

可以看到多了一个shard0002的分片。

- 移除分片

移除分片不是立刻实现的，他需要一个把分片上的数据转移到其他分片的过程，当转移完成后该分片才会被正式踢下线。这时候也需要多次调用命令，查看移除操作执行到了那里。

移除命令是：db.runCommand({"removeshard":"ip+端口"})，注意是这里需要用admin数据库来执行操作
![](/mdimg/2012072611011137.png)
这里看以清晰的看到状态：

> started：移除的动作刚刚开始

> ongoing：移除正在进行

> completed：完成

到最后一个提示"can't find shard" 说明已经是被踢下线了。