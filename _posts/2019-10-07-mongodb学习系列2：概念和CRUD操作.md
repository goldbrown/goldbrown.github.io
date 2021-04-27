---
layout:     post
title:      mongodb学习系列2：概念和CRUD操作
subtitle:   概念和CRUD操作
date:       2019-10-07
author:     Chris
header-img: img/post-bg-climber.jpeg
catalog: true
tags:
    - mongodb
    - nosql
    - CRUD
---



# 1 简介
MongoDB是为了快速开发互联网Web应用而生的数据库系统。下面，将从快速开发、互联网web应用和数据库系统三个方面来说明。

* 数据库系统

  首先，mongodb是一个数据库系统，类似于mysql关系型数据库。与mysql的作用是一样的，都是用来持久化的存储数据的。但是，在最直观的使用方面，mysql存储需要将数据结构化之后存储，即分割为多个字段再存储；mongodb则是直接存储整个文档（文档，可以认为是一个json对象，后面再介绍概念），不需要再拆分为多个字段。注意，mongodb和mysql一样，目的都是用来做持久存储的，而不是用来做缓存的。
* 快速开发

  1. 由于mongodb可以直接存储json对象，则有利于用户直接将前端展示的json对象或者后端处理的对象json化之后存储到mongodb中。这样，就省去字段的拆分过程。
  2. 由于mongodb是直接存储json对象，则后面维护时添加属性很方便。
  3. 通过json对象的嵌套或者mongodb的引用，在有些情况下可以代替mysql中JOIN操作，从而可以做到快速开发。
* 互联网web应用

  互联网应用一般都具有很大的数据量，因此，要求存储具有高扩展性和高性能。mongodb的数据模型和持久化策略支持高性能的读/写，支持高扩展性。相比mysql，mongodb很容易做到读写的扩展。

## 2 为什么使用mongodb
mongodb和mysql都是作为持久化存储的设备，那么，什么时候应该选取mongodb呢？先看看mongodb相对于mysql的不同。mongodb相关的概念在后面介绍。

![](https://tva1.sinaimg.cn/large/006y8mN6gy1g7pjmniw7qj31jl0u0qjg.jpg)


# 3 基本概念

* **文档(document)**：mongodb的文档，相当于关系数据库中的一行记录。
* **集合(collection)**：多个文档组成一个集合，相当于关系型数据库中的一个表。
* **数据库(database)**：多个集合，逻辑上组织在一起，就是一个数据库。
* 一个运行的mongodb server支持多个数据库。

# 4 CRUD操作
下面的示例中，使用的mongodb服务端和客户端版本都是4.0之后的版本。

## 4.1 数据库的操作

* 创建新的数据库

**语法**：
use {DATABASE_NAME}，然后插入一条记录

**示例**：
```
> use db_test // 切换到使用db_test数据库，虽然db_test还不存在。
> db.col_test.save(o)  // o为一个json对象。这里创建一个名为col_test的集合，然后插入一条记录。
```
> **注意**：使用use {DATABASE_NAME}命令之后，必须插入记录，才会真正创建一个数据库。

* 删除数据库

先切换到目标数据库，然后使用db.dropDatabase()

**示例**：
```
> use db_test
> db.dropDatabase()
```

* 显示所有数据库

```
> show dbs
```

* 显示当前正在使用的数据库

```
> db
```

## 4.2 数据表（集合）的操作

* 创建集合

**语法**：
使用db.createCollection(name, options)命令。options参数可以省略。

**示例**：
```
> db.createCollection("col_test") // 创建一个col_test集合
```

* 删除集合

**语法**：
使用db.{COLLECTION_NAME}.drop()命令

**示例**：
```
> db.col_test.drop() // 删除col_test集合
```

* 显示当前使用的集合

 使用show tables或者show collections命令
```
> show collections // 显示所有的集合
```

## 4.3 文档（数据行）的操作

* 插入文档

**语法**：

使用db.{COLLECTION_NAME}.insert(document)命令或者db.{COLLECTION_NAME}.save(document)命令

**示例**：
```
> db.col_book.save({
    "title":"Head First Java",
    "price":10
})
```
或者先将json对象赋给一个变量，再插入。

```
> var book = {
    "title":"Head First Java",
    "price":10
};
> db.col_book.save(book)
```

>> insert()和save()的区别是，如果不指定\_id字段save()方法类似于insert()方法。如果指定\_id字段，则会替换该_id对应的文档数据。

* 查询文档

**语法**：

使用db.{COLLECTION_NAME}.find(query, projection)命令

query:使用查询操作符组成的查询条件，可选。若省略该参数，则返回集合中的所有文档。     
projection ：使用投影操作符指定的返回的键，可选。若省略该参数，则查询时返回文档中所有键值。

**示例**：

```
> db.col_book.find() // 返回集合col_book所有文档，文档中的所有字段
```

下面的示例，查询所有title为"Head First Java"的书，只返回title键。
```
> var query = {
    "title": "Head First Java"
};
> var projection = {
    "title": 1
};
> db.col_book.find(query, projection);
{ "_id" : ObjectId("5d99d7aade304a0a030ff95e"), "title" : "Head First Java" }
```
可以看到，上面的返回结果中出现了\_id这个键。存储在mongodb中的每个文档都有一个默认的主键\_id，作用类似于mysql的主键，用来唯一标志每个文档。但是，在数据类型和生成方式上不一样。 \_id可以在插入文档时指定，可以是任何类型；如果不指定，则默认生成一个数据类型为ObjectId的主键。默认生成的\_id主键并不是自增的，而是通过[snowflake算法](https://segmentfault.com/a/1190000011282426)类似的算法生成的，定位于分布式存储系统，其格式如下图所示。
![](https://tva1.sinaimg.cn/large/006y8mN6gy1g7phdjk2r8j30hd07awf8.jpg)


* 更新文档

**语法**：
```
db.{COLLECTION_NAME}.update(
   <query>,
   <update>,
   {
     upsert: <boolean>,
     multi: <boolean>,
     writeConcern: <document>
   }
)
```
query：update的查询条件，类似于sql update语句后面的where条件部分。    
update：需要更新的键和一些有关的操作符（如$set,$inc...）等，可以理解为sql update语句set后面的部分    
upsert：如果不存在update的记录，是否插入新的文档。true为插入，false为不插入，默认是false。该参数可选。    
multi：是否只更新找到的第一条记录，默认为false。若为true，则把按条件查出来多条记录全部更新。   
writeConcern：可选，抛出异常的级别

**示例**：

```
> var query = {
    "title": "Head First Mysql"
};
> var update = {
    "$set": {
        "price": 100
    }
};
> var params = {
    "multi": true
};
> db.col_book.update(query, update, params)
WriteResult({ "nMatched" : 2, "nUpserted" : 0, "nModified" : 2 })
```

或者通过指定\_id键方式的save()命令来替换整个文档。

```
> var data = {
    "_id": ObjectId("5d99d746ac39b2706ef53073"),
    "title": "Head First Mysql",
    "price": 2000
};
> db.col_book.save(data)
WriteResult({ "nMatched" : 1, "nUpserted" : 0, "nModified" : 1 })
```

* 删除文档

**语法**：
```
db.{COLLECTION_NAME}.remove(
   <query>,
   {
     justOne: <boolean>,
     writeConcern: <document>
   }
)
```
query：删除条件。可选。   
justOne：是否只删除符合条件的一个文档，可选。默认为false，即默认删除符合条件的所有文档。   
writeConcern：抛出异常的级别，可选。   

**示例**： 

下面的代码，删除title为"Head First Mysql"的其中一个文档。
```
> var query = {
    "title": "Head First Mysql"
};
> var params = {
    "justOne": true
}
> db.col_book.remove(query, params)
```

如果要删除某个集合下的所有文档，则可以使用db.{COLLECTION_NAME}.remove({})命令


# 参考
[MongoDB 进阶模式设计](http://www.mongoing.com/mongodb-advanced-pattern-design)   
[MongoDB Vs MySQL: The Differences Explained](https://blog.panoply.io/mongodb-and-mysql)   
[MongoDB开发规范](https://www.cnblogs.com/ae6623/p/6183674.html)  