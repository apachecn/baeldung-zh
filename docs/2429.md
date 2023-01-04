# 如何在 MongoDB 中检查字段存在？

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/mongodb-check-field-exists>

## 1.概观

在这个简短的教程中，我们将看到如何在 [MongoDB](/web/20220926194125/https://www.baeldung.com/java-mongodb) 中检查字段的存在。

首先，我们将创建一个简单的 Mongo 数据库和样本集合。然后，我们将在其中放入虚拟数据，以便在后面的示例中使用。之后，我们将展示如何在本地 Mongo 查询以及 Java 中检查该字段是否存在。

## 2.示例配置

在我们开始检查字段是否存在之前，我们需要一个现有的数据库、集合和虚拟数据供以后使用。为此，我们将使用 Mongo shell。

首先，让我们将 Mongo shell 上下文切换到一个`existence`数据库:

```
use existence
```

值得指出的是，MongoDB 只在您第一次在数据库中存储数据时创建数据库。我们将在`users`集合中插入一个用户:

```
db.users.insert({name: "Ben", surname: "Big" })
```

现在我们有了所有需要检查的东西，不管这个场是否存在。

## 3.检查 Mongo Shell 中的字段是否存在

有时我们需要通过使用基本查询来检查特定字段的存在，例如在 Mongo Shell 或任何其他数据库控制台中。幸运的是，Mongo 为此提供了一个特殊的查询操作符`$exists`:

```
db.users.find({ 'name' : { '$exists' : true }})
```

我们使用一个标准的`find` Mongo 方法，在该方法中，我们指定要查找的字段，并使用`$exists`查询操作符。如果`name`字段存在于`users`集合中，将返回包含该字段的所有行:

```
[
  {
    "_id": {"$oid": "6115ad91c4999031f8e6f582"},
    "name": "Ben",
    "surname": "Big"
  }
]
```

如果字段丢失，我们将得到一个空结果。

## 4.检查 Java 中的字段是否存在

在我们讨论 Java 中检查字段存在的可能方法之前，让我们向我们的项目添加必要的 [Mongo 依赖项。以下是 Maven 的依赖性:](https://web.archive.org/web/20220926194125/https://search.maven.org/search?q=a:mongo-java-driver%20AND%20g:org.mongodb)

```
<dependency>
    <groupId>org.mongodb</groupId>
    <artifactId>mongo-java-driver</artifactId>
    <version>3.12.10</version>
</dependency>
```

这是格雷尔的版本:

```
implementation group: 'org.mongodb', name: 'mongo-java-driver', version: '3.12.10'
```

最后，让我们连接到`existence`数据库和`users`集合:

```
MongoClient mongoClient = new MongoClient();
MongoDatabase db = mongoClient.getDatabase("existence");
MongoCollection<Document> collection = db.getCollection("users");
```

### 4.1.使用过滤器

`com.mongodb.client.model.Filters `是来自 Mongo 依赖项的一个 util 类，包含了许多有用的方法。在我们的例子中，我们将使用`exists()`方法:

```
Document nameDoc = collection.find(Filters.exists("name")).first();
assertNotNull(nameDoc);
assertFalse(nameDoc.isEmpty());
```

首先，我们试图从`users`集合中找到元素，并获得找到的第一个元素。如果指定的字段存在，我们将得到一个`nameDoc`文档作为响应。不是 null 也不是空的。

现在，让我们看看当我们试图找到一个不存在的字段时会发生什么:

```
Document nameDoc = collection.find(Filters.exists("non_existing")).first();
assertNull(nameDoc);
```

如果没有找到元素，我们将得到一个空文档作为响应。

### 4.2.使用文档查询

`com.mongodb.client.model.Filters`类不是检查字段存在的唯一方法。我们可以使用`com.mongodb.BasicDBObject:`的一个实例

```
Document query = new Document("name", new BasicDBObject("$exists", true));
Document doc = collection.find(query).first();
assertNotNull(doc);
assertFalse(doc.isEmpty());
```

行为与上一个示例相同。如果找到了元素，我们收到一个 not `null` `Document`，为空。

当我们试图查找一个不存在的字段时，代码的行为也是一样的:

```
Document query = new Document("non_existing", new BasicDBObject("$exists", true));
Document doc = collection.find(query).first();
assertNull(doc);
```

如果没有找到元素，我们得到一个`null` `Document`作为响应。

## 5.结论

在本文中，我们讨论了如何在 MongoDB 中检查字段的存在性。首先，我们展示了如何创建 Mongo 数据库、集合，以及如何插入虚拟数据。然后，我们解释了如何使用一个基本查询来检查一个字段在 Mongo shell 中是否存在。最后，我们解释了如何使用`com.mongodb.client.model.Filters`和`Document`查询方法来检查字段的存在。

和往常一样，GitHub 上的[提供了这篇文章的完整源代码。](https://web.archive.org/web/20220926194125/https://github.com/eugenp/tutorials/tree/master/persistence-modules/java-mongodb-2)