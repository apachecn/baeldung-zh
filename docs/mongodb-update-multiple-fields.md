# 更新 MongoDB 文档中的多个字段

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/mongodb-update-multiple-fields>

## 1.概观

MongoDB 是一个面向文档的 NoSQL 数据库，是公开可用的。我们可以使用各种方法更新集合中的文档，如`update`、`replace`和`save`。为了改变文档的特定字段，我们将使用不同的操作符，如`$set`、`$inc,`等。

在本教程中，我们将学习使用`update`和`replace`查询来修改文档的多个字段。出于演示的目的，我们将首先讨论 mongo shell 查询，然后讨论它在 Java 中的相应实现。

现在让我们来看看实现这一目的的各种方法。

## 2.用于更新不同字段的 Shell 查询

在开始之前，让我们首先创建一个新的数据库`baeldung`和一个样本集合`employee`。我们将在所有示例中使用该集合:

```java
use baeldung;
db.createCollection(employee);
```

现在让我们使用`insertMany`查询将一些文档添加到这个集合中:

```java
db.employee.insertMany([
    {
        "employee_id": 794875,
        "employee_name": "David Smith",
        "job": "Sales Representative",
        "department_id": 2,
        "salary": 20000,
        "hire_date": NumberLong("1643969311817")
    },
    {
        "employee_id": 794876,
        "employee_name": "Joe Butler",
        "job": "Sales Manager",
        "department_id": 3,
        "salary": 30000,
        "hire_date": NumberLong("1645338658000")
    }
]);
```

因此，我们将为这两个文档获得一个带有 ObjectId 的 JSON，如下所示:

```java
{
    "acknowledged": true,
    "insertedIds": [
        ObjectId("6211e034b76b996845f3193d"),
        ObjectId("6211e034b76b996845f3193e")
        ]
}
```

到目前为止，我们已经建立了所需的环境。现在让我们更新刚刚插入的文档。

### 2.1.更新单个文档的多个字段

我们可以使用`$set`和`$inc`操作符来更新 MongoDB 中的任何字段。`$set`操作者将设置新的指定值，而`$inc`操作者将增加指定值。

让我们首先来看看使用`$set`操作符更新雇员集合的两个字段的 MongoDB 查询:

```java
db.employee.updateOne(
    {
        "employee_id": 794875,
        "employee_name": "David Smith"
    },
    {
        $set:{
            department_id:3,
            job:"Sales Manager"
        }
    }
);
```

在上面的查询中，`employee_id`和`employee_name`字段用于过滤文档，而`$set`操作符用于更新`job`和`department_id`字段。

我们还可以在一个更新查询中同时使用`$set`和`$inc`操作符:

```java
db.employee.updateOne(
    {
        "employee_id": 794875
    },
    {
        $inc: {
            department_id: 1
        },
        $set: {
            job: "Sales Manager"
        }
    }
);
```

这将把`job`字段更新为销售经理，并将`department_id`加 1。

### 2.2.更新多个文档的多个字段

此外，我们还可以在 MongoDB 中更新多个文档的多个字段。我们只需要包含选项`multi:true`来修改所有符合过滤查询标准的文档:

```java
db.employee.update(
    {
        "job": "Sales Representative"
    },
    {
        $inc: { 
            salary: 10000
        }, 
        $set: { 
            department_id: 5
        }
    },
    {
        multi: true 
    }
);
```

或者，我们将使用`updateMany`查询得到相同的结果:

```java
db.employee.updateMany(
    {
        "job": "Sales Representative"
    },
    {
        $inc: {
            salary: 10000
        },
        $set: {
            department_id: 5
        }
    }
);
```

在上面的查询中，我们使用了`updateMany`方法来更新集合中的多个文档。

### 2.3.更新多个字段时的常见问题

到目前为止，我们已经学会了通过提供两个不同的操作符或对多个字段使用单个操作符来使用更新查询来更新多个字段。

现在，如果我们在一个查询中对不同的字段多次使用一个操作符， **MongoDB 将只更新更新查询**的最后一条语句，而忽略其余的:

```java
db.employee.updateMany(
    {
        "employee_id": 794875
    },
    {
        $set: {
            department_id: 3
        },
        $set: {
            job:"Sales Manager"
        }
    }
);
```

上述查询将返回与此类似的输出:

```java
{
    "acknowledged":true,
    "matchedCount":1,
    "modifiedCount":1
}
```

在这种情况下，唯一的`job`将更新为“销售经理”。`department_id`值不会更新为 3。

## 3.用 Java 驱动程序更新字段

到目前为止，我们已经讨论了原始的 MongoDB 查询。现在让我们使用 Java 来执行同样的操作[。MongoDB Java 驱动程序支持两个类来表示一个 MongoDB 文档，`com.mongodb.BasicDBObject`和`org.bson.Document.`我们将研究这两种方法来更新文档中的字段。](/web/20220524063523/https://www.baeldung.com/queries-in-spring-data-mongodb)

在我们继续之前，让我们首先连接到`baeldung` DB 中的`employee`集合:

```java
MongoClient mongoClient = new MongoClient(new MongoClientURI("localhost", 27017);
MongoDatabase database = mongoClient.getDatabase("baeldung");
MongoCollection<Document> collection = database.getCollection("employee");
```

这里，我们假设 MongoDB 运行在本地的默认端口 27017 上。

### 3.1.使用`DBObject`

为了在 MongoDB 中创建文档，我们将使用`com.mongodb.` DBObject 接口及其实现类`com.mongodb.BasicDBObject`。

`DBObject`的实现基于键值对。**`BasicDBObject`继承自`util`包中的`LinkedHashMap`类。**

现在让我们使用`com.mongodb.BasicDBObject`对多个字段执行更新操作:

```java
BasicDBObject searchQuery = new BasicDBObject("employee_id", 794875);
BasicDBObject updateFields = new BasicDBObject();
updateFields.append("department_id", 3);
updateFields.append("job", "Sales Manager");
BasicDBObject setQuery = new BasicDBObject();
setQuery.append("$set", updateFields);
UpdateResult updateResult = collection.updateMany(searchQuery, setQuery); 
```

这里，首先，我们在`employee_id.` 的基础上创建了一个过滤查询，这个操作将返回一组文档。此外，我们已经根据设置的查询更新了`department_id`和`job`的值。

### 3.2.使用`bson`文件

我们可以使用 [`bson`](https://web.archive.org/web/20220524063523/https://baeldung-cn.com/mongodb-bson) 文档执行所有的 MongoDB 操作。为此，首先，我们需要集合对象，然后使用带有`filter`和`set`函数的`updateMany`方法执行更新操作。

```java
UpdateResult updateQueryResult = collection.updateMany(Filters.eq("employee_id", 794875),
Updates.combine(Updates.set("department_id", 3), Updates.set("job", "Sales Manager")));
```

这里，我们将一个查询过滤器传递给`updateMany`方法。`eq`过滤器将`employee_id`与完全匹配的文本‘794875’进行匹配。然后，我们使用`set`操作符更新`department_id`和`job`。

## 4.使用替换查询

更新文档的多个字段的简单方法是用一个具有更新值的新文档来替换它。

例如，如果我们希望用`employee_id` 794875 替换一个文档，我们可以执行下面的查询:

```java
db.employee.replaceOne(
    {
        "employee_id": 794875
    },
    {
        "employee_id": 794875,
        "employee_name": "David Smith",
        "job": "Sales Manager",
        "department_id": 3,
        "salary": 30000,
        "hire_date": NumberLong("1643969311817")
    }
);
```

上面的命令将在输出中打印一个确认 JSON:

```java
{
    "acknowledged":true,
    "matchedCount":1,
    "modifiedCount":1
}
```

这里，`employee_id`字段用于过滤文档。更新查询的第二个参数表示现有文档将被替换的文档。

在上面的查询中，我们正在执行`replaceOne`，因此，它将用那个过滤器只替换一个文档。或者，如果我们想用过滤查询替换所有文档，那么我们需要使用`updateMany`方法。

## 5.结论

在本文中，我们探索了在 MongoDB 中更新文档的多个字段的各种方法。我们广泛地讨论了两种实现，使用 MongoDB shell 和使用 Java 驱动程序。

有各种选项可以更新文档的多个字段，包括`$inc `和`$set`操作符。

所有这些例子和代码片段的实现都可以在 GitHub 上找到[。](https://web.archive.org/web/20220524063523/https://github.com/eugenp/tutorials/tree/master/persistence-modules/java-mongodb)