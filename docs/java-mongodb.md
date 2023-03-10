# 使用 Java 的 MongoDB 指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-mongodb>

## 1。概述

在本文中，我们将看看如何将 [MongoDB](https://web.archive.org/web/20220625180307/https://www.mongodb.com/) 与一个独立的 Java 客户端相集成，这是一个非常流行的 NoSQL 开源数据库。

MongoDB 是用 C++编写的，有很多可靠的特性，比如 map-reduce、自动分片、复制、高可用性等等。

## 2。MongoDB

让我们从 MongoDB 本身的几个关键点开始:

*   将数据存储在类似于 [JSON](https://web.archive.org/web/20220625180307/https://www.w3schools.com/js/js_json_intro.asp) 的文档中，这些文档可以有不同的结构
*   使用动态模式，这意味着我们可以在不预先定义任何东西的情况下创建记录
*   只需添加新字段或删除现有字段，就可以更改记录的结构

上面提到的数据模型使我们能够轻松地表示层次关系、存储数组和其他更复杂的结构。

## 3。术语

如果我们可以将 MongoDB 中的概念与关系数据库结构进行比较，那么理解它们会变得更容易。

让我们看看 Mongo 和传统 MySQL 系统之间的相似之处:

*   MySQL 中的`Table`变成了 Mongo 中的`Collection`
*   `Row`变成了`Document`
*   `Column`变成了`Field`
*   `Joins`定义为`linking`和`embedded`文件

当然，这是看待 MongoDB 核心概念的一种简单方式，但仍然很有用。

现在，让我们深入实现来理解这个强大的数据库。

## 4。 **美芬依存**

我们需要从定义 MongoDB 的 Java 驱动程序的依赖性开始:

```java
<dependency>
    <groupId>org.mongodb</groupId>
    <artifactId>mongo-java-driver</artifactId>
    <version>3.4.1</version>
</dependency> 
```

要检查是否有新版本的库发布-[在这里跟踪发布版本](https://web.archive.org/web/20220625180307/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.mongodb%22%20AND%20a%3A%22mongo-java-driver%22)。

## 5。 **利用`MongoDB`**

现在，让我们开始用 Java 实现 Mongo 查询。我们将从基本的 CRUD 操作开始，因为它们是最好的开始。

### 5.1。`MongoClient`与建立联系

首先，让我们连接一个 MongoDB 服务器。对于版本> = 2.10.0，我们将使用`MongoClient`:

```java
MongoClient mongoClient = new MongoClient("localhost", 27017);
```

对于旧版本，使用`Mongo` 类:

```java
Mongo mongo = new Mongo("localhost", 27017);
```

### 5.2。连接到数据库

现在，让我们连接到数据库。有趣的是，我们不需要创建一个。当 Mongo 发现数据库不存在时，它会为我们创建一个数据库:

```java
DB database = mongoClient.getDB("myMongoDb");
```

有时，默认情况下，MongoDB 运行在认证模式下。在这种情况下，我们需要在连接到数据库时进行身份验证。

我们可以按照下面的方式来做:

```java
MongoClient mongoClient = new MongoClient();
DB database = mongoClient.getDB("myMongoDb");
boolean auth = database.authenticate("username", "pwd".toCharArray());
```

### 5.3。显示现有数据库

让我们显示所有现有的数据库。当我们想要使用命令行时，显示数据库的语法类似于 MySQL:

```java
show databases;
```

在 Java 中，我们使用下面的代码片段显示数据库:

```java
mongoClient.getDatabaseNames().forEach(System.out::println);
```

输出将是:

```java
local      0.000GB
myMongoDb  0.000GB
```

以上，`local`是默认的 Mongo 数据库。

### 5.4。创造一个`Collection`

让我们从为数据库创建一个`Collection`(MongoDB 的等价表)开始。一旦我们连接到数据库，我们可以将`Collection`设为:

```java
database.createCollection("customers", null);
```

现在，让我们显示当前数据库的所有现有集合:

```java
database.getCollectionNames().forEach(System.out::println);
```

输出将是:

```java
customers
```

### 5.5。T3`Save – Insert`

`save`操作有保存或更新语义:如果有`id`，它执行`update`，如果没有，它执行`insert`。

当我们遇到新客户时:

```java
DBCollection collection = database.getCollection("customers");
BasicDBObject document = new BasicDBObject();
document.put("name", "Shubham");
document.put("company", "Baeldung");
collection.insert(document);
```

实体将被插入到数据库中:

```java
{
    "_id" : ObjectId("33a52bb7830b8c9b233b4fe6"),
    "name" : "Shubham",
    "company" : "Baeldung"
}
```

接下来，我们将看看具有`update`语义的相同操作——`save`。

### 5.6。T3`Save – Update`

现在让我们看看带有`update`语义的`save`，对现有客户进行操作:

```java
{
    "_id" : ObjectId("33a52bb7830b8c9b233b4fe6"),
    "name" : "Shubham",
    "company" : "Baeldung"
}
```

现在，当我们`save`现有客户时，我们将更新它:

```java
BasicDBObject query = new BasicDBObject();
query.put("name", "Shubham");

BasicDBObject newDocument = new BasicDBObject();
newDocument.put("name", "John");

BasicDBObject updateObject = new BasicDBObject();
updateObject.put("$set", newDocument);

collection.update(query, updateObject);
```

数据库将如下所示:

```java
{
    "_id" : ObjectId("33a52bb7830b8c9b233b4fe6"),
    "name" : "John",
    "company" : "Baeldung"
}
```

正如你所看到的，在这个特殊的例子中，`save`使用了`update`的语义，因为我们使用了带有给定`_id`的对象。

### 5.7。从`Collection` 中读取一个`Document`

让我们通过查询在`Collection`中搜索`Document`:

```java
BasicDBObject searchQuery = new BasicDBObject();
searchQuery.put("name", "John");
DBCursor cursor = collection.find(searchQuery);

while (cursor.hasNext()) {
    System.out.println(cursor.next());
}
```

它将显示我们目前在`Collection`中仅有的`Document`:

```java
[
    {
      "_id" : ObjectId("33a52bb7830b8c9b233b4fe6"),
      "name" : "John",
      "company" : "Baeldung"
    }
]
```

### 5.8。`Delete`一个`Document`一个

让我们前进到最后一个 CRUD 操作，删除:

```java
BasicDBObject searchQuery = new BasicDBObject();
searchQuery.put("name", "John");

collection.remove(searchQuery);
```

随着上述命令的执行，我们唯一的`Document`将从`Collection`中移除。

## 6。结论

本文是从 Java 使用 MongoDB 的快速介绍。

所有这些例子和代码片段的实现都可以在 GitHub 上找到[——这是一个基于 Maven 的项目，所以它应该很容易导入和运行。](https://web.archive.org/web/20220625180307/https://github.com/eugenp/tutorials/tree/master/persistence-modules/java-mongodb)