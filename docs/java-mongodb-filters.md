# MongoDB 中的过滤器指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-mongodb-filters>

## 1.概观

在本教程中，我们将学习如何使用 [`Filters`](https://web.archive.org/web/20221107213538/https://www.mongodb.com/docs/drivers/java/sync/current/fundamentals/builders/filters/) 构建器为 [MongoDB](/web/20221107213538/https://www.baeldung.com/java-mongodb) 中的查询指定过滤器。

`Filters`类是一种帮助我们构建查询过滤器的构建器。 **`Filters`是 MongoDB 用来根据特定条件**限制结果的操作。

## 2.建筑商的类型

Java MongoDB 驱动程序提供了各种类型的[构建器](https://web.archive.org/web/20221107213538/https://www.mongodb.com/docs/drivers/java/sync/current/fundamentals/builders/)，帮助我们构建 [BSON](/web/20221107213538/https://www.baeldung.com/mongodb-bson) 文档。**构建者提供了一个方便的 API 来简化执行各种 CRUD 和聚合操作的过程**。

让我们回顾一下现有的不同类型的构建器:

*   `Filters`用于构建查询过滤器
*   `Projections`对于建筑场投影，指定包括和排除哪些场
*   `Sorts`用于建立排序标准
*   `Updates`进行建筑更新操作
*   `Aggregates`用于建设聚合管道
*   `Indexes`用于建立索引键

现在让我们深入探讨一下在 MongoDB 中使用`Filters`的不同方式。

## 3.数据库初始化

首先，为了演示各种过滤操作，让我们建立一个数据库`baeldung`和一个样本集合`user`:

```
use baeldung;
db.createCollection("user");
```

此外，让我们将一些文档填充到`user`集合中:

```
db.user.insertMany([
{
    "userId":"123",
    "userName":"Jack",
    "age":23,
    "role":"Admin"
},
{
    "userId":"456",
    "userName":"Lisa",
    "age":27,
    "role":"Admin",
    "type":"Web"
},
{
    "userId":"789",
    "userName":"Tim",
    "age":31,
    "role":"Analyst"
}]);
```

成功插入后，上述查询将返回确认结果的响应:

```
{
    "acknowledged" : true,
    "insertedIds" : [
        ObjectId("6357c4736c9084bcac72eced"),
        ObjectId("6357c4736c9084bcac72ecee"),
        ObjectId("6357c4736c9084bcac72ecef")
    ]
}
```

我们将使用这个集合作为所有过滤器操作示例的样本。

## 4.使用`Filters`类

如前所述，`Filters`是 MongoDB 用来将结果限制在我们希望看到的范围内的操作。`Filters`类为不同类型的 MongoDB 操作提供了各种`static`工厂方法。每个方法返回一个 BSON 类型，然后可以将其传递给任何需要查询过滤器的方法。

**当前 MongoDB Java 驱动 API 中的`Filters`类替换了遗留 API 中的 [`QueryBuilder`](https://web.archive.org/web/20221107213538/https://mongodb.github.io/mongo-java-driver/4.7/apidocs/mongodb-driver-legacy/com/mongodb/QueryBuilder.html) 。**

根据操作类型进行分类:条件、逻辑、数组、元素、求值、按位和地理空间。

接下来，我们来看看一些最常用的`Filters` 方法。

### 4.1.`eq()`方法

`Filters.eq()`方法**创建一个过滤器，匹配指定字段的值等于指定值**的所有文档。

首先，让我们看一下 MongoDB Shell 查询来过滤`user`集合文档，其中`userName`等于`“Jack”`:

```
db.getCollection('user').find({"userName":"Jack"})
```

上面的查询从`user`集合中返回一个文档:

```
{
    "_id" : ObjectId("6357c4736c9084bcac72eced"),
    "userId" : "123",
    "userName" : "Jack",
    "age" : 23.0,
    "role" : "Admin"
}
```

现在，让我们看看相应的 MongoDB Java 驱动程序代码:

```
Bson filter = Filters.eq("userName", "Jack");
FindIterable<Document> documents = collection.find(filter);

MongoCursor<Document> cursor = documents.iterator();
while (cursor.hasNext()) {
    System.out.println(cursor.next());
}
```

我们可以注意到，`Filters.eq()`方法返回一个 BSON 类型，然后我们将它作为过滤器传递给 [`find()`](/web/20221107213538/https://www.baeldung.com/mongodb-find) 方法。

另外， **`Filters.ne()`正好与`Filters.eq()`方法**相反——它匹配所有命名字段的值不等于指定值的文档:

```
Bson filter = Filters.ne("userName", "Jack");
FindIterable<Document> documents = collection.find(filter);

MongoCursor<Document> cursor = documents.iterator();
while (cursor.hasNext()) {
    System.out.println(cursor.next());
}
```

### 4.2.`gt()`方法

`Filters.gt()`方法**创建一个过滤器，匹配指定字段的值大于指定值**的所有文档。

让我们看一个例子:

```
Bson filter = Filters.gt("age", 25);
FindIterable<Document> documents = collection.find(filter);

MongoCursor<Document> cursor = documents.iterator();
while (cursor.hasNext()) {
    System.out.println(cursor.next());
}
```

上面的代码片段获取所有`age`大于`25.`的`user`集合文档，就像`Filters.gt()`方法一样，**还有一个`Filters.lt()`方法匹配所有命名字段的值小于指定值**的文档:

```
Bson filter = Filters.lt("age", 25);
FindIterable<Document> documents = collection.find(filter);

MongoCursor<Document> cursor = documents.iterator();
while (cursor.hasNext()) {
    System.out.println(cursor.next());
}
```

还有，**有方法`Filters.gte()`和`Filters.lte()`分别匹配大于等于和小于等于指定值的值**。

### 4.3.`in()`方法

`Filters.in()`方法**创建一个匹配所有文档的过滤器，其中指定字段的值等于指定值列表**中的任何值。

让我们来看看它的实际应用:

```
Bson filter = Filters.in("userName", "Jack", "Lisa");
FindIterable<Document> documents = collection.find(filter);

MongoCursor<Document> cursor = documents.iterator();
while (cursor.hasNext()) {
    System.out.println(cursor.next());
}
```

这个代码片段获取所有的`user`集合文档，其中`userName`等于`“Jack”`或`“Lisa”.`

就像`Filters.in()`方法一样，**有一个`Filters.nin()`方法匹配所有文档，其中指定字段的值不等于指定值列表**中的任何值:

```
Bson filter = Filters.nin("userName", "Jack", "Lisa");
FindIterable<Document> documents = collection.find(filter);

MongoCursor<Document> cursor = documents.iterator();
while (cursor.hasNext()) {
    System.out.println(cursor.next());
}
```

### 4.4.`and()`方法

`Filters.and()`方法**创建一个过滤器，为提供的过滤器列表**执行逻辑与操作。

让我们找出所有的`age`大于`25`、`role`等于`“Admin”`的`user`收款单据:

```
Bson filter = Filters.and(Filters.gt("age", 25), Filters.eq("role", "Admin"));
FindIterable<Document> documents = collection.find(filter);

MongoCursor<Document> cursor = documents.iterator();
while (cursor.hasNext()) {
    System.out.println(cursor.next());
}
```

### 4.5.`or()`方法

正如我们所料，`Filters.or()`方法**创建了一个过滤器，它对提供的过滤器列表**执行逻辑 OR 操作。

让我们编写一个代码片段，返回所有的`user`集合文档，其中`age`大于`30`或者`role`等于`“Admin”`:

```
Bson filter = Filters.or(Filters.gt("age", 30), Filters.eq("role", "Admin"));
FindIterable<Document> documents = collection.find(filter);

MongoCursor<Document> cursor = documents.iterator();
while (cursor.hasNext()) {
    System.out.println(cursor.next());
}
```

### 4.6.`exists()`方法

此外，`Filters.exists()`方法**创建一个过滤器，匹配包含给定字段**的所有文档:

```
Bson filter = Filters.exists("type");
FindIterable<Document> documents = collection.find(filter);

MongoCursor<Document> cursor = documents.iterator();
while (cursor.hasNext()) {
    System.out.println(cursor.next());
}
```

上面的代码返回所有有一个`type`字段的`user`集合文档。

### 4.7.`regex()`方法

最后，`Filters.regex()`方法**创建一个匹配文档的过滤器，其中指定字段的值匹配给定的正则表达式模式**:

```
Bson filter = Filters.regex("userName", "a");
FindIterable<Document> documents = collection.find(filter);

MongoCursor<Document> cursor = documents.iterator();
while (cursor.hasNext()) {
    System.out.println(cursor.next());
}
```

这里，我们获取了所有与正则表达式`“a”`匹配的`user`集合文档。

到目前为止，我们已经讨论了一些最常用的过滤操作符。我们可以使用查询过滤器操作符的任意组合作为`find()`方法的过滤器。

此外，过滤器可以用在各种其他地方，例如聚合的匹配阶段、`deleteOne()`方法和`updateOne()`方法等等。

## 5.结论

在本文中，我们首先讨论了如何使用`Filters`构建器在 MongoDB 集合上执行过滤操作。然后，我们看到了如何使用 MongoDB Java 驱动程序 API 实现一些最常用的过滤器操作。

和往常一样，所有例子的完整代码都可以在 GitHub 上找到[。](https://web.archive.org/web/20221107213538/https://github.com/eugenp/tutorials/tree/master/persistence-modules/java-mongodb-3)