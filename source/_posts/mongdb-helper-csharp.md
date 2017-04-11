title: mongodb C#操作类
date: 2012-7-17 12:45:21
tags: [C#,mongodb]
---

Mongodb研究了一些日子，以后会把遇到的问题都贴出来。先把自己的操作类贴出来

操作类使用的是C# samus 客户端([下载](https://github.com/samus/mongodb-csharp "下载"))，据老戴测试相率不错。

```js
public class MongoDbHelper
    {
        public static readonly string connectionString = "Servers=192.168.0.3:27017;ConnectTimeout=30000;ConnectionLifetime=300000;MinimumPoolSize=8;MaximumPoolSize=256;Pooled=true";
        public static readonly string database = "Friends";

        #region 新增
        /// <summary>
        /// 插入新数据
        /// </summary>
        /// <typeparam name="T"></typeparam>
        /// <param name="collectionName"></param>
        /// <param name="entiry"></param>
        public static void InsertOne<T>(string collectionName, T entity) where T : class
        {
            using (Mongo mongo = new Mongo(connectionString))
            {
                mongo.Connect();
                IMongoDatabase friends = mongo.GetDatabase(database);
                IMongoCollection<T> categories = friends.GetCollection<T>(collectionName);
                categories.Insert(entity,true);
                mongo.Disconnect();
                
            }
        }
        /// <summary>
        /// 插入多个数据
        /// </summary>
        /// <typeparam name="T"></typeparam>
        /// <param name="collectionName"></param>
        /// <param name="entiry"></param>
        public static void InsertAll<T>(string collectionName, IEnumerable<T> entity) where T : class
        {
            using (Mongo mongo = new Mongo(connectionString))
            {
                mongo.Connect();
                IMongoDatabase friends = mongo.GetDatabase(database);
                IMongoCollection<T> categories = friends.GetCollection<T>(collectionName);
                categories.Insert(entity, true);
                mongo.Disconnect();
                
            }
        }
        #endregion

        #region 更新
        /// <summary>
        /// 更新操作
        /// </summary>
        /// <typeparam name="T">类型</typeparam>
        /// <param name="collectionName">表名</param>
        /// <param name="query">条件</param>
        /// <param name="entry">新实体</param>
        public static void Update<T>(string collectionName,Document entity,Document query) where T:class
        {
            using (Mongo mongo = new Mongo(connectionString))
            {
                mongo.Connect();
                IMongoDatabase friends = mongo.GetDatabase(database);
                IMongoCollection<T> categories = friends.GetCollection<T>(collectionName);
                categories.Update(entity, query,true);
                mongo.Disconnect();
            }
        }
        /// <summary>
        /// 更新操作
        /// </summary>
        /// <typeparam name="T">类型</typeparam>
        /// <param name="collectionName">表名</param>
        /// <param name="query">条件</param>
        /// <param name="entry">新实体</param>
        public static void UpdateAll<T>(string collectionName, Document entity, Document query) where T : class
        {
            using (Mongo mongo = new Mongo(connectionString))
            {
                mongo.Connect();
                IMongoDatabase friends = mongo.GetDatabase(database);
                IMongoCollection<T> categories = friends.GetCollection<T>(collectionName);
                categories.Update(entity, query,UpdateFlags.MultiUpdate,true);
                mongo.Disconnect();
            }
        }
        #endregion

        #region 查询
        /// <summary>
        /// 获取一条数据
        /// </summary>
        /// <typeparam name="T"></typeparam>
        /// <param name="collectionName"></param>
        /// <param name="query"></param>
        /// <returns></returns>
        public static T GetOne<T>(string collectionName, Document query) where T : class
        {
            T result = default(T);
            using (Mongo mongo = new Mongo(connectionString))
            {
                mongo.Connect();
                IMongoDatabase friends = mongo.GetDatabase(database);
                IMongoCollection<T> categories = friends.GetCollection<T>(collectionName);
                result = categories.FindOne(query);
                mongo.Disconnect();
                
            }
            return result;
        }
        /// <summary>
        /// 获取一条数据
        /// </summary>
        /// <typeparam name="T"></typeparam>
        /// <param name="collectionName"></param>
        /// <param name="query"></param>
        /// <returns></returns>
        public static T GetOne<T>(string collectionName, Document query,Document fields) where T : class
        {
            T result = default(T);
            using (Mongo mongo = new Mongo(connectionString))
            {
                mongo.Connect();
                IMongoDatabase friends = mongo.GetDatabase(database);
                IMongoCollection<T> categories = friends.GetCollection<T>(collectionName);
                result = categories.Find(query, fields).Skip(0).Limit(1).Documents.First();
                mongo.Disconnect();

            }
            return result;
        }
        /// <summary>
        /// 获取一个集合下所有数据
        /// </summary>
        /// <param name="collectionName"></param>
        /// <returns></returns>
        public static List<T> GetAll<T>(string collectionName) where T:class
        {
           List<T> result=new List<T>();
            using (Mongo mongo = new Mongo(connectionString))
            {
                mongo.Connect();
                IMongoDatabase friends = mongo.GetDatabase(database);
                IMongoCollection<T> categories = friends.GetCollection<T>(collectionName);
                foreach (T entity in categories.FindAll().Limit(50).Documents)
                {
                    result.Add(entity);
                }
                mongo.Disconnect();
                
            }
            return result;
        }
        /// <summary>
        /// 获取列表
        /// </summary>
        /// <typeparam name="T"></typeparam>
        /// <param name="collectionName"></param>
        /// <param name="query"></param>
        /// <param name="Sort"></param>
        /// <param name="cp"></param>
        /// <param name="mp"></param>
        /// <returns></returns>
        public static List<T> GetList<T>(string collectionName,object selector, Document sort, int cp, int mp) where T : class
        {
            List<T> result = new List<T>();
            using (Mongo mongo = new Mongo(connectionString))
            {
                mongo.Connect();
                IMongoDatabase friends = mongo.GetDatabase(database);
                IMongoCollection<T> categories = friends.GetCollection<T>(collectionName);
                foreach (T entity in categories.Find(selector).Sort(sort).Skip((cp - 1) * mp).Limit(mp).Documents)
                {
                    result.Add(entity);
                }
                mongo.Disconnect();
                
            }
            return result;
        }
        /// <summary>
        /// 获取列表
        /// </summary>
        /// <typeparam name="T"></typeparam>
        /// <param name="collectionName"></param>
        /// <param name="query"></param>
        /// <param name="Sort"></param>
        /// <param name="cp"></param>
        /// <param name="mp"></param>
        /// <returns></returns>
        public static List<T> GetList<T>(string collectionName, object selector,object fields, Document sort, int cp, int mp) where T : class
        {
            List<T> result = new List<T>();
            using (Mongo mongo = new Mongo(connectionString))
            {
                mongo.Connect();
                IMongoDatabase friends = mongo.GetDatabase(database);
                IMongoCollection<T> categories = friends.GetCollection<T>(collectionName);
                foreach (T entity in categories.Find(selector,fields).Sort(sort).Skip((cp - 1) * mp).Limit(mp).Documents)
                {
                    result.Add(entity);
                }
                mongo.Disconnect();

            }
            return result;
        }
        #endregion


        #region 删除
        /// <summary>
        /// 删除数据
        /// </summary>
        /// <typeparam name="T"></typeparam>
        /// <param name="collectionName"></param>
        /// <param name="entity"></param>
        public static void Delete<T>(string collectionName, Document query) where T : class
        {
            using (Mongo mongo = new Mongo(connectionString))
            {
                mongo.Connect();
                IMongoDatabase friends = mongo.GetDatabase(database);
                IMongoCollection<T> categories = friends.GetCollection<T>(collectionName);
                categories.Remove(query, true);
                mongo.Disconnect();
            }
        }
        #endregion
    }
```