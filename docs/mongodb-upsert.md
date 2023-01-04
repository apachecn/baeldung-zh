# MongoDB 中的 Upsert 指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/mongodb-upsert>

## 1.概观

**Upsert 是 insert 和 update 的组合(inSERT + UPdate = upsert)。**我们可以使用不同更新方式的`upsert` ，即`update`、`findAndModify`和`replaceOne`。

在 [MongoDB 中，](https://web.archive.org/web/20220524054354/https://www.mongodb.com/)`upsert`选项是一个`Boolean`值。假设值是`true`，文档匹配指定的查询过滤器。在这种情况下，应用的更新操作将更新文档。如果值是`true`并且没有文档符合条件，那么这个选项会在集合中插入一个新文档。新文档将包含基于过滤器和应用操作的字段。

在本教程中，我们将首先查看 MongoDB Shell 查询中的`upsert`,然后使用 Java 驱动程序代码。

## 2.数据库初始化

在我们继续执行`upsert`操作之前，首先我们需要建立一个新的数据库`baeldung`和一个样本集合`vehicle`:

```
db.vehicle.insertMany([
{
    "companyName":"Nissan", 
    "modelName":"GTR",
    "launchYear":2016,
    "type":"Sports",
    "registeredNo":"EPS 5561"
},
{ 
    "companyName":"BMW",
    "modelName":"X5",
    "launchYear":2020,
    "type":"SUV",
    "registeredNo":"LLS 6899"
},
{
    "companyName":"Honda",
    "modelName":"Gold Wing",
    "launchYear":2018,
    "type":"Bike",
    "registeredNo":"LKS 2477"
}]);
```

如果插入成功，上面的命令将打印一个 JSON，如下所示:

```
{
    "acknowledged" : true, 
    "insertedIds" : [
        ObjectId("623c1db39d55d4e137e4781b"),
	ObjectId("623c1db39d55d4e137e4781c"),
	ObjectId("623c1db39d55d4e137e4781d")
    ]
}
```

我们已经成功地将虚拟数据添加到集合`vehicle`中。

## 3.使用`update`方法

在这一节中，我们将学习在`update`方法中使用`upsert`选项。**`upsert`选项的主要目的是根据应用的过滤器更新现有文档，或者如果过滤器没有得到**的匹配，则插入一个新文档。

作为一个例子，我们将使用带有`upsert`选项的`$setOnInsert`操作符来获得在文档中插入新字段的额外优势。

让我们检查一个查询，其中的过滤条件与集合中的现有文档相匹配:

```
db.vehicle.update(
{
    "modelName":"X5"
},
{
    "$set":{
        "companyName":"Hero Honda"
    }
},
{
    "upsert":true
});
```

上述查询将返回以下文档:

```
{ 
    "nMatched" : 1, 
    "nUpserted" : 0,
    "nModified" : 1 
}
```

在这里，我们将看到与上述 mongo shell 查询相对应的 Java 驱动程序代码:

```
UpdateOptions options = new UpdateOptions().upsert(true);
UpdateResult updateResult = collection.updateOne(Filters.eq("modelName", "X5"), 
  Updates.combine(Updates.set("companyName", "Hero Honda")), options);
System.out.println("updateResult:- " + updateResult);
```

在上面的查询中，字段`modelName`“X5”已经存在于集合中，因此该文档的字段`companyName`将被更新为“Hero Honda”。

现在让我们看看使用$ `setOnInsert`操作符的`upsert`选项的例子。它仅适用于添加新文档的情况:

```
db.vehicle.update(
{
    "modelName":"GTPR"
},
{
    "$set":{
        "companyName":"Hero Honda"
    },
    "$setOnInsert":{
        "launchYear" : 2022,
	"type" : "Bike",
	"registeredNo" : "EPS 5562"
    },  
},
{
    "upsert":true
});
```

上述查询将返回以下文档:

```
{
    "nMatched" : 0,
    "nUpserted" : 1,
    "nModified" : 0,
    "_id" : ObjectId("623b378ed648af670fe50e7f")
}
```

上面带有`$setOnInsert`选项的更新查询的 Java 驱动程序代码将是:

```
UpdateResult updateSetOnInsertResult = collection.updateOne(Filters.eq("modelName", "GTPR"),
  Updates.combine(Updates.set("companyName", "Hero Honda"),
  Updates.setOnInsert("launchYear", 2022),
  Updates.setOnInsert("type", "Bike"),
  Updates.setOnInsert("registeredNo", "EPS 5562")), options);
System.out.println("updateSetOnInsertResult:- " + updateSetOnInsertResult);
```

这里，在上面的查询中，字段`modelName`“GTPR”的过滤条件不匹配任何集合文档，所以我们将向集合添加一个新文档。**需要注意的关键点是`$setOnInsert`将所有字段添加到新文档中。**

## 4.使用`findAndModify`方法

我们也可以通过使用`findAndModify`方法来使用`upsert`选项。对于这种方法，`upsert`选项的默认值是`false`。如果我们将`upsert`选项设置为`true`，它将与更新方法完全一样。

让我们看看带有`upsert`选项`true`的`findAndModify`方法的一个用例:

```
db.vehicle.findAndModify(
{
    query:{
        "modelName":"X7"
    },
    update: {
        "$set":{
            "companyName":"Hero Honda"
        }
    },
    "upsert":true,
    "new":true
});
```

在这种情况下，上述查询将返回新创建的文档。让我们来看看上面查询的 Java 驱动程序代码:

```
FindOneAndUpdateOptions upsertOptions = new FindOneAndUpdateOptions();
  upsertOptions.returnDocument(ReturnDocument.AFTER);
  upsertOptions.upsert(true);
Document resultDocument = collection.findOneAndUpdate(Filters.eq("modelName", "X7"),
  Updates.set("companyName", "Hero Honda"), upsertOptions);
System.out.println("resultDocument:- " + resultDocument);
```

在这里，我们首先创建了过滤条件，基于此，我们要么更新现有文档，要么向集合`vehicle`添加新文档。

## 5.使用`replaceOne`方法

让我们使用`replaceOne`方法执行`upsert`操作。如果条件匹配，MongoDB 的`replaceOne`方法只是替换集合中的单个文档。

首先，让我们看看 replace 方法的 Mongo shell 查询:

```
db.vehicle.replaceOne(
{
    "modelName":"GTPR"
},
{
    "modelName" : "GTPR",
    "companyName" : "Hero Honda",
    "launchYear" : 2022,
    "type" : "Bike",
    "registeredNo" : "EPS 5562"
},
{
    "upsert":true
});
```

上述查询将返回以下响应:

```
{ 
    "acknowledged" : true, 
    "matchedCount" : 1,
    "modifiedCount" : 1 
}
```

现在，让我们使用 Java 驱动程序代码编写上面的查询:

```
Document replaceDocument = new Document();
replaceDocument.append("modelName", "GTPP")
  .append("companyName", "Hero Honda")
  .append("launchYear", 2022)
  .append("type", "Bike")
  .append("registeredNo", "EPS 5562");
UpdateResult updateReplaceResult = collection.replaceOne(Filters.eq("modelName", "GTPP"), replaceDocument, options);
System.out.println("updateReplaceResult:- " + updateReplaceResult);
```

在这里，在这种情况下，我们首先需要创建一个新文档来替换现有的文档，使用`upsert`选项`true`，我们将只在条件匹配时替换文档。

## 6.结论

在本文中，我们已经看到了如何使用 MongoDB 的各种更新方法执行`upsert`操作。首先，我们学习用`update`和`findAndModify`方法执行`upsert`，然后使用`replaceOne`方法。简而言之，我们使用 Mongo shell 查询和 Java 驱动程序代码实现了 upsert 操作。

所有案例的实现都可以在 GitHub 上找到[。](https://web.archive.org/web/20220524054354/https://github.com/eugenp/tutorials/tree/master/persistence-modules/java-mongodb-2)