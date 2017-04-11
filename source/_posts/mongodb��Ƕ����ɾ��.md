title: mongodb内嵌对象删除
date: 2013-02-25 15:09:08
tags: mongodb
---


这个问题在做数据结构的时候经常用到，刚开始没怎么留意，因为我的数组都只是单元素文档：只有一个ObjectId，这样用pull操作完全没有问题，但后来用对象作为了数据的内容，就是数组内嵌的对象，这时候用pull就是各种不生效。发现Mongodb对数组内对象的get和pull使用的书写格式不一致。下面我列出可以使用的书写方式：

先列出mongodb的数据结构

```js
{
  "_id" : ObjectId,
  "Uid" : ObjectId,
  "Visit" : Visit[16]
}
```

其中Visit为对象：

```js
///访客id
ObjectId  VId;
///访问时间
DateTime Time;
则删除数组内某个Vid的写法为(我的客户端是：samus)：

MongoDbHelper.Update<FriendTopicVisit>("FriendTopicVisit", new Document("$pull", new Document("Visit",new Document("Vid",visit.Vid))), new Document("_id", id));
```

方法不复杂，但不知道方法再去找方法也确实挺费时间的。