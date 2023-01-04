# MongoDB 中的推送操作

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/mongodb-push-operations>

## 1.概观

在本教程中，我们将介绍如何在 [MongoDB](https://web.archive.org/web/20221009191031/https://www.mongodb.com/) 中将文档插入数组。此外，我们将看到`$push and` `$addToset`操作符在数组中的各种应用。

首先，我们将创建一个示例数据库，一个集合，并在其中插入虚拟数据。此外，我们将研究几个使用`$push`操作符更新文档的基本例子。稍后，我们还将讨论`$push`和`$addtoSet`操作符的各种用例。

让我们深入研究在 MongoDB 中将文档插入数组的各种方法。

## 2.数据库初始化

首先，让我们建立一个新的数据库`baeldung`和一个样本集合`orders`:

```java
use baeldung;
db.createCollection(orders);
```

现在让我们使用`insertMany`方法向集合中添加一些文档:

```java
db.orders.insertMany([
    {
        "customerId": 1023,
        "orderTimestamp": NumberLong("1646460073000"),
        "shippingDestination": "336, Street No.1 Pawai Mumbai",
        "purchaseOrder": 1000,
        "contactNumber":"9898987676",
        "items": [ 
            {
                "itemName": "BERGER",
                "quantity": 1,
                "price": 500
            },
            {
                "itemName": "VEG PIZZA",
                "quantity": 1,
                "price": 800
            } 
          ]
    },
    {
        "customerId": 1027,
        "orderTimestamp": NumberLong("1646460087000"),
        "shippingDestination": "445, Street No.2 Pawai Mumbai",
        "purchaseOrder": 2000,
        "contactNumber":"9898987676",
        "items": [ 
            {
               "itemName": "BERGER",
               "quantity": 1,
               "price": 500
            },
            {
               "itemName": "NON-VEG PIZZA",
               "quantity": 1,
               "price": 1200
            } 
          ]
    }
]);
```

如果插入成功，上面的命令将打印一个 JSON，如下所示:

```java
{
    "acknowledged" : true,
    "insertedIds" : [
        ObjectId("622300cc85e943405d04b567"),
	ObjectId("622300cc85e943405d04b568")
    ]
}
```

到目前为止，我们已经成功地建立了数据库和收集。我们将在所有的例子中使用这个数据库和集合。

## 3.使用 Mongo 查询的推送操作

MongoDB 提供了各种类型的数组操作符来更新 MongoDB 文档中的数组。**MongoDB 中的 [`$push`](https://web.archive.org/web/20221009191031/https://docs.mongodb.com/manual/reference/operator/update/push/) 运算符将值追加到数组的末尾。**根据查询的类型，我们可以在`updateOne`、`updateMany`、`findAndModify`等方法中使用`$push`操作符。

现在让我们使用`$push`操作符来研究 shell 查询:

```java
db.orders.updateOne(
    {
        "customerId": 1023
    },
    {
        $push: {
            "items":{
                "itemName": "PIZZA MANIA",
                "quantity": 1,
                "price": 800
            }
        }
    });
```

上述查询将返回以下文档:

```java
{
    "acknowledged":true,
    "matchedCount":1,
    "modifiedCount":1
}
```

现在让我们检查带有`customerId ` 1023 的文档。在这里，我们可以看到新条目被插入到列表"`items`"的末尾:

```java
{
    "customerId" : 1023,
    "orderTimestamp" : NumberLong("1646460073000"),
    "shippingDestination" : "336, Street No.1 Pawai Mumbai",
    "purchaseOrder" : 1000,
    "contactNumber" : "9898987676",
    "items" : [
        {
            "itemName" : "BERGER",
            "quantity" : 1,
	    "price" : 500
        },
	{
            "itemName" : "VEG PIZZA",
	    "quantity" : 1,
	    "price" : 800
	},
	{
	    "itemName" : "PIZZA MANIA",
	    "quantity" : 1,
	    "price" : 800
        }
    ]
}
```

## 4.使用 Java 驱动程序代码的推送操作

到目前为止，我们已经讨论了将文档推入数组的 MongoDB shell 查询。现在让我们使用 Java 代码实现推送更新查询。

在执行更新操作之前，让我们首先连接到`baeldung`数据库中的`orders`集合:

```java
mongoClient = new MongoClient("localhost", 27017);
MongoDatabase database = mongoClient.getDatabase("baeldung");
MongoCollection<Document> collection = database.getCollection("orders");
```

在这里，在这种情况下，我们连接到 MongoDB，它运行在本地主机的默认端口 27017 上。

### 4.1.使用`DBObject`

[MongoDB Java 驱动](/web/20221009191031/https://www.baeldung.com/java-mongodb)提供了对`DBObject`和`BSON`文档的支持。这里，`DBObject`是 MongoDB 遗留驱动程序的一部分，但是**在 MongoDB 的新版本中被废弃了。**

现在让我们看一下 Java 驱动程序代码，将新值插入数组:

```java
DBObject listItem = new BasicDBObject("items", new BasicDBObject("itemName", "PIZZA MANIA")
  .append("quantity", 1)
  .append("price", 800));
BasicDBObject searchFilter = new BasicDBObject("customerId", 1023);
BasicDBObject updateQuery = new BasicDBObject();
updateQuery.append("$push", listItem);
UpdateResult updateResult = collection.updateOne(searchFilter, updateQuery);
```

在上面的查询中，我们首先使用`BasicDBObject`创建了项目文档。在`searchQuery,`的基础上，集合的文档将被过滤，并且值将被推入数组。

### 4.2.使用`BSON`文件

[`BSON`](/web/20221009191031/https://www.baeldung.com/mongodb-bson) 文档是用 Java 访问 MongoDB 文档的新方法，它是用新的客户端栈构建的。`org.bson.Document`类不太复杂，更容易使用。

让我们使用`org.bson.Document `类将值推入数组"`items”`:

```java
Document item = new Document()
  .append("itemName1", "PIZZA MANIA")
  .append("quantity", 1).append("price", 800);
UpdateResult updateResult = collection.updateOne(Filters.eq("customerId", 1023), Updates.push("items", item));
```

在这种情况下，`BSON`的实现类似于使用`DBObject,`运行的代码，更新也是一样的。这里，我们使用了`updateOne`方法来更新单个文档。

## 5.使用`addToSet`操作符

[`$addToSet`](https://web.archive.org/web/20221009191031/https://docs.mongodb.com/manual/reference/operator/update/addToSet/) 操作符也可以用来在数组中推送一个值。只有当数组中不存在该值时，该运算符才会添加值。否则，它只会忽略它。而 push 操作符会将值作为过滤条件进行推送，以获得匹配。

**需要注意的一个关键点是`$addToSet` 操作符不会在出现重复项的情况下推送值工作。**另一方面，`$push operator` 只是将值推入数组，而不考虑任何其他条件。

### 5.1.使用`addToSet`操作符的 Shell 查询

mongo shell 查询的`$addToSet`操作符类似于`$push`操作符，但是`$addToSet`没有在数组中插入重复值。

现在让我们来看看使用`$addToset`将值推入数组的 MongoDB 查询:

```java
db.orders.updateOne(
    {
        "customerId": 1023
    },
    {
        $addToSet: {
            "items":{
                "itemName": "PASTA",
                "quantity": 1,
                "price": 1000
            }
        }
    });
```

在这种情况下，输出如下:

```java
{
    "acknowledged":true,
    "matchedCount":1,
    "modifiedCount":1
} 
```

在这种情况下，我们使用了`$addToSet`操作符，只有当文档是惟一的时，它才会被推送到数组“items”中。

### 5.2.使用`addToSet`操作符的 Java 驱动程序

与 push 操作符相比,＄`addToSet`操作符提供了一种不同类型的数组更新操作:

```java
Document item = new Document()
  .append("itemName1", "PIZZA MANIA")
  .append("quantity", 1).append("price", 800);
UpdateResult updateResult = collection
  .updateOne(Filters.eq("customerId", 1023), Updates.addToSet("items", item));
System.out.println("updateResult:- " + updateResult);
```

在上面的代码中，首先我们创建了文档“`item`”，在`customerId`过滤器的基础上，`updateOne`方法会尝试将文档“`item`”推入数组“`items`”。

## 6.结论

在本文中，我们学习了使用`$push and $addToSet`操作符将新值推入数组。首先，我们研究了 MongoDB shell 查询中`$push`操作符的使用，然后我们讨论了相应的 Java 驱动程序代码。

所有案例的实现都可以在 GitHub 上找到[。](https://web.archive.org/web/20221009191031/https://github.com/eugenp/tutorials/tree/master/persistence-modules/java-mongodb-2)