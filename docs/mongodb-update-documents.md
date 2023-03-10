# 更新 MongoDB 中的文档

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/mongodb-update-documents>

## 1.概观

MongoDB 是一个跨平台、面向文档、开源的 NoSQL 数据库，用 C++编写。此外，MongoDB 提供了高性能、高可用性和自动伸缩。

为了更新 MongoDB 中的文档，我们可以使用不同的方法，如`updateOne`、`findOneAndUpdate,`等。此外，MongoDB 为更新方法提供了各种操作符。

在本教程中，我们将讨论在 MongoDB 中执行更新操作的不同方法。对于每种方法，我们将首先讨论 mongo shell 查询，然后讨论它在 Java 中的实现。

## 2.数据库设置

在我们继续更新查询之前，让我们首先创建一个数据库`baeldung`和一个样本集合`student:`

```java
use baeldung;
db.createCollection(student);
```

作为一个例子，让我们使用`insertMany`查询将一些文档添加到集合`student`中:

```java
db.student.insertMany([
    {
        "student_id": 8764,
        "student_name": "Paul Starc",
        "address": "Hostel 1",
        "age": 16,
        "roll_no":199406
    },
    {
        "student_id": 8765,
        "student_name": "Andrew Boult",
        "address": "Hostel 2",
        "age": 18,
        "roll_no":199408
    }
]);
```

成功插入后，我们将获得一个带有`acknowledged:true`的 JSON:

```java
{
    "acknowledged" : true,
    "insertedIds" : [
        ObjectId("621b078485e943405d04b557"),
	ObjectId("621b078485e943405d04b558")
    ]
}
```

现在让我们深入研究在 MongoDB 中更新文档的不同方法。

## 3.使用`updateOne`方法

MongoDB 中的更新操作可以通过添加新字段、删除字段或更新现有字段来完成。`[updateOne](https://web.archive.org/web/20221008232507/https://docs.mongodb.com/manual/reference/method/db.collection.updateOne/) `方法根据应用的查询过滤器更新集合中的单个文档。它首先找到匹配过滤器的文档，然后更新指定的字段。

另外，**我们可以使用不同的运算符如`$set`、`$unset`、`$inc`等。，使用更新方法。**

为了进行演示，我们来看看更新集合中的单个文档的查询:

```java
db.student.updateOne(
    { 
        "student_name" : "Paul Starc"
    },
    { 
        $set: {
            "address" : "Hostel 2"
        }
    }
 );
```

我们将得到类似如下所示的输出:

```java
{
    "acknowledged":true,
    "matchedCount":1,
    "modifiedCount":1
}
```

现在让我们来看看上面的`updateOne`查询的 Java 驱动程序代码:

```java
UpdateResult updateResult = collection.updateOne(Filters.eq("student_name", "Paul Starc"),
Updates.set("address", "Hostel 2"));
```

这里，首先我们使用了`student_name`字段来过滤文档。然后我们用`student_name`“Paul Starc”更新文档的地址。

## 4.使用`updateMany`方法

[`updateMany`](https://web.archive.org/web/20221008232507/https://docs.mongodb.com/manual/reference/method/db.collection.updateMany/) 方法更新 MongoDB 集合中匹配给定过滤器的所有文档。**使用`updateMany`的好处之一是我们可以更新多个文档，而不会丢失旧文档的字段。**

让我们看看使用`updateMany`方法的 MongoDB shell 查询:

```java
db.student.updateMany(
    { 
        age: { 
            $lt: 20
         } 
    },
    { 
        $set:{ 
            "Review" : true 
        }
    }
);
```

上述命令将返回以下输出:

```java
{
    "acknowledged":true,
    "matchedCount":2,
    "modifiedCount":2
}
```

这里，`matchedCount`包含匹配文档的数量，而`modifiedCount`包含修改的文档数量。

现在让我们使用`updateMany`方法来研究 Java 驱动程序代码:

```java
UpdateResult updateResult = collection.updateMany(Filters.lt("age", 20), Updates.set("Review", true));
```

这里将过滤掉所有`age`小于 20 的文档，并将`Review`字段设置为`true`。

## 5.使用`replaceOne`方法

MongoDB 的 [`replaceOne`](https://web.archive.org/web/20221008232507/https://docs.mongodb.com/manual/reference/method/db.collection.replaceOne/) 方法替换整个文档。**`replaceOne`的缺点之一是所有旧字段将被新字段替换，并且旧字段也将丢失:**

```java
db.student.replaceOne(
    { 
        "student_id": 8764
    },
    {
        "student_id": 8764,
        "student_name": "Paul Starc",
        "address": "Hostel 2",
        "age": 18,
        "roll_no":199406
    }
);
```

在这种情况下，我们将得到以下输出:

```java
{
    "acknowledged":true,
    "matchedCount":1,
    "modifiedCount":1
}
```

如果没有找到匹配，该操作将`matchedCount`返回为 0:

```java
{
    "acknowledged":true,
    "matchedCount":0,
    "modifiedCount":0
}
```

让我们使用`replaceOne`方法编写相应的 Java 驱动程序代码:

```java
Document replaceDocument = new Document();
replaceDocument
  .append("student_id", 8764)
  .append("student_name", "Paul Starc")
  .append("address", "Hostel 2")
  .append("age",18)
  .append("roll_no", 199406);
UpdateResult updateResult = collection.replaceOne(Filters.eq("student_id", 8764), replaceDocument);
```

在上面的代码中，我们创建了一个文档，旧的文档将被替换。带有`student_id` 8764 的文档将被新创建的文档替换。

## 6.使用`findOneAndReplace`方法

[`findOneAndReplace`](https://web.archive.org/web/20221008232507/https://docs.mongodb.com/manual/reference/method/db.collection.findOneAndReplace/) 方法是 MongoDB 提供的高级更新方法之一，它根据给定的选择标准替换第一个匹配的文档。默认情况下，此方法返回原始文档。如果需要，我们可以使用`findOneAndReplace`的不同选项对文档进行分类和投影。

简而言之，`findOneAndReplace`根据应用的过滤器替换集合中的第一个匹配文档:

```java
db.student.findOneAndReplace(
    { 
        "student_id" : { 
            $eq : 8764 
        }
    },
    { 
        "student_id" : 8764,
        "student_name" : "Paul Starc",
        "address": "Hostel 2",
        "age": 18,
        "roll_no":199406 
    },
    {
        returnNewDocument: false
    }
);
```

该查询将返回以下文档:

```java
{
    "student_id":8764,
    "student_name":"Paul Starc",
    "address":"Hostel 1",
    "age":16,
    "roll_no":199406
}
```

如果我们将`returnNewDocument`设置为`true`，那么操作将返回被替换的文档:

```java
{
    "student_id":8764,
    "student_name":"Paul Starc",
    "address":"Hostel 2",
    "age":18,
    "roll_no":199406
}
```

现在让我们使用`findOneAndReplace`方法来投影返回文档中的`student_id`和`age`字段:

```java
db.student.findOneAndReplace(
    { 
        "student_id" : {
        $eq : 8764 
        } 
    },
    { 
        "student_id" : 8764, 
        "student_name" : "Paul Starc",
        "address": "Hostel 2",
        "age": 18,
        "roll_no":199406 
    },
    { 
        projection: { 
            "_id" : 0,
            "student_id":1,
            "age" : 1 
        } 
    }
);
```

上述查询的输出将只包含投影的字段:

```java
{
    "student_id":"8764",
    "age":16
}
```

上面的 Java 驱动代码查询了`findOneAndReplace:`的各个选项

```java
Document replaceDocument = new Document();
replaceDocument
  .append("student_id", 8764)
  .append("student_name", "Paul Starc")
  .append("address", "Hostel 2")
  .append("age", 18)
  .append("roll_no", 199406);
Document sort = new Document("roll_no", 1);
Document projection = new Document("_id", 0).append("student_id", 1).append("address", 1);
Document resultDocument = collection.findOneAndReplace(
  Filters.eq("student_id", 8764), 
  replaceDocument,
  new FindOneAndReplaceOptions().upsert(true).sort(sort).projection(projection).returnDocument(ReturnDocument.AFTER)); 
```

在上面的查询中，`findOneAndReplace`方法将首先根据`roll_no,` 对文档进行升序排序，新创建的文档将替换为带有`student_id`“8764”的文档。

## 7.使用`findOneAndUpdate`方法

[`findOneAndUpdate`](https://web.archive.org/web/20221008232507/https://docs.mongodb.com/manual/reference/method/db.collection.findOneAndUpdate/) 方法更新集合中第一个匹配的文档。如果有多个文档与选择标准匹配，那么它只更新第一个匹配的文档。当我们更新文档时，`_id`字段的值保持不变:

```java
db.student.findOneAndUpdate(
    { 
        "student_id" : 8764
    },
    { 
        $inc : { 
            "roll_no" : 5
        } 
    },
    { 
        sort: { 
            "roll_no" : 1 
        }, 
        projection: { 
            "_id" : 0,
            "student_id":1,
            "address" : 1
        }
    }
);

```

查询的输出将只包含旧文档的`studentId`和`address`:

```java
{
    "student_id":8764,
    "address":"Hostel 1"
} 
```

上面查询的 Java 驱动代码，使用`findOneAndUpdate` 的不同选项如下`:`

```java
Document sort = new Document("roll_no", 1);
Document projection = new Document("_id", 0).append("student_id", 1).append("address", 1);
Document resultDocument = collection.findOneAndUpdate(
  Filters.eq("student_id", 8764),
  Updates.inc("roll_no", 5), 
  new FindOneAndUpdateOptions().sort(sort).projection(projection).returnDocument(ReturnDocument.BEFORE));
```

在这种情况下，`findOneAndUpdate`方法将首先根据`roll_no`对文档进行升序排序。上面的查询将`roll_no `递增 5，然后返回`student_id`和`address`字段。

## 8.结论

在本文中，我们看到了在 MongoDB 中更新文档的各种方法。首先，我们研究了 MongoDB shell 查询，然后讨论了相应的 Java 驱动程序代码。

所有这些例子和代码片段的实现都可以在 GitHub 上找到[。](https://web.archive.org/web/20221008232507/https://github.com/eugenp/tutorials/tree/master/persistence-modules/java-mongodb)