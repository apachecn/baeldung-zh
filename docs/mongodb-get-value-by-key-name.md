# 通过键值名从 MongoDB 中检索值

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/mongodb-get-value-by-key-name>

## 1.概观

在本教程中，我们将学习如何通过键名从 [MongoDB](https://web.archive.org/web/20220524062711/https://www.mongodb.com/) 中检索一个值。我们将探索 MongoDB 的各种方法，根据应用的过滤器获取文档的关键字段名称。首先，我们将使用 [`find`](https://web.archive.org/web/20220524062711/https://www.mongodb.com/docs/manual/reference/method/db.collection.find/) 或`findone`方法获取所需的数据，然后使用 [`aggregation`](https://web.archive.org/web/20220524062711/https://www.mongodb.com/docs/manual/aggregation/) 方法。这里，我们将在 MongoDB shell 查询和 Java 驱动程序代码中编写查询。

让我们看看在 MongoDB 中通过字段名检索值的不同方法。

## 2.数据库初始化

首先，我们需要建立一个新的数据库`baeldung`和一个新的集合`travel`:

```java
use baeldung;
db.createCollection(travel);
```

现在让我们使用 MongoDB 的`insertMany`方法向集合中添加一些虚拟数据:

```java
db.travel.insertMany([
{ 
    "passengerId":145,
    "passengerName":"Nathan Green",
    "passengerAge":25,
    "sourceStation":"London",
    "destinationStation":"Birmingham",
    "seatType":"Slepper",
    "emailAddress":"[[email protected]](/web/20220524062711/https://www.baeldung.com/cdn-cgi/l/email-protection)"
},
{ 
    "passengerId":148,
    "passengerName":"Kevin Joseph",
    "passengerAge":28,
    "sourceStation":"Manchester",
    "destinationStation":"London",
    "seatType":"Slepper",
    "emailAddress":"[[email protected]](/web/20220524062711/https://www.baeldung.com/cdn-cgi/l/email-protection)"
},
{ 
    "passengerId":154,
    "passengerName":"Sheldon burns",
    "passengerAge":26,
    "sourceStation":"Cambridge",
    "destinationStation":"Leeds",
    "seatType":"Slepper",
    "emailAddress":"[[email protected]](/web/20220524062711/https://www.baeldung.com/cdn-cgi/l/email-protection)"
},
{ 
    "passengerId":168,
    "passengerName":"Jack Ferguson",
    "passengerAge":24,
    "sourceStation":"Cardiff",
    "destinationStation":"Coventry",
    "seatType":"Slepper",
    "emailAddress":"[[email protected]](/web/20220524062711/https://www.baeldung.com/cdn-cgi/l/email-protection)"
}
]);
```

上面的`insertMany`查询将返回以下 JSON:

```java
{
    "acknowledged" : true,
    "insertedIds" : [
        ObjectId("623d7f079d55d4e137e47825"),
	ObjectId("623d7f079d55d4e137e47826"),
	ObjectId("623d7f079d55d4e137e47827"),
        ObjectId("623d7f079d55d4e137e47828")
    ]
}
```

到目前为止，我们已经将虚拟数据插入到集合`travel`中。

## 3.使用`find`方法

`find`方法在集合中查找并返回符合指定查询标准的文档。如果多个文档符合条件，那么它将根据文档在磁盘上的顺序返回所有文档。此外，在 MongoDB 中，`find`方法支持查询中的参数投影。如果我们在`find`方法中指定一个投影参数，它将返回所有只包含投影字段的文档。

**需要注意的一个要点是，`_id`字段总是包含在响应中，除非被显式删除。**

为了进行演示，让我们研究一下 shell 查询来投射一个关键字段:

```java
db.travel.find({},{"passengerId":1}).pretty();
```

对上述查询的响应将是:

```java
{ "_id" : ObjectId("623d7f079d55d4e137e47825"), "passengerId" : 145 }
{ "_id" : ObjectId("623d7f079d55d4e137e47826"), "passengerId" : 148 }
{ "_id" : ObjectId("623d7f079d55d4e137e47827"), "passengerId" : 154 }
{ "_id" : ObjectId("623d7f079d55d4e137e47828"), "passengerId" : 168 }
```

这里，在这个查询中，我们简单地投影了`passengerId.` ，现在让我们看看排除了`_id`的关键字段:

```java
db.travel.find({},{"passengerId":1,"_id":0}).pretty();
```

上述查询将有以下响应:

```java
{ "passengerId" : 145 }
{ "passengerId" : 148 }
{ "passengerId" : 154 }
{ "passengerId" : 168 }
```

这里，在这个查询中，我们从响应预测中排除了`_id`字段。让我们看看上面查询的 Java 驱动程序代码:

```java
MongoClient mongoClient = new MongoClient("localhost", 27017);
DB database = mongoClient.getDB("baeldung");
DBCollection collection = database.getCollection("travel");
BasicDBObject queryFilter = new BasicDBObject();
BasicDBObject projection = new BasicDBObject();
projection.put("passengerId", 1);
projection.put("_id", 0);
DBCursor dbCursor = collection.find(queryFilter, projection);
while (dbCursor.hasNext()) {
    System.out.println(dbCursor.next());
}
```

在上面的代码中，首先，我们创建了一个与运行在端口`27017`上的本地 mongo 服务器的`MongoClient`连接。接下来，我们使用了`find`方法，它有两个参数，即`queryFilter,`和投影。查询`DBObject`包含我们需要获取数据的过滤器。在这里，我们使用`DBCursor`打印旅行文件的所有投影字段。

## 4.使用`aggregation`方法

MongoDB 中的`aggregation`操作处理数据记录和文档，并返回计算结果。它从各种文档中收集值，并在对分组数据执行不同类型的操作(如求和、平均、最小值、最大值等)之前将它们分组。

当我们需要进行更复杂的聚合时，可以使用 MongoDB 聚合管道。**聚合管道是与 MongoDB 查询语法相结合以产生聚合结果的阶段集合。**

让我们研究一下聚合查询，通过键名检索值:

```java
db.travel.aggregate([
{
    "$project":{
        "passengerId":1
    }
}
]).pretty();
```

对上述聚合查询的响应将是:

```java
{ "_id" : ObjectId("623d7f079d55d4e137e47825"), "passengerId" : 145 }
{ "_id" : ObjectId("623d7f079d55d4e137e47826"), "passengerId" : 148 }
{ "_id" : ObjectId("623d7f079d55d4e137e47827"), "passengerId" : 154 }
{ "_id" : ObjectId("623d7f079d55d4e137e47828"), "passengerId" : 168 }
```

在本例中，我们使用了聚合管道的`$project`阶段。`$project`指定要包含或排除的字段。在我们的查询中，我们只将 passengerId 传递到投影阶段。

让我们看看上面查询的 Java 驱动程序代码:

```java
ArrayList<Document> response = new ArrayList<>();
ArrayList<Document> pipeline = new ArrayList<>(Arrays.asList(new Document("$project", new Document("passengerId", 1L))));
database = mongoClient.getDatabase("baeldung");
database.getCollection("travel").aggregate(pipeline).allowDiskUse(true).into(response);
System.out.println("response:- " + response);
```

我们还可以按照以下方式编写聚合管道:

```java
ArrayList<Document> response = new ArrayList<>();
ArrayList<Bson> pipeline = new ArrayList<>(Arrays.asList(
  project(fields(Projections.exclude("_id"), Projections.include("passengerId")))));
MongoDatabase database = mongoClient.getDatabase("baeldung");
database.getCollection("travel").aggregate(pipeline).allowDiskUse(true).into(response);
System.out.println("response:- "+response);
```

我们用 java 驱动程序代码创建了一个聚合管道，并将项目阶段设置为只包含`passengerId`字段。最后，我们将聚合管道传递给`aggregate`方法来检索数据。

## 5.结论

在本文中，我们学习了在 MongoDB 中通过键名检索值。我们探索了 MongoDB 获取数据的不同方法。首先，我们使用`find`方法检索数据，然后使用`aggregate`方法。我们研究了 MongoDB 中字段投影的用例。简而言之，我们使用 Mongo shell 查询和 Java 驱动程序代码实现了字段的投影。

我们可以在 GitHub 上找到所有案例[的实现。](https://web.archive.org/web/20220524062711/https://github.com/eugenp/tutorials/tree/master/persistence-modules/java-mongodb-2)