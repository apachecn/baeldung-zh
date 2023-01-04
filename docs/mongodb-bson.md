# mongodb bson 指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/mongodb-bson>

## 1.介绍

在本教程中，我们将关注 [BSON](https://web.archive.org/web/20221225164858/http://bsonspec.org/) 以及我们如何使用它与 [MongoDB](https://web.archive.org/web/20221225164858/https://www.mongodb.com/) 进行交互。

现在，对 MongoDB 及其所有功能的深入描述超出了本文的范围。然而，理解几个关键概念是有用的。

MongoDB 是一个分布式的 NoSQL 文档存储引擎。文档存储为 BSON 数据，并分组到集合中。集合中的文档类似于关系数据库表中的行。

要更深入地了解，请看一下介绍 MongoDB 的文章。

## 2.什么是`BSON`？

**BSON 代表`Binary JSON`** 。这是一个类似 JSON 数据的二进制序列化协议。

JSON 是现代 web 服务中流行的一种数据交换格式。它提供了一种灵活的方式来表示复杂的数据结构。

与使用常规 JSON 相比，BSON 提供了几个优势:

*   紧凑:在大多数情况下，存储一个 **BSON 结构比它的 JSON 等价结构**需要更少的空间
*   数据类型: **BSON 提供了常规 JSON 中没有的附加数据类型**，比如`Date`和`BinData`

使用 BSON 的一个主要好处是很容易穿越。BSON 文档包含额外的元数据，允许轻松操作文档的字段，而不必阅读整个文档本身。

## 3.MongoDB 驱动程序

现在我们对 BSON 和 MongoDB 有了基本的了解，让我们看看如何一起使用它们。我们将关注 CRUD 首字母缩写( **C** reate， **R** ead， **U** pdate， **D** elete)的主要动作。

MongoDB 为大多数现代编程语言提供了[软件驱动](https://web.archive.org/web/20221225164858/https://docs.mongodb.com/ecosystem/drivers/)。**驱动程序构建在 BSON 库**之上，这意味着我们在构建查询时将直接使用 BSON API。有关更多信息，请参见我们的 MongoDB 查询语言的[指南。](/web/20221225164858/https://www.baeldung.com/queries-in-spring-data-mongodb)

在这一节中，我们将了解如何使用驱动程序连接到集群，以及如何使用 BSON API 来执行不同类型的查询。注意，MongoDB 驱动程序提供了一个 [`Filters`类](https://web.archive.org/web/20221225164858/https://mongodb.github.io/mongo-java-driver/3.10/javadoc/?com/mongodb/client/model/Filters.html)，可以帮助我们编写更紧凑的代码。然而，对于本教程，我们将只关注核心 BSON API 的使用。

作为直接使用 MongoDB 驱动程序和 BSON 的替代方法，看看我们的 [Spring Data MongoDB 指南](/web/20221225164858/https://www.baeldung.com/spring-data-mongodb-guide)。

### 3.1.连接

首先，我们将 [MongoDB 驱动程序](https://web.archive.org/web/20221225164858/https://search.maven.org/search?q=g:org.mongodb%20AND%20a:mongodb-driver-sync)作为一个依赖项添加到我们的应用程序中:

```java
<dependency>
    <groupId>org.mongodb</groupId>
    <artifactId>mongodb-driver-sync</artifactId>
    <version>3.10.1</version>
</dependency>
```

然后我们创建一个到 MongoDB 数据库和集合的连接:

```java
MongoClient mongoClient = MongoClients.create();
MongoDatabase database = mongoClient.getDatabase("myDB");
MongoCollection<Document> collection = database.getCollection("employees");
```

其余部分将着眼于使用`collection`引用创建查询。

### 3.2.插入

假设我们有下面的 JSON，我们想把它作为一个新文档插入到一个`employees`集合中:

```java
{
  "first_name" : "Joe",
  "last_name" : "Smith",
  "title" : "Java Developer",
  "years_of_service" : 3,
  "skills" : ["java","spring","mongodb"],
  "manager" : {
     "first_name" : "Sally",
     "last_name" : "Johanson"
  }
}
```

这个 JSON 示例展示了我们在 MongoDB 文档中会遇到的最常见的数据类型:文本、数字、数组和嵌入式文档。

为了使用 BSON 来插入它，我们将使用 MongoDB 的`Document` API:

```java
Document employee = new Document()
    .append("first_name", "Joe")
    .append("last_name", "Smith")
    .append("title", "Java Developer")
    .append("years_of_service", 3)
    .append("skills", Arrays.asList("java", "spring", "mongodb"))
    .append("manager", new Document()
                           .append("first_name", "Sally")
                           .append("last_name", "Johanson"));
collection.insertOne(employee); 
```

类是 BSON 使用的主要 API。它扩展了 Java `Map`接口，并包含几个重载方法。这使得处理本机类型以及通用对象(如对象 id、日期和列表)变得很容易。

### 3.3.发现

为了在 MongoDB 中找到一个文档，我们提供了一个搜索文档，指定要查询哪些字段。例如，要查找姓氏为“Smith”的所有文档，我们可以使用下面的 JSON 文档:

```java
{  
  "last_name": "Smith"
}
```

如果写在 BSON，这将是:

```java
Document query = new Document("last_name", "Smith");
List results = new ArrayList<>();
collection.find(query).into(results);
```

“Find”查询可以接受多个字段，默认行为是使用逻辑运算符`and`来组合它们。**这意味着只有匹配所有字段的文档才会被返回**。

为了解决这个问题，MongoDB 提供了`or`查询操作符:

```java
{
  "$or": [
    { "first_name": "Joe" },
    { "last_name":"Smith" }
  ]
}
```

这将查找名字为“Joe”或姓氏为“Smith”的所有文档。要将它写成 BSON，我们将使用嵌套的`Document`，就像上面的插入查询一样:

```java
Document query = 
  new Document("$or", Arrays.asList(
      new Document("last_name", "Smith"),
      new Document("first_name", "Joe")));
List results = new ArrayList<>();
collection.find(query).into(results);
```

### 3.4.更新

**更新查询在 MongoDB 中略有不同，因为它们需要两个文档**:

1.  查找一个或多个文档的筛选条件
2.  指定要修改哪些字段的更新文档

例如，假设我们想要为每个已经拥有“弹簧”技能的员工添加一个“安全”技能。第一个文档将找到所有拥有“spring”技能的员工，第二个文档将向他们的技能数组中添加一个新的“security”条目。

在 JSON 中，这两个查询类似于:

```java
{
  "skills": { 
    $elemMatch:  { 
      "$eq": "spring"
    }
  }
}

{
  "$push": { 
    "skills": "security"
  }
}
```

在 BSON，他们会是:

```java
Document query = new Document(
  "skills",
  new Document(
    "$elemMatch",
    new Document("$eq", "spring")));
Document update = new Document(
  "$push",
  new Document("skills", "security"));
collection.updateMany(query, update); 
```

### 3.5.删除

MongoDB 中的删除查询使用与查找查询相同的语法。我们只是提供一个文档，指定一个或多个匹配标准。

例如，假设我们在员工数据库中发现了一个错误，并意外地创建了服务年限为负值的员工 a。为了找到它们，我们将使用下面的 JSON:

```java
{
  "years_of_service" : { 
    "$lt" : 0
  }
}
```

相应的 BSON 文件将是:

```java
Document query = new Document(
  "years_of_service", 
  new Document("$lt", 0));
collection.deleteMany(query);
```

## 4.结论

在本教程中，我们已经看到了使用 BSON 库构建 MongoDB 查询的基本介绍。仅使用 BSON API，我们为 MongoDB 集合实现了基本的 CRUD 操作。

我们没有涉及的是更高级的主题，如投影、聚合、地理空间查询、批量操作等。仅仅使用 BSON 图书馆，所有这些都是可能的。我们在这里看到的例子构成了我们用来实现这些更高级操作的构建块。

和往常一样，你可以在我们的 [GitHub repo](https://web.archive.org/web/20221225164858/https://github.com/eugenp/tutorials/tree/master/persistence-modules/java-mongodb) 中找到上面的代码示例。