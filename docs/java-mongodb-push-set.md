# 在同一个 MongoDB 更新中的推送和设置操作

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-mongodb-push-set>

## 1.概观

`$push`是 MongoDB 中的一个更新操作符，它将数组中的值相加。相反，`$set`操作符用于更新文档中现有字段的值。

在这个简短的教程中，我们将介绍如何在一个更新查询中一起执行`$push`和`$set`操作。

## 2.数据库初始化

在我们继续执行[多重更新操作](/web/20221008212435/https://www.baeldung.com/mongodb-update-multiple-fields)之前，我们首先需要设置一个数据库`baeldung`和样本收集`marks`:

```java
use baeldung;
db.createCollection(marks);
```

让我们使用 MongoDB 的`insertMany`方法将一些文档插入到集合`marks`中:

```java
db.marks.insertMany([
    {
        "studentId": 1023,
        "studentName":"James Broad",
        "joiningYear":"2018",
        "totalMarks":100,
        "subjectDetails":[
            {
                "subjectId":123,
                "subjectName":"Operating Systems Concepts",
                "marks":40
            },
            {
                "subjectId":124,
                "subjectName":"Numerical Analysis",
                "marks":60
            }
        ]
    },
    {
        "studentId": 1024,
        "studentName":"Chris Overton",
        "joiningYear":"2018",
        "totalMarks":110,
        "subjectDetails":[
            {
                "subjectId":123,
                "subjectName":"Operating Systems Concepts",
                "marks":50
            },
            {
                "subjectId":124,
                "subjectName":"Numerical Analysis",
                "marks":60
            }
        ]
    }
]);
```

成功插入后，上述查询将返回以下响应:

```java
{
    "acknowledged" : true,
    "insertedIds" : [
        ObjectId("622300cc85e943405d04b567"),
        ObjectId("622300cc85e943405d04b568")
    ]
}
```

到目前为止，我们已经成功地将一些样本文档插入到集合`marks`中。

## 3.理解问题

为了理解问题，让我们首先理解我们刚刚插入的文档。它包括学生的详细资料和他们在不同科目上取得的成绩。`totalMarks `是各科成绩的总和。

让我们考虑这样一种情况，我们希望在`subjectDetails`数组中添加一个新主题。为了使数据一致，我们还需要更新`totalMarks`字段。

在 MongoDB 中，首先，我们将使用`$push`操作符将新主题添加到数组中。然后我们将使用`$set`操作符将`totalMarks`字段设置为一个特定的值。

这两个操作可以分别使用`$push`和`$set`操作符单独执行。但是我们可以编写 MongoDB 查询来一起执行这两种操作。

## 4.使用 MongoDB Shell 查询

在 MongoDB 中，我们可以使用不同的更新操作符来更新文档的多个字段。这里，我们将在一个`updateOne`查询中同时使用`$push`和`$set`操作符。

让我们看看同时包含`$push`和`$set`操作符的例子:

```java
db.marks.updateOne(
    {
        "studentId": 1023
    },
    {
        $set: {
            totalMarks: 170
        },
        $push: {
            "subjectDetails":{
                "subjectId": 126,
                "subjectName": "Java Programming",
                "marks": 70
            }
        }
    }
);
```

这里，在上面的查询中，我们已经添加了基于`studentId.` 的过滤查询。一旦我们获得了过滤后的文档，我们就使用$set 操作符更新`totalMarks`。除此之外，我们使用`$push`操作符将新的主题数据插入到`subjectDetails`数组中。

因此，上述查询将返回以下输出:

```java
{
    "acknowledged":true,
    "matchedCount":1,
    "modifiedCount":1
}
```

这里，`matchedCount`包含匹配过滤器的文档数，而`modifiedCount`包含修改的文档数。

## 5.Java 驱动程序代码

到目前为止，我们讨论了一起使用`$push`和`$set`操作符的 mongo shell 查询。在这里，我们将学习使用 Java 驱动程序代码实现同样的功能。

在我们继续之前，让我们首先连接到数据库和所需的集合:

```java
MongoClient mongoClient = new MongoClient(new MongoClientURI("localhost", 27017);
MongoDatabase database = mongoClient.getDatabase("baeldung");
MongoCollection<Document> collection = database.getCollection("marks");
```

在这里，我们连接到 MongoDB，它运行在本地主机的默认端口 27017 上。

现在让我们来看看 Java 驱动程序代码:

```java
Document subjectData = new Document()
  .append("subjectId", 126)
  .append("subjectName", "Java Programming")
  .append("marks", 70); 
UpdateResult updateQueryResult = collection.updateOne(Filters.eq("studentId", 1023), 
  Updates.combine(Updates.set("totalMarks", 170), 
  Updates.push("subjectDetails", subjectData)));
```

在这个代码片段中，我们使用了`updateOne`方法，该方法根据应用的过滤器`studentId` 1023 只更新一个文档。然后，我们使用`Updates.combine`在一个调用中执行多个操作。字段`totalMarks`将被更新为 170，一个新的文档`subjectData`将被推送到数组字段`“subjectDetails”`。

## 6.结论

在本文中，我们理解了在单个 MongoDB 查询中一起应用多个操作的用例。此外，我们使用 MongoDB shell 查询和 Java 驱动程序代码执行了同样的操作。

与往常一样，所有示例的源代码和代码片段都可以在 GitHub 上的[处获得。](https://web.archive.org/web/20221008212435/https://github.com/eugenp/tutorials/tree/master/persistence-modules/java-mongodb-2)