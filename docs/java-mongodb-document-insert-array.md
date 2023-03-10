# 在 MongoDB 中的对象内插入数组

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-mongodb-document-insert-array>

## 1.概观

MongoDB 是最流行的开源分布式面向文档的 NoSQL 数据库。MongoDB 中的文档是一种数据结构，其中类似 JSON 的对象具有字段和值对。

为了将文档插入到 MongoDB 集合中，我们可以使用不同的方法，例如 [`insert()`](https://web.archive.org/web/20221207153629/https://www.mongodb.com/docs/manual/reference/method/db.collection.insert/) 、 [`insertOne()`](https://web.archive.org/web/20221207153629/https://www.mongodb.com/docs/manual/reference/method/db.collection.insertOne/) 和 [`insertMany()`](https://web.archive.org/web/20221207153629/https://www.mongodb.com/docs/manual/reference/method/db.collection.insertMany/) 。

在本教程中，我们将讨论如何在一个 [MongoDB](/web/20221207153629/https://www.baeldung.com/java-mongodb) 文档中插入一个数组。首先，我们将看看如何使用 MongoDB Shell 查询将数组插入到文档中。然后我们将使用 [MongoDB Java 驱动程序](/web/20221207153629/https://www.baeldung.com/java-mongodb)代码。

## 2.数据库初始化

在我们继续插入查询之前，让我们首先创建一个数据库。让我们称之为`baeldung. `我们还将创建一个名为`student:`的样本集合

```java
use baeldung;
db.createCollection(student);
```

使用这个命令，我们的示例数据库`baeldung`和`student`集合被成功设置。我们将在所有示例中使用这些来演示。

## 3.使用 MongoDB Shell

**要使用 MongoDB Shell 将数组插入到集合中，我们可以简单地将数组作为 JSON 数组类型**传递给 Shell:

```java
db.student.insert({
    "studentId" : "STU1",
    "name" : "Avin",
    "Age" : 12,
    "courses" : ["Science", "Math"]
});
```

上面的查询在`student`集合中插入一个带有数组的文档。我们可以通过使用 [`find`](/web/20221207153629/https://www.baeldung.com/mongodb-find) 操作符查询`student`集合的文档来验证结果:

```java
db.student.find();
```

以上查询返回插入的`student`收款单:

```java
{
    "_id" : ObjectId("631da4197581ba6bc1d2524d"),
    "studentId" : "STU1",
    "name" : "Avin",
    "Age" : 12,
    "courses" : [ "Science", "Math" ]
}
```

## 4.使用 Java 驱动程序代码的插入操作

MongoDB Java 驱动程序提供了各种方便的方法来帮助我们将文档插入到集合中:

*   `insert()`–将单个文档或多个文档插入集合
*   `insertOne()`–将单个文档插入集合
*   `insertMany()`–将多个文档插入一个集合

以上任何方法都可以用来对 MongoDB 集合执行`insert`操作。

接下来，让我们深入研究使用 Java MongoDB 驱动程序实现数组插入操作。MongoDB Java 驱动程序支持`DBObject`和`BSON`文档。

## 5.使用`DBObject`

这里， **`DBObject`是 MongoDB 遗留驱动程序的一部分，但是在 MongoDB 的新版本中不推荐使用。**

让我们将一个带有数组的`DBObject`文档插入到`student`集合中:

```java
BasicDBList coursesList = new BasicDBList();
coursesList.add("Chemistry");
coursesList.add("Science");

DBObject student = new BasicDBObject().append("studentId", "STU1")
  .append("name", "Jim")
  .append("age", 13)
  .append("courses", coursesList);

dbCollection.insert(student);
```

上面的查询将带有数组的单个`DBObject`文档插入到`student`集合中。

## 6.使用`BSON`文件

[`BSON`](/web/20221207153629/https://www.baeldung.com/mongodb-bson) 文档是用 Java 访问 MongoDB 文档的新方法，是用更新的客户端栈构建的。幸运的是，它也更容易使用。

**Java 驱动程序提供了一个`org.bson.Document `类来将一个带有数组**的`Bson`文档对象插入到`student`集合中。

### 6.1.用数组插入单个文档

首先，让我们使用`insertOne()`方法将带有数组的单个文档插入集合:

```java
List coursesList = new ArrayList<>();
coursesList.add("Science");
coursesList.add("Geography");

Document student = new Document().append("studentId", "STU2")
  .append("name", "Sam")
  .append("age", 13)
  .append("courses", coursesList);

collection.insertOne(student);
```

上面的查询将带有数组的单个文档插入到`student`集合中。需要注意的是，`Document`类的`append(String, Object)`方法接受一个`Object`作为值。我们可以传递任何类型的`List`作为值，将其作为数组插入到文档中。

### 6.2.用数组插入多个文档

让我们使用`insertMany()`方法将带有数组的多个文档插入到集合中:

```java
List coursesList1 = new ArrayList<>();
coursesList1.add("Chemistry");
coursesList1.add("Geography");

Document student1 = new Document().append("studentId", "STU3")
  .append("name", "Sarah")
  .append("age", 12)
  .append("courses", coursesList1);

List coursesList2 = new ArrayList<>();
coursesList2.add("Math");
coursesList2.add("History");

Document student2 = new Document().append("studentId", "STU4")
  .append("name", "Tom")
  .append("age", 13)
  .append("courses", coursesList2);

List<Document> students = new ArrayList<>();
students.add(student1);
students.add(student2);

collection.insertMany(students);
```

上面的查询将多个带有数组的文档插入到`student`集合中。

### 6.3.插入一个`Object`数组

最后，让我们将一个`Object`数组类型的文档插入 MongoDB 集合:

```java
Document course1 = new Document().append("name", "C1")
  .append("points", 5);

Document course2 = new Document().append("name", "C2")
  .append("points", 7);

List<Document> coursesList = new ArrayList<>();
coursesList.add(course1);
coursesList.add(course2);

Document student = new Document().append("studentId", "STU5")
  .append("name", "Sam")
  .append("age", 13)
  .append("courses", coursesList);

collection.insertOne(student);
```

上面的查询将多个带有`Object`数组的文档插入到`student`集合中。这里，我们将一个包含文档列表的文档作为数组插入到集合中。类似地，我们可以构造任何复杂的数组`Object`，并将其插入到 MongoDB 集合中。

## 7.结论

在本文中，我们看到了将带有数组`Object`的文档插入 MongoDB 集合的各种方法。我们使用 MongoDB Shell 查询和相应的 Java 驱动程序代码实现讨论了这些用例。

对于 Java 驱动代码，我们首先看一下使用不推荐的`DBObject`类的实现。然后，我们学习使用新的`BSON`文档类来实现相同的功能。

所有这些例子和代码片段的实现都可以在 GitHub 的[上找到。](https://web.archive.org/web/20221207153629/https://github.com/eugenp/tutorials/tree/master/persistence-modules/java-mongodb-3)