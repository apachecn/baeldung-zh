# 检查 MongoDB 中的集合是否存在

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-check-collection-existence-mongodb>

## 1.概观

MongoDB 是一个 NoSQL 数据库，它将数据记录作为 [`BSON`](/web/20220524053703/https://www.baeldung.com/mongodb-bson) 文档存储到一个集合中。我们可以有多个数据库，每个数据库可以有一个或多个文档集合。

与关系数据库不同，MongoDB 使用插入的文档创建集合，而不需要任何结构定义。在本教程中，我们将学习检查集合是否存在的各种方法。我们将使用`collectionExists, createCollection, listCollectionNames,` 和 `count`方法来检查集合是否存在。

## 2.数据库连接

为了访问集合中的任何数据，我们首先需要建立与数据库的连接。让我们连接到本地运行在我们机器上的 MongoDB 数据库。

### 2.1.使用`MongoClient`创建连接

`[MongoClient](https://web.archive.org/web/20220524053703/https://docs.mongodb.com/drivers/java-drivers/) `是一个 Java 类，用于建立与 MongoDB 实例的连接:

```
MongoClient mongoClient = new MongoClient("localhost", 27017);
```

这里，我们连接到 MongoDB，它运行在本地主机的默认端口 27017 上。

### 2.2.连接到数据库

现在，让我们使用`MongoClient`对象来访问数据库。使用`MongoClient`访问数据库有两种方法。

首先，我们将使用`getDatabase`方法来访问`baeldung`数据库:

```
MongoDatabase database = mongoClient.getDatabase("baeldung");
```

我们也可以使用 Mongo Java 驱动程序的`getDB`方法连接数据库:

```
DB db = mongoClient.getDB("baeldung");
```

不推荐使用`getDB `方法，因此不推荐使用。

到目前为止，我们已经使用 MongoClient 建立了与 MongoDB 的连接，并进一步连接到了`baeldung`数据库。

让我们深入研究不同的方法来检查 MongoDB 中集合的存在性。

## 3.使用`DB`类

MongoDB Java 驱动程序提供同步和异步方法调用。为了连接到数据库，我们只需要指定数据库名称。如果数据库不存在，MongoDB 将自动创建一个。

`collectionExists`方法可用于检查集合是否存在:

```
MongoClient mongoClient = new MongoClient("localhost", 27017);
DB db = mongoClient.getDB("baeldung");
String testCollectionName = "student";
System.out.println("Collection Name " + testCollectionName + " " + db.collectionExists(testCollectionName));
```

这里，如果集合存在，`collectionExists`方法将返回`true`，否则返回 false。

**MongoDB Java 驱动程序的`com.mongodb.DB` API 从 3.x 版本开始被弃用，但仍然可以访问。**因此，不建议在新项目中使用`DB`类。

## 4.使用`MongoDatabase`类

`com.mongodb.client.MongoDatabase`是 Mongo 3.x 及以上版本的更新 API。与 DB 类不同，MongoDatabase 类不提供任何特定的方法来检查集合的存在性。但是，我们可以使用各种方法来获得想要的结果。

### 4.1.使用`createCollection`方法

`createCollection`方法在 MongoDB 中创建新的集合。但是我们也可以用它来检查集合是否存在:

```
String databaseName="baeldung";
MongoDatabase database = mongoClient.getDatabase(databaseName);
String testCollectionName = "student";
try {
    database.createCollection(testCollectionName);
} catch (Exception exception) {
    System.err.println("Collection:- "+testCollectionName +" already Exists");
}
```

**上面的代码将创建一个新的集合“`student”`，如果它还没有出现在数据库中的话。**如果集合已经存在，`createCollection`方法将抛出一个异常。

不推荐这种方法，因为它会在数据库中创建一个新集合。

### 4.2.使用`listCollectionNames`方法

`listCollectionNames`方法列出了数据库中所有集合的名称。因此，我们可以用这种方法来解决集合存在的问题。

现在让我们看看使用 Java 驱动程序代码的`listCollectionNames` 方法的示例代码:

```
String databaseName="baeldung";
MongoDatabase database = mongoClient.getDatabase(databaseName);
String testCollectionName = "student";
boolean collectionExists = database.listCollectionNames()
  .into(new ArrayList()).contains(testCollectionName);
System.out.println("collectionExists:- " + collectionExists);
```

这里，我们遍历了数据库中所有集合名称的列表`baeldung.`，对于每一次出现，我们将集合字符串名称与`testCollectionName`进行匹配。如果匹配成功，它将返回`true`，否则返回`false`。

### 4.3.使用`count`方法

`MongoCollection`的`count`方法统计集合中存在的文档数量。

作为一种变通方法，我们可以使用这个方法来检查集合是否存在。下面是相同的 Java 代码片段:

```
String databaseName="baeldung";
MongoDatabase database = mongoClient.getDatabase(databaseName);
String testCollectionName = "student";
MongoCollection<Document> collection = database.getCollection(testCollectionName);
Boolean collectionExists = collection.count() > 0 ? true : false;
System.out.println("collectionExists:- " + collectionExists);
Boolean expectedStatus = false;
assertEquals(expectedStatus, collectionExists);
```

如果集合中没有任何数据，这个方法就不起作用，在这种情况下，它将返回 0，但是集合中存在空数据。

## 5。结论

在本文中，我们探索了使用`MongoDatabase`和`DB`类方法检查集合是否存在的各种方法。

简而言之，推荐使用`com.mongodb.DB`类的`collectionExists`方法和`com.mongodb.client.MongoDatabase`的`listCollectionNames`方法来检查集合的存在性。

与往常一样，所有示例的源代码和代码片段都可以在 GitHub 上的[处获得。](https://web.archive.org/web/20220524053703/https://github.com/eugenp/tutorials/tree/master/persistence-modules/java-mongodb)