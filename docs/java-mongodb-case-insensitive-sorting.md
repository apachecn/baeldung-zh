# MongoDB 中不区分大小写的排序

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-mongodb-case-insensitive-sorting>

## 1.概观

默认情况下， [MongoDB](/web/20221011171933/https://www.baeldung.com/java-mongodb) 引擎在对提取的数据进行排序时会考虑字符大小写。**通过指定`Aggregations`或`Collations`，可以执行不区分大小写的排序查询。**

在这个简短的教程中，我们将研究使用 MongoDB Shell 和 Java 的两种解决方案。

## 2.设置环境

首先，我们需要运行一个 MongoDB 服务器。让我们使用一个 Docker 图像:

```java
$ docker run -d -p 27017:27017 --name example-mongo mongo:latest
```

这将创建一个名为“`example-mongo`”的新临时 Docker 容器，公开端口`27017`。现在，我们需要用测试解决方案所需的数据创建一个基本的 Mongo 数据库。

首先，让我们打开容器内部的 Mongo Shell:

```java
$ docker exec -it example-mongo mongosh
```

进入 shell 后，让我们切换上下文，进入名为“`sorting`”的数据库:

```java
> use sorting
```

最后，让我们插入一些数据来尝试我们的排序操作:

```java
> db.users.insertMany([
  {name: "ben", surname: "ThisField" },
  {name: "aen", surname: "Does" },
  {name: "Aen", surname: "Not" },
  {name: "Ben", surname: "Matter" },
])
```

我们在一些文档的`name`字段中插入了类似的值。唯一的区别是第一个字母的大小写。至此，数据库创建完毕，数据被适当地插入，所以我们已经准备好行动了。

## 3.默认排序

让我们运行没有定制的标准查询:

```java
> db.getCollection('users').find({}).sort({name:1})
```

返回的数据将根据情况进行排序。这意味着，例如，**大写字符"`B”`将在小写字符"`a”`** 之前被考虑:

```java
[
  {
    _id: ..., name: 'Aen', surname: 'Not'
  },
  {
    _id: ..., name: 'Ben', surname: 'Matter'
  },
  {
    _id: ..., name: 'aen', surname: 'Does'
  },
  {
    _id: ..., name: 'ben', surname: 'ThisField'
  }
]
```

现在让我们看看如何使我们的排序不区分大小写，以便`Ben`和`ben`一起出现。

## 4.Mongo Shell 中不区分大小写的排序

### 4.1.使用`Collation`进行分类

让我们尝试使用 [MongoDB 排序规则](https://web.archive.org/web/20221011171933/https://docs.mongodb.com/manual/reference/collation/)。仅在 MongoDB 3.4 和后续版本中可用，它支持特定于语言的字符串比较规则。

**Collation ICU`locale`参数驱动数据库如何进行排序。**让我们用`“en”`(英语)`locale:`

```java
> db.getCollection('users').find({}).collation({locale: "en"}).sort({name:1})
```

这将生成按字母分类的名称输出:

```java
[
  {
    _id: ..., name: 'aen', surname: 'Does'
  },
  {
    _id: ..., name: 'Aen', surname: 'Not'
  },
  {
    _id: ..., name: 'ben', surname: 'ThisField'
  },
  {
    _id: ..., name: 'Ben', surname: 'Matter'
  }
]
```

### 4.2.使用`Aggregation`进行分类

现在让我们使用`Aggregation`函数:

```java
> db.getCollection('users').aggregate([{
        "$project": {
            "name": 1,
            "surname": 1,
            "lowerName": {
                "$toLower": "$name"
            }
        }
    },
    {
        "$sort": {
            "lowerName": 1
        }
    }
])
```

使用 [`$project`](https://web.archive.org/web/20221011171933/https://docs.mongodb.com/manual/reference/operator/aggregation/project/) 功能，我们添加一个`lowerName`字段作为`name`字段的小写版本。这允许我们使用该字段进行排序。它将为我们提供一个带有附加字段的结果对象，按照期望的排序顺序:

```java
[
  {
    _id: ..., name: 'aen', surname: 'Does', lowerName: 'aen'
  },
  {
    _id: ..., name: 'Aen', surname: 'Not', lowerName: 'aen'
  },
  {
    _id: ..., name: 'ben', surname: 'ThisField', lowerName: 'ben'
  },
  {
    _id: ..., name: 'Ben', surname: 'Matter', lowerName: 'ben'
  }
]
```

## 5.用 Java 进行不区分大小写的排序

让我们试着用 Java 实现同样的方法。

### 5.1.配置样板代码

让我们首先添加 [mongo-java-driver](https://web.archive.org/web/20221011171933/https://search.maven.org/artifact/org.mongodb/mongo-java-driver) 依赖关系:

```java
<dependency>
    <groupId>org.mongodb</groupId>
    <artifactId>mongo-java-driver</artifactId>
    <version>3.12.10</version>
</dependency>
```

然后，让我们使用`MongoClient`进行连接:

```java
MongoClient mongoClient = new MongoClient();
MongoDatabase db = mongoClient.getDatabase("sorting");
MongoCollection<Document> collection = db.getCollection("users");
```

### 5.2.在 Java 中使用排序规则进行排序

让我们看看如何用 Java 实现`“Collation”`解决方案:

```java
FindIterable<Document> nameDoc = collection.find().sort(ascending("name"))
  .collation(Collation.builder().locale("en").build());
```

这里，我们使用`“en”`语言环境构建了排序规则。然后，我们将创建的`[Collation](https://web.archive.org/web/20221011171933/https://mongodb.github.io/mongo-java-driver/3.6/javadoc/com/mongodb/client/model/Collation.html)`对象传递给`[FindIterable](https://web.archive.org/web/20221011171933/https://mongodb.github.io/mongo-java-driver/3.6/javadoc/com/mongodb/client/FindIterable.html)`对象的`collation`方法。

接下来，让我们使用`[MongoCursor](https://web.archive.org/web/20221011171933/https://mongodb.github.io/mongo-java-driver/3.6/javadoc/index.html?com/mongodb/client/MongoCursor.html)`逐个读取结果:

```java
MongoCursor cursor = nameDoc.cursor();
List expectedNamesOrdering = Arrays.asList("aen", "Aen", "ben", "Ben", "cen", "Cen");
List actualNamesOrdering = new ArrayList<>();
while (cursor.hasNext()) {
    Document document = cursor.next();
    actualNamesOrdering.add(document.get("name").toString());
}
assertEquals(expectedNamesOrdering, actualNamesOrdering);
```

### 5.3.在 Java 中使用聚合进行排序

我们还可以使用`Aggregation`对集合进行排序。让我们使用 Java API 重新创建我们的命令行版本。

首先，我们依靠`[project](https://web.archive.org/web/20221011171933/https://mongodb.github.io/mongo-java-driver/3.6/javadoc/com/mongodb/client/model/Aggregates.html#project-org.bson.conversions.Bson-)`方法创建一个`[Bson](https://web.archive.org/web/20221011171933/https://mongodb.github.io/mongo-java-driver/3.4/bson/)`对象。该对象还将包括通过使用 [`Projections`](https://web.archive.org/web/20221011171933/https://mongodb.github.io/mongo-java-driver/3.6/javadoc/com/mongodb/client/model/Projections.html) 类将姓名的每个字符转换成小写字母而计算出的`lowerName` 字段:

```java
Bson projectBson = project(
  Projections.fields(
    Projections.include("name","surname"),
    Projections.computed("lowerName", Projections.computed("$toLower", "$name")))); 
```

接下来，我们向`aggregate`方法提供一个包含前面代码片段的 [`Bson`](https://web.archive.org/web/20221011171933/https://mongodb.github.io/mongo-java-driver/3.4/bson/) 和`sort`方法的列表:

```java
AggregateIterable<Document> nameDoc = collection.aggregate(
  Arrays.asList(projectBson,
  sort(Sorts.ascending("lowerName")))); 
```

在这种情况下，和上一个例子一样，我们可以使用`[MongoCursor](https://web.archive.org/web/20221011171933/https://mongodb.github.io/mongo-java-driver/3.6/javadoc/index.html?com/mongodb/client/MongoCursor.html)`轻松地读取结果。

## 6.结论

在本文中，我们看到了如何对 MongoDB 集合执行简单的不区分大小写的排序。

我们在 MongoDB shell 中使用了`Aggregation`和`Collation`方法。最后，我们翻译了这些查询，并使用`mongo-java-driver`库提供了一个简单的 Java 实现。

和往常一样，GitHub 上的[提供了这篇文章的完整源代码。](https://web.archive.org/web/20221011171933/https://github.com/eugenp/tutorials/tree/master/persistence-modules/java-mongodb-2)