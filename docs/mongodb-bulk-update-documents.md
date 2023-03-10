# MongoDB 中文档的批量更新

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/mongodb-bulk-update-documents>

## 1.概观

在本教程中，我们将看看如何在 [MongoDB](https://web.archive.org/web/20220524063820/https://www.mongodb.com/) 中执行批量更新和插入操作。此外，MongoDB 提供 API 调用，允许在单个操作中插入或检索多个文档。MongoDB 使用`Array`或`Batch`接口，通过减少客户机和数据库之间的调用次数，极大地提高了数据库性能。

在本教程中，我们将研究使用 MongoDB Shell 和 Java 驱动程序代码的两种解决方案。

让我们深入研究如何在 MongoDB 中实现文档的批量更新。

## 2.数据库初始化

首先，我们需要连接到 mongo shell:

```java
mongo --host localhost --port 27017
```

现在，建立一个数据库`baeldung`和一个样本集合`populations`:

```java
use baeldung;
db.createCollection(populations);
```

让我们使用`insertMany`方法将一些样本数据添加到集合`populations`中:

```java
db.populations.insertMany([
{
    "cityId":1124,
    "cityName":"New York",
    "countryName":"United States",
    "continentName":"North America",
    "population":22
},
{
    "cityId":1125,
    "cityName":"Mexico City",
    "countryName":"Mexico",
    "continentName":"North America",
    "population":25
},
{
    "cityId":1126,
    "cityName":"New Delhi",
    "countryName":"India",
    "continentName":"Asia",
    "population":45
},
{
    "cityId":1134,
    "cityName":"London",
    "countryName":"England",
    "continentName":"Europe",
    "population":32
}]);
```

上面的`insertMany`查询将返回以下文档:

```java
{
    "acknowledged" : true,
    "insertedIds" : [
        ObjectId("623575049d55d4e137e477f6"),
        ObjectId("623575049d55d4e137e477f7"),
        ObjectId("623575049d55d4e137e477f8"),
        ObjectId("623575049d55d4e137e477f9")
    ]
}
```

这里，我们在上面的查询中插入了四个文档，以在 MongoDB 中执行所有类型的写批量操作。

数据库`baeldung`已经成功创建，所有需要的数据也已经插入到集合`populations`中，所以我们已经准备好执行批量更新。

## 3.使用 MongoDB Shell 查询

MongoDB 的批量操作构建器用于为单个集合批量构建写操作列表。我们可以用两种不同的方式初始化批量操作。方法`initializeOrderedBulkOp`用于在写操作的有序列表中执行批量操作。**`initializeOrderedBulkOp`的缺点之一是，如果在处理任何写操作时发生错误，MongoDB 将返回，而不处理列表中剩余的写操作。**

我们可以使用 insert、update、replace 和 remove 方法在单个 DB 调用中执行不同类型的操作。作为一个例子，让我们来看看使用 MongoDB shell 的批量写操作查询:

```java
db.populations.bulkWrite([
    { 
        insertOne :
            { 
                "document" :
                    {
                        "cityId":1128,
                        "cityName":"Kathmandu",
                        "countryName":"Nepal",
                        "continentName":"Asia",
                        "population":12
                    }
            }
    },
    { 
        insertOne :
            { 
                "document" :
                    {
                        "cityId":1130,
                        "cityName":"Mumbai",
                        "countryName":"India",
                        "continentName":"Asia",
                        "population":55
                    }
            }
    },
    { 
        updateOne :
            { 
                "filter" : 
                     { 
                         "cityName": "New Delhi"
                     },
                 "update" : 
                     { 
                         $set : 
                         { 
                             "status" : "High Population"
                         } 
                     }
            }
    },
    { 
        updateMany :
            { 
                "filter" : 
                     { 
                         "cityName": "London"
                     },
                 "update" : 
                     { 
                         $set : 
                         { 
                             "status" : "Low Population"
                         } 
                     }
            }
    },
    { 
        deleteOne :
            { 
                "filter" : 
                    { 
                        "cityName":"Mexico City"
                    } 
            }
    },
    { 
        replaceOne :
            {
                "filter" : 
                    { 
                        "cityName":"New York"
                    },
                 "replacement" : 
                    {
                        "cityId":1124,
                        "cityName":"New York",
                        "countryName":"United States",
                        "continentName":"North America",
                        "population":28
                    }
             }
    }
]);
```

上面的`bulkWrite`查询将返回以下文档:

```java
{
    "acknowledged" : true,
    "deletedCount" : 1,
    "insertedCount" : 2,
    "matchedCount" : 3,
    "upsertedCount" : 0,
    "insertedIds" : 
        {
            "0" : ObjectId("623575f89d55d4e137e477f9"),
            "1" : ObjectId("623575f89d55d4e137e477fa")
        },
    "upsertedIds" : {}
}
```

这里，在上面的查询中，我们执行了所有类型的写操作，即`insertOne`、`updateOne`、`deleteOne`、`replaceOne`。

首先，我们使用`insertOne`方法向集合中插入一个新文档。其次，我们用`updateOne` 更新了`cityName`“新德里”的文档。稍后，我们使用`deleteOne`方法根据过滤器从集合中删除一个文档。最后，我们使用`replaceOne `将一个完整的文档替换为过滤器`cityName` “New York”。

## 4.使用 Java 驱动程序

我们已经讨论了执行批量写操作的 MongoDB shell 查询。在创建批量写操作之前，让我们首先创建一个与数据库`baeldung`的集合`populations`的`MongoClient`连接:

```java
MongoClient mongoClient = new MongoClient("localhost", 27017);
MongoDatabase database = mongoClient.getDatabase("baeldung");
MongoCollection<Document> collection = database.getCollection("populations");
```

这里，我们创建了与 MongoDB 服务器的连接，运行在默认端口 27017 上。现在让我们使用 Java 代码实现相同的批量操作:

```java
List<WriteModel<Document>> writeOperations = new ArrayList<WriteModel<Document>>();
writeOperations.add(new InsertOneModel<Document>(new Document("cityId", 1128)
  .append("cityName", "Kathmandu")
  .append("countryName", "Nepal")
  .append("continentName", "Asia")
  .append("population", 12)));
writeOperations.add(new InsertOneModel<Document>(new Document("cityId", 1130)
  .append("cityName", "Mumbai")
  .append("countryName", "India")
  .append("continentName", "Asia")
  .append("population", 55)));
writeOperations.add(new UpdateOneModel<Document>(new Document("cityName", "New Delhi"),
  new Document("$set", new Document("status", "High Population"))
));
writeOperations.add(new UpdateManyModel<Document>(new Document("cityName", "London"),
  new Document("$set", new Document("status", "Low Population"))
));
writeOperations.add(new DeleteOneModel<Document>(new Document("cityName", "Mexico City")));
writeOperations.add(new ReplaceOneModel<Document>(new Document("cityId", 1124), 
  new Document("cityName", "New York").append("cityName", "United States")
    .append("continentName", "North America")
    .append("population", 28)));
BulkWriteResult bulkWriteResult = collection.bulkWrite(writeOperations);
System.out.println("bulkWriteResult:- " + bulkWriteResult);
```

这里，我们首先创建了一个`writeModel`列表，将所有不同类型的写操作添加到一个更新列表中。此外，我们在查询中使用了`InsertOneModel`、`UpdateOneModel`、`UpdateManyModel`、`DeleteOneModel`和`ReplaceOneModel`。最后，`bulkWrite`方法一次性执行了所有操作。

## 5.结论

在本文中，我们已经学习了使用不同种类的写操作在 MongoDB 中执行批量操作。我们在单个 DB 查询中执行了文档的插入、更新、删除和替换。此外，我们在 MongoDB 的批量更新中学习了`initializeOrderedBulkOp `的用例。

首先，我们研究了 MongoDB shell 查询中批量操作的用例，然后讨论了相应的 Java 驱动程序代码。

所有案例的例子可以在 GitHub 上找到[。](https://web.archive.org/web/20220524063820/https://github.com/eugenp/tutorials/tree/master/persistence-modules/java-mongodb-2)