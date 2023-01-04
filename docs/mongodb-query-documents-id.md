# 在 MongoDB 中使用文档 ID 查询文档

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/mongodb-query-documents-id>

## 1.概观

在本教程中，我们将看看如何使用 [MongoDB](/web/20220810175714/https://www.baeldung.com/java-mongodb) 中的文档 ID 执行查询操作。MongoDB 提供了一个 [`find`](/web/20220810175714/https://www.baeldung.com/mongodb-find) 操作符来查询集合中的文档。

在本教程中，我们将首先在 MongoDB Shell 查询中使用文档 ID 查询文档，然后使用 Java 驱动程序代码。

## 2.MongoDB 文档的文档 ID 是什么？

就像任何其他数据库管理系统一样，MongoDB 要求存储在集合中的每个文档都有一个惟一的标识符。这个唯一标识符充当集合的主键。

在 MongoDB 中，ID 由 12 个字节组成:

*   一个 4 字节的时间戳，表示自 Unix 纪元以来以秒为单位的 ID 创建
*   一个 5 字节随机生成的值，对于机器和过程是唯一的
*   一个 3 字节递增计数器

在 MongoDB 中，**ID 存储在名为`_id`的字段中，由客户端生成。**因此，应该在将文档发送到数据库之前生成 ID。在客户端，我们可以使用驱动程序生成的 ID，也可以生成自定义 ID。

唯一标识符存储在 [`ObjectId`](https://web.archive.org/web/20220810175714/https://www.mongodb.com/docs/manual/reference/method/ObjectId/) 类中。这个类提供了方便的方法来获取存储在 ID 中的数据，而不需要实际解析它。如果在插入文档时没有指定`_id`，MongoDB 将添加`_id`字段并为文档分配一个唯一的`ObjectId`。

## 3.数据库初始化

首先，让我们建立一个新的数据库`baeldung`和一个样本集合`vehicle`:

```
use baeldung;
db.createCollection("vehicle");
```

此外，让我们使用`insertMany`方法向集合中添加一些文档:

```
db.vehicle.insertMany([
{
    "companyName":"Skoda", 
    "modelName":"Octavia",
    "launchYear":2016,
    "type":"Sports",
    "registeredNo":"SKO 1134"
},
{ 
    "companyName":"BMW",
    "modelName":"X5",
    "launchYear":2020,
    "type":"SUV",
    "registeredNo":"BMW 3325"
},
{
    "companyName":"Mercedes",
    "modelName":"Maybach",
    "launchYear":2021,
    "type":"Luxury",
    "registeredNo":"MER 9754"
}]);
```

如果插入成功，上面的命令将打印一个 JSON，如下所示:

```
{
    "acknowledged" : true,
    "insertedIds" : [
        ObjectId("62d01d17cdd1b7c8a5f945b9"),
        ObjectId("62d01d17cdd1b7c8a5f945ba"),
        ObjectId("62d01d17cdd1b7c8a5f945bb")
    ]
}
```

我们已经成功地建立了数据库和集合。我们将在所有的例子中使用这个数据库和集合。

## 4.使用 MongoDB Shell

我们将利用`db.collection.find(query, projection)`方法从 MongoDB 中查询文档。

首先，让我们编写一个将返回所有`vehicle`集合文档的查询:

```
db.vehicle.find({});
```

以上查询返回所有文档:

```
{ "_id" : ObjectId("62d01d17cdd1b7c8a5f945b9"), "companyName" : "Skoda",
    "modelName" : "Octavia", "launchYear" : 2016, "type" : "Sports", "registeredNo" : "SKO 1134" }
{ "_id" : ObjectId("62d01d17cdd1b7c8a5f945ba"), "companyName" : "BMW",
    "modelName" : "X5", "launchYear" : 2020, "type" : "SUV", "registeredNo" : "BMW 3325" }
{ "_id" : ObjectId("62d01d17cdd1b7c8a5f945bb"), "companyName" : "Mercedes",
    "modelName" : "Maybach", "launchYear" : 2021, "type" : "Luxury", "registeredNo" : "MER 9754" }
```

此外，让我们编写一个查询，使用上面结果中返回的 ID 获取`vehicle`集合文档:

```
db.vehicle.find(
{
    "_id": ObjectId("62d01d17cdd1b7c8a5f945b9")
});
```

以上查询返回`_id`等于`ObjectId(“62d01d17cdd1b7c8a5f945b9”)`的`vehicle`收款单:

```
{ "_id" : ObjectId("62d01d17cdd1b7c8a5f945b9"), "companyName" : "Skoda",
    "modelName" : "Octavia", "launchYear" : 2016, "type" : "Sports", "registeredNo" : "SKO 1134" }
```

此外，我们可以使用带有`in`查询操作符的 id 来检索多个`vehicle`集合文档:

```
db.vehicle.find(
{
    "_id": {
        $in: [
            ObjectId("62d01d17cdd1b7c8a5f945b9"),
            ObjectId("62d01d17cdd1b7c8a5f945ba"),
            ObjectId("62d01d17cdd1b7c8a5f945bb")
        ]
    }
});
```

上述查询返回在`in`运算符中查询到的 id 的所有`vehicle`收款单据:

```
{ "_id" : ObjectId("62d01d17cdd1b7c8a5f945b9"), "companyName" : "Skoda",
    "modelName" : "Octavia", "launchYear" : 2016, "type" : "Sports", "registeredNo" : "SKO 1134" }
{ "_id" : ObjectId("62d01d17cdd1b7c8a5f945ba"), "companyName" : "BMW",
    "modelName" : "X5", "launchYear" : 2020, "type" : "SUV", "registeredNo" : "BMW 3325" }
{ "_id" : ObjectId("62d01d17cdd1b7c8a5f945bb"), "companyName" : "Mercedes",
    "modelName" : "Maybach", "launchYear" : 2021, "type" : "Luxury", "registeredNo" : "MER 9754" }
```

同样，任何一个[查询操作符](https://web.archive.org/web/20220810175714/https://www.mongodb.com/docs/manual/reference/operator/query/)都可以被用作带有要查询的 id 的`find()`方法的过滤器。

另外，需要注意的是，当使用`_id`字段查询文档时，文档 ID 字符串值应该被指定为`ObjectId()`而不是`String`。

让我们尝试使用 ID 作为`String`值来查询一个现有的文档:

```
db.vehicle.find(
{
    "_id": "62d01d17cdd1b7c8a5f945b9"
});
```

不幸的是，上面的查询不会返回任何文档，因为不存在任何 ID 为`String`值`62d01d17cdd1b7c8a5f945b9`的文档。

## 5.使用 Java 驱动程序

到目前为止，我们已经学习了如何通过 MongoDB Shell 使用 ID 查询文档。现在让我们使用 MongoDB Java 驱动程序实现同样的功能。

在执行更新操作之前，让我们首先连接到`baeldung`数据库中的`vehicle`集合:

```
MongoClient mongoClient = new MongoClient("localhost", 27017);
MongoDatabase database = mongoClient.getDatabase("baeldung");
MongoCollection<Document> collection = database.getCollection("vehicle");
```

在这里，在这种情况下，我们连接到 MongoDB，它运行在本地主机的默认端口 27017 上。

首先，让我们编写使用 ID 查询文档的代码:

```
Bson filter = Filters.eq("_id", new ObjectId("62d01d17cdd1b7c8a5f945b9"));
FindIterable<Document> documents = collection.find(filter);

MongoCursor<Document> cursor = documents.iterator();
while (cursor.hasNext()) {
    System.out.println(cursor.next());
}
```

这里，我们将一个`Bson`过滤器作为参数传递给带有要查询的`_id`字段的`find()`方法。上面的代码片段将返回`vehicle`托收单据，其中`_id`等于`ObjectId(“62d01d17cdd1b7c8a5f945b9”)`。

此外，让我们编写一个代码片段来查询具有多个 id 的文档:

```
Bson filter = Filters.in("_id", new ObjectId("62d01d17cdd1b7c8a5f945b9"),
  new ObjectId("62d01d17cdd1b7c8a5f945ba"));
FindIterable<Document> documents = collection.find(filter);

MongoCursor<Document> cursor = documents.iterator();
while (cursor.hasNext()) {
    System.out.println(cursor.next());
}
```

以上查询返回查询 id 的所有`vehicle`收款单据。

最后，让我们尝试使用驱动程序生成的 ID 查询`vehicle`集合:

```
Bson filter = Filters.eq("_id", new ObjectId());
FindIterable<Document> documents = collection.find(filter);

MongoCursor<Document> cursor = documents.iterator();
while (cursor.hasNext()) {
    System.out.println(cursor.next());
}
```

上面的查询不会返回任何文档，因为带有新生成的 ID 的文档不会存在于`vehicle`集合中。

## 6.结论

在本文中，我们学习了在 MongoDB 中使用文档 ID 查询文档。首先，我们在 MongoDB Shell 查询中研究了这些用例，然后我们讨论了相应的 Java 驱动程序代码。

所有这些例子和代码片段的实现都可以在 GitHub 的[上找到。](https://web.archive.org/web/20220810175714/https://github.com/eugenp/tutorials/tree/master/persistence-modules/java-mongodb-3)