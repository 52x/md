title: golang中使用mongodb
date: 2015-12-27 12:47:31
tags: [golang,mongodb]
---

## mgo简介
****
mongodb官方没有关于go的mongodb的驱动，因此只能使用第三方驱动，mgo就是使用最多的一种。
mgo（音mango）是MongoDB的Go语言驱动，它用基于Go语法的简单API实现了丰富的特性，并经过良好测试。

 官网：<http://labix.org/mgo>

 文档：http://godoc.org/gopkg.in/mgo.v2


## 安装与使用
****
安装
````js
go get gopkg.in/mgo.v2
````

go中使用
````js
package main

import (
	"gopkg.in/mgo.v2"
	"gopkg.in/mgo.v2/bson"
)

type Person struct {
	Id    bson.ObjectId `bson:"_id"`
	Name  string        `bson:"tname"` //bson:"name" 表示mongodb数据库中对应的字段名称
	Phone string        `bson:"tphone"`
}

const URL = "192.168.1.43:50000" //mongodb连接字符串

var (
	mgoSession *mgo.Session
	dataBase   = "mydb"
)

/**
 * 公共方法，获取session，如果存在则拷贝一份
 */
func getSession() *mgo.Session {
	if mgoSession == nil {
		var err error
		mgoSession, err = mgo.Dial(URL)
		if err != nil {
			panic(err) //直接终止程序运行
		}
	}
	//最大连接池默认为4096
	return mgoSession.Clone()
}
//公共方法，获取collection对象
func witchCollection(collection string, s func(*mgo.Collection) error) error {
	session := getSession()
	defer session.Close()
	c := session.DB(dataBase).C(collection)
	return s(c)
}

/**
 * 添加person对象
 */
func AddPerson(p Person) string {
	p.Id = bson.NewObjectId()
	query := func(c *mgo.Collection) error {
		return c.Insert(p)
	}
	err := witchCollection("person", query)
	if err != nil {
		return "false"
	}
	return p.Id.Hex()
}

/**
 * 获取一条记录通过objectid
 */
func GetPersonById(id string) *Person {
	objid := bson.ObjectIdHex(id)
	person := new(Person)
	query := func(c *mgo.Collection) error {
		return c.FindId(objid).One(&person)
	}
	witchCollection("person", query)
	return person
}

//获取所有的person数据
func PagePerson() []Person {
	var persons []Person
	query := func(c *mgo.Collection) error {
		return c.Find(nil).All(&persons)
	}
	err := witchCollection("person", query)
	if err != nil {
		return persons
	}
	return persons
}

//更新person数据
func UpdatePerson(query bson.M, change bson.M) string {
	exop := func(c *mgo.Collection) error {
		return c.Update(query, change)
	}
	err := witchCollection("person", exop)
	if err != nil {
		return "true"
	}
	return "false"
}

/**
 * 执行查询，此方法可拆分做为公共方法
 * [SearchPerson description]
 * @param {[type]} collectionName string [description]
 * @param {[type]} query          bson.M [description]
 * @param {[type]} sort           bson.M [description]
 * @param {[type]} fields         bson.M [description]
 * @param {[type]} skip           int    [description]
 * @param {[type]} limit          int)   (results      []interface{}, err error [description]
 */
func SearchPerson(collectionName string, query bson.M, sort string, fields bson.M, skip int, limit int) (results []interface{}, err error) {
	exop := func(c *mgo.Collection) error {
		return c.Find(query).Sort(sort).Select(fields).Skip(skip).Limit(limit).All(&results)
	}
	err = witchCollection(collectionName, exop)
	return
}


````
### 解释说明
****
#### 连接字符串
连接字符串可以使用mongodb标准形式
````js

mongodb://myuser:mypass@localhost:40001,otherhost:40001/mydb

````
#### 结构体声明
````js
type Person struct {
	Id_   bson.ObjectId `bson:"_id"`
	Name  string        `bson:"tname"` //bson:"name" 表示mongodb数据库中对应的字段名称
	Phone string        `bson:"tphone"`
}
````
注意Person的字段首字母大写，不然不可见。通过bson:”name”这种方式可以定义MongoDB中集合的字段名，如果不定义，mgo自动把struct的字段名首字母小写作为集合的字段名。如果不需要获得id_，Id_可以不定义，在插入的时候会自动生成。但是建议是通过程序生成，这样可以提高mongodb的运行效率，也可以在插入完成之后直接返回ObjectId，供其他程序使用

手动创建一个ObjecitId
````js
bson.NewObjectId()//创建一个objectid
````

