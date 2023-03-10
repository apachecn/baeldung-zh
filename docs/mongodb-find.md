# 在 MongoDB 中查找指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/mongodb-find>

## 1.概观

在本教程中，我们将研究如何在 [MongoDB](https://web.archive.org/web/20220810165323/https://www.mongodb.com/) 中执行搜索操作来检索文档。MongoDB 提供了一个`find`操作符来查询集合中的文档。**`find`操作符的主要目的是根据查询条件从集合中选择文档，并将光标返回到所选的文档。**

在本教程中，我们将首先查看 MongoDB Shell 查询中的`find`操作符，然后使用 Java 驱动程序代码。

## 2.数据库初始化

在我们继续执行`find`操作之前，我们首先需要建立一个数据库`baeldung`和一个样本集合`employee`:

```java
db.employee.insertMany([
{
    "employeeId":"EMP1",
    "name":"Sam", 
    "age":23,
    "type":"Full Time",
    "department":"Engineering"
},
{ 
    "employeeId":"EMP2",
    "name":"Tony",
    "age":31,
    "type":"Full Time",
    "department":"Admin"
},
{
    "employeeId":"EMP3",
    "name":"Lisa",
    "age":42,
    "type":"Part Time",
    "department":"Engineering"
}]);
```

成功插入后，上面的查询将返回类似如下所示的 JSON 结果:

```java
{
    "acknowledged" : true,
    "insertedIds" : [
        ObjectId("62a88223ff0a77909323a7fa"),
        ObjectId("62a88223ff0a77909323a7fb"),
        ObjectId("62a88223ff0a77909323a7fc")
    ]
}
```

此时，我们已经将一些文档插入到我们的集合中，以执行各种类型的`find`操作。

## 3.使用 MongoDB Shell

为了从 MongoDB 集合中查询文档，我们使用了`db.collection.find(query, projection)`方法。该方法接受两个可选参数——`query`和`projection`——作为 MongoDB [BSON](/web/20220810165323/https://www.baeldung.com/mongodb-bson) 文档。

`query`参数接受带有查询操作符的选择过滤器。为了从 MongoDB 集合中检索所有文档，我们可以省略这个参数或者传递一个空白文档。

接下来，`projection`参数用于指定从匹配文档返回的字段。要返回匹配文档中的所有字段，我们可以省略这个参数。

此外，让我们从一个基本的`find`查询开始，该查询将返回所有的集合文档:

```java
db.employee.find({});
```

上面的查询将返回来自`employee`集合的所有文档:

```java
{ "_id" : ObjectId("62a88223ff0a77909323a7fa"), "employeeId" : "1", "name" : "Sam", "age" : 23, "type" : "Full Time", "department" : "Engineering" }
{ "_id" : ObjectId("62a88223ff0a77909323a7fb"), "employeeId" : "2", "name" : "Tony", "age" : 31, "type" : "Full Time", "department" : "Admin" }
{ "_id" : ObjectId("62a88223ff0a77909323a7fc"), "employeeId" : "3", "name" : "Ray", "age" : 42, "type" : "Part Time", "department" : "Engineering" }
```

接下来，让我们编写一个查询来返回属于`“Engineering”` `department`的所有雇员:

```java
db.employee.find(
{
    "department":"Engineering"
});
```

以上查询返回所有`department`等于`Engineering”`的`employee`收款单据:

```java
{ "_id" : ObjectId("62a88223ff0a77909323a7fa"), "employeeId" : "1", "name" : "Sam", "age" : 23, "type" : "Full Time", "department" : "Engineering" }
{ "_id" : ObjectId("62a88223ff0a77909323a7fc"), "employeeId" : "3", "name" : "Ray", "age" : 42, "type" : "Part Time", "department" : "Engineering" }
```

最后，让我们编写一个查询来获取属于`“Engineering” department`的所有雇员的`name`和`age`:

```java
db.employee.find(
{
    "department":"Engineering"
},
{
    "name":1,
    "age":1
});
```

上面的查询只返回符合查询条件的文档的`name`和`age`字段:

```java
{ "_id" : ObjectId("62a88223ff0a77909323a7fa"), "name" : "Sam", "age" : 23 }
{ "_id" : ObjectId("62a88223ff0a77909323a7fc"), "name" : "Ray", "age" : 42 }
```

注意，默认情况下，除非明确排除，否则在所有文档中都返回`_id`字段。

另外，需要注意的是， **`find`操作符将光标返回到匹配查询过滤器**的文档。MongoDB Shell 自动迭代光标以显示多达 20 个文档。

此外，MongoDB Shell 提供了一个 **`findOne()`方法，该方法只返回一个满足上述查询标准**的文档。如果多个文档匹配，将根据文档在磁盘上的自然顺序返回第一个文档:

```java
db.employee.findOne();
```

与`find()`不同，上面的查询将只返回一个文档，而不是一个光标:

```java
{
    "_id" : ObjectId("62a99e22a849e1472c440bbf"),
    "employeeId" : "EMP1",
    "name" : "Sam",
    "age" : 23,
    "type" : "Full Time",
    "department" : "Engineering"
}
```

## 4.使用 Java 驱动程序

到目前为止，我们已经看到了如何使用 MongoDB Shell 来执行`find`操作。接下来，让我们使用 MongoDB Java 驱动程序实现同样的功能。在我们开始之前，让我们首先创建一个到`employee`集合的`MongoClient`连接:

```java
MongoClient mongoClient = new MongoClient("localhost", 27017);
MongoDatabase database = mongoClient.getDatabase("baeldung");
MongoCollection<Document> collection = database.getCollection("employee");
```

这里，我们创建了一个到运行在默认端口 27017 上的 MongoDB 服务器的连接。接下来，我们从连接创建的`MongoDatabase`实例中获得了一个`MongoCollection`实例。

首先，为了执行一个`find`操作，我们在一个`MongoCollection`的实例上调用`find()`方法。让我们检查代码，从集合中检索所有文档:

```java
FindIterable<Document> documents = collection.find();
MongoCursor<Document> cursor = documents.iterator();
while (cursor.hasNext()) {
    System.out.println(cursor.next());
}
```

注意，`find()`方法返回了`FindIterable<Document>`的一个实例。然后，我们通过调用`FindIterable.`的`iterator()`方法来获得`MongoCursor`的实例。最后，我们遍历光标来检索每个文档。

接下来，让我们添加查询操作符来过滤从`find`操作返回的文档:

```java
Bson filter = Filters.eq("department", "Engineering");
FindIterable<Document> documents = collection.find(filter);

MongoCursor<Document> cursor = documents.iterator();
while (cursor.hasNext()) {
    System.out.println(cursor.next());
}
```

这里，我们将一个`Bson`过滤器作为参数传递给`find()`方法。我们可以使用[查询操作符](https://web.archive.org/web/20220810165323/https://www.mongodb.com/docs/manual/reference/operator/query/)的任意组合作为`find()`方法的过滤器。上面的代码片段将返回所有`department`等于“工程”的文档。

此外，让我们编写一个代码片段，只返回文档中符合选择标准的`name`和`age`字段:

```java
Bson filter = Filters.eq("department", "Engineering");
Bson projection = Projections.fields(Projections.include("name", "age"));
FindIterable<Document> documents = collection.find(filter)
  .projection(projection);

MongoCursor<Document> cursor = documents.iterator();
while (cursor.hasNext()) {
    System.out.println(cursor.next());
}
```

这里，我们在`FindIterable`实例上调用`projection()`方法。我们将一个`Bson`过滤器作为参数传递给`projection()`方法。我们可以使用`projection`操作从最终结果中包含或排除任何字段。作为直接使用 MongoDB 驱动程序和`Bson`的替代方法，请看看我们的[Spring Data MongoDB](/web/20220810165323/https://www.baeldung.com/mongodb-return-specific-fields)中的投影指南。

最后，我们可以在`FindIterable`实例上使用`first()`方法检索结果的第一个文档。这将返回一个单独的文档，而不是一个`MongoCursor`实例:

```java
FindIterable<Document> documents = collection.find();
Document document = documents.first();
```

## 5.结论

在本文中，我们学习了使用各种方法在 MongoDB 中执行`find`操作。我们使用查询操作符执行`find`来检索符合选择标准的特定文档。此外，我们还学习了执行一个`projection`来确定匹配文档中返回哪些字段。

首先，我们研究了 MongoDB Shell 查询中`find`操作的用例，然后我们讨论了相应的 Java 驱动程序代码。

所有这些例子和代码片段的实现都可以在 GitHub 的[上找到。](https://web.archive.org/web/20220810165323/https://github.com/eugenp/tutorials/tree/master/persistence-modules/java-mongodb-3)