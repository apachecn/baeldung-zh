# Spring Data MongoDB 中的 GridFS

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-data-mongodb-gridfs>

## 1。概述

本教程将探讨 Spring Data MongoDB 的**核心特性之一:与** `**GridFS**.` 交互

GridFS 存储规范主要用于处理超过 16MB 文档大小限制的文件。Spring Data 提供了一个`GridFsOperations`接口及其实现`– GridFsTemplate`——来轻松地与这个文件系统交互。

## 2。配置

### 2.1。XML 配置

让我们从`GridFsTemplate`的简单 XML 配置开始:

```java
<bean id="gridFsTemplate" class="org.springframework.data.mongodb.gridfs.GridFsTemplate">
    <constructor-arg ref="mongoDbFactory" />
    <constructor-arg ref="mongoConverter" />
</bean> 
```

`GridFsTemplate`的构造函数参数包括对`mongoDbFactory`和`mongoConverter`的 bean 引用，前者创建一个 Mongo 数据库，后者在 Java 和 MongoDB 类型之间转换。它们的 bean 定义如下。

```java
<mongo:db-factory id="mongoDbFactory" dbname="test" mongo-client-ref="mongoClient" />

<mongo:mapping-converter id="mongoConverter" base-package="com.baeldung.converter">
    <mongo:custom-converters base-package="com.baeldung.converter"/>
</mongo:mapping-converter>
```

### 2.2。Java 配置

让我们创建一个类似的配置，只是使用 Java:

```java
@Configuration
@EnableMongoRepositories(basePackages = "com.baeldung.repository")
public class MongoConfig extends AbstractMongoClientConfiguration {
    @Autowired
    private MappingMongoConverter mongoConverter;

    @Bean
    public GridFsTemplate gridFsTemplate() throws Exception {
        return new GridFsTemplate(mongoDbFactory(), mongoConverter);
    }

    // ...
}
```

对于这个配置，我们使用了`mongoDbFactory()` 方法，并自动连接了父类`AbstractMongoClientConfiguration`中定义的 M `appingMongoConverter`。

## 3。`GridFsTemplate`核心方法

### 3.1。`store`

`store`方法将文件保存到 MongoDB 中。

假设我们有一个空数据库，并希望在其中存储一个文件:

```java
InputStream inputStream = new FileInputStream("src/main/resources/test.png"); 
gridFsTemplate.store(inputStream, "test.png", "image/png", metaData).toString();
```

注意，我们可以通过向`store`方法传递一个`DBObject`来保存额外的元数据和文件。对于我们的例子， *DBObject* 可能看起来像这样:

```java
DBObject metaData = new BasicDBObject();
metaData.put("user", "alex");
```

`GridFS`使用两个集合来存储文件元数据及其内容。文件的元数据存储在`files`集合中，文件的内容存储在`chunks`集合中。两个集合都以 *fs* 为前缀。

如果我们执行 MongoDB 命令`db[‘fs.files'].find()`，我们将看到`fs.files`集合:

```java
{
    "_id" : ObjectId("5602de6e5d8bba0d6f2e45e4"),
    "metadata" : {
        "user" : "alex"
    },
    "filename" : "test.png",
    "aliases" : null,
    "chunkSize" : NumberLong(261120),
    "uploadDate" : ISODate("2015-09-23T17:16:30.781Z"),
    "length" : NumberLong(855),
    "contentType" : "image/png",
    "md5" : "27c915db9aa031f1b27bb05021b695c6"
}
```

命令`db[‘fs.chunks'].find()` 检索文件的内容:

```java
{
    "_id" : ObjectId("5602de6e5d8bba0d6f2e45e4"),
    "files_id" : ObjectId("5602de6e5d8bba0d6f2e45e4"),
    "n" : 0,
    "data" : 
    { 
        "$binary" : "/9j/4AAQSkZJRgABAQAAAQABAAD/4QAqRXhpZgAASUkqAAgAAAABADEBAgAHAAAAGgAAAAAAAABHb29nbGUAAP/bAIQAAwICAwICAwMDAwQDAwQFCAUFBAQFCgcHBggM
          CgwMCwoLCw0OEhANDhEOCwsQFhARExQVFRUMDxcYFhQYEhQVFAEDBAQGBQUJBgYKDw4MDhQUFA8RDQwMEA0QDA8VDA0NDw0MDw4MDA0ODxAMDQ0MDAwODA8MDQ4NDA0NDAwNDAwQ/8AA
          EQgAHAAcAwERAAIRAQMRAf/EABgAAQEBAQEAAAAAAAAAAAAAAAgGBwUE/8QALBAAAgEDAgQFAwUAAAAAAAAAAQIDBAURBiEABwgSIjFBYXEyUYETFEKhw
          f/EABoBAAIDAQEAAAAAAAAAAAAAAAMEAQIFBgD/xAAiEQACAgEDBAMAAAAAAAAAAAAAAQIRAyIx8BJRYYETIUH/2gAMAwEAAhEDEQA/AHDyq1Bb6GjFPMAszLkZHHCTi1I6O
          cXOFRZ1ZqoX6aqzRClkhb9MGVh2SsNyVI/hjG5389tuGcUaLK1GmFfpn5r3rnXpfV82pGtS3a0XmaGOO3zguKV1SWDwBQDH2uUWTOWMZzuM8bS0VQtJKRb2li9LL3l+4VNQPEfQTOB/WO
          G1K0JtUzwad1eZaYBiqzL4S2N8cZUsa7DqlRGdWvMq5aX6b9Tvb5pIZbggt7VcU3YacSkDbfuLNuu3lkk+98GNfIrLt2gK9K/NWl5Z87Ldebj3R0NTa2tVVKhOI0KoQ5AG4DRqSPk+gHGn
          khUPYNOx92vW9PcrdDW0FUJqOp7po5ETIYMxOdyOAK0qAvcgKPWa0oMTo7SEYDKPp98/5wPoJsx3rZ1wLhojS9iinLD9w9W47iSwVe0Z3wfrPoce2eC4I6rCX9MxrpUpWqudNunUosNLR1EkiyIGDqUKF
          fyZB+AeG80riueQdVfObC/tN1pLdaLfSxMiRQ08aIg2CjtGAB9uEyCSqSWujICUXwghT57A5+ePEoMvUdc5a3XlSsgUhZGjGM/TGAqjz+SfuT7DDmGC6WzzeyOv0+2amOrr3KylzTUwjjDeWGbJJ9/COI
          yvRFFv1iRsVGDaqYGWVsIoBZydsDhQGf/Z", 
        "$type" : "00" 
    }
}
```

### 3.2。`findOne`

`findOne`只返回一个满足指定查询条件的文档。

```java
String id = "5602de6e5d8bba0d6f2e45e4";
GridFSFile gridFsFile = gridFsTemplate.findOne(new Query(Criteria.where("_id").is(id))); 
```

上面的代码将返回在上例中添加的结果记录。如果数据库包含多个与查询匹配的记录，它将只返回一个文档。将根据自然排序(文档在数据库中存储的顺序)选择返回的特定记录。

### 3.3。`find`

从集合中选择文档，并将光标返回到所选文档。

假设我们有以下数据库，包含 2 条记录:

```java
[
    {
        "_id" : ObjectId("5602de6e5d8bba0d6f2e45e4"),
        "metadata" : {
            "user" : "alex"
        },
        "filename" : "test.png",
        "aliases" : null,
        "chunkSize" : NumberLong(261120),
        "uploadDate" : ISODate("2015-09-23T17:16:30.781Z"),
        "length" : NumberLong(855),
        "contentType" : "image/png",
        "md5" : "27c915db9aa031f1b27bb05021b695c6"
    },
    {
        "_id" : ObjectId("5702deyu6d8bba0d6f2e45e4"),
        "metadata" : {
            "user" : "david"
        },
        "filename" : "test.png",
        "aliases" : null,
        "chunkSize" : NumberLong(261120),
        "uploadDate" : ISODate("2015-09-23T17:16:30.781Z"),
        "length" : NumberLong(855),
        "contentType" : "image/png",
        "md5" : "27c915db9aa031f1b27bb05021b695c6"
    }
]
```

如果我们使用`GridFsTemplate` 来执行下面的查询:

```java
List<GridFSFile> fileList = new ArrayList<GridFSFile>();
gridFsTemplate.find(new Query()).into(fileList);
```

结果列表应该包含两个记录，因为我们没有提供标准。

当然，我们可以为`find`方法提供一些标准。例如，如果我们想要获取元数据包含名为`alex` `,`的用户的文件，代码应该是:

```java
List<GridFSFile> gridFSFiles = new ArrayList<GridFSFile>();
gridFsTemplate.find(new Query(Criteria.where("metadata.user").is("alex"))).into(gridFSFiles);
```

结果列表将只包含一条记录。

### 3.4。`delete`

从收藏中删除文档。

使用前面示例中的数据库，假设我们有代码:

```java
String id = "5702deyu6d8bba0d6f2e45e4";
gridFsTemplate.delete(new Query(Criteria.where("_id").is(id))); 
```

执行`delete`后，数据库中只剩下一条记录:

```java
{
    "_id" : ObjectId("5702deyu6d8bba0d6f2e45e4"),
    "metadata" : {
        "user" : "alex"
    },
    "filename" : "test.png",
    "aliases" : null,
    "chunkSize" : NumberLong(261120),
    "uploadDate" : ISODate("2015-09-23T17:16:30.781Z"),
    "length" : NumberLong(855),
    "contentType" : "image/png",
    "md5" : "27c915db9aa031f1b27bb05021b695c6"
}
```

带组块:

```java
{
    "_id" : ObjectId("5702deyu6d8bba0d6f2e45e4"),
    "files_id" : ObjectId("5702deyu6d8bba0d6f2e45e4"),
    "n" : 0,
    "data" : 
    { 
        "$binary" : "/9j/4AAQSkZJRgABAQAAAQABAAD/4QAqRXhpZgAASUkqAAgAAAABADEBAgAHAAAAGgAAAAAAAABHb29nbGUAAP/bAIQAAwICAwICAwMDAwQDAwQFCAUFBAQFCgcHBggM
          CgwMCwoLCw0OEhANDhEOCwsQFhARExQVFRUMDxcYFhQYEhQVFAEDBAQGBQUJBgYKDw4MDhQUFA8RDQwMEA0QDA8VDA0NDw0MDw4MDA0ODxAMDQ0MDAwODA8MDQ4NDA0NDAwNDAwQ/8AA
          EQgAHAAcAwERAAIRAQMRAf/EABgAAQEBAQEAAAAAAAAAAAAAAAgGBwUE/8QALBAAAgEDAgQFAwUAAAAAAAAAAQIDBAURBiEABwgSIjFBYXEyUYETFEKhw
          f/EABoBAAIDAQEAAAAAAAAAAAAAAAMEAQIFBgD/xAAiEQACAgEDBAMAAAAAAAAAAAAAAQIRAyIx8BJRYYETIUH/2gAMAwEAAhEDEQA/AHDyq1Bb6GjFPMAszLkZHHCTi1I6O
          cXOFRZ1ZqoX6aqzRClkhb9MGVh2SsNyVI/hjG5389tuGcUaLK1GmFfpn5r3rnXpfV82pGtS3a0XmaGOO3zguKV1SWDwBQDH2uUWTOWMZzuM8bS0VQtJKRb2li9LL3l+4VNQPEfQTOB/WO
          G1K0JtUzwad1eZaYBiqzL4S2N8cZUsa7DqlRGdWvMq5aX6b9Tvb5pIZbggt7VcU3YacSkDbfuLNuu3lkk+98GNfIrLt2gK9K/NWl5Z87Ldebj3R0NTa2tVVKhOI0KoQ5AG4DRqSPk+gHGn
          khUPYNOx92vW9PcrdDW0FUJqOp7po5ETIYMxOdyOAK0qAvcgKPWa0oMTo7SEYDKPp98/5wPoJsx3rZ1wLhojS9iinLD9w9W47iSwVe0Z3wfrPoce2eC4I6rCX9MxrpUpWqudNunUosNLR1EkiyIGDqUKF
          fyZB+AeG80riueQdVfObC/tN1pLdaLfSxMiRQ08aIg2CjtGAB9uEyCSqSWujICUXwghT57A5+ePEoMvUdc5a3XlSsgUhZGjGM/TGAqjz+SfuT7DDmGC6WzzeyOv0+2amOrr3KylzTUwjjDeWGbJJ9/COI
          yvRFFv1iRsVGDaqYGWVsIoBZydsDhQGf/Z", 
        "$type" : "00" 
    }
}
```

### 3.5。`getResources`

`getResources`返回所有具有给定文件名模式的`GridFsResource`。

假设我们在数据库中有以下记录:

```java
[
   {
       "_id" : ObjectId("5602de6e5d8bba0d6f2e45e4"),
       "metadata" : {
           "user" : "alex"
       },
       "filename" : "test.png",
       "aliases" : null,
       "chunkSize" : NumberLong(261120),
       "uploadDate" : ISODate("2015-09-23T17:16:30.781Z"),
       "length" : NumberLong(855),
       "contentType" : "image/png",
       "md5" : "27c915db9aa031f1b27bb05021b695c6"
   },
   {
       "_id" : ObjectId("5505de6e5d8bba0d6f8e4574"),
       "metadata" : {
           "user" : "david"
       },
       "filename" : "test.png",
       "aliases" : null,
       "chunkSize" : NumberLong(261120),
       "uploadDate" : ISODate("2015-09-23T17:16:30.781Z"),
       "length" : NumberLong(855),
       "contentType" : "image/png",
       "md5" : "27c915db9aa031f1b27bb05021b695c6"
    },
    {
       "_id" : ObjectId("5777de6e5d8bba0d6f8e4574"),
       "metadata" : {
           "user" : "eugen"
       },
       "filename" : "baeldung.png",
       "aliases" : null,
       "chunkSize" : NumberLong(261120),
       "uploadDate" : ISODate("2015-09-23T17:16:30.781Z"),
       "length" : NumberLong(855),
       "contentType" : "image/png",
       "md5" : "27c915db9aa031f1b27bb05021b695c6"
    }
```

```java
]
```

现在让我们使用文件模式执行`getResources`:

```java
GridFsResource[] gridFsResource = gridFsTemplate.getResources("test*");
```

这将返回文件名以“test”开头的两条记录(在本例中，它们都被命名为`test.png`)。

## 4。`GridFSFile`核心方法

`GridFSFile` API 也非常简单:

*   `getFilename`–获取文件的文件名
*   `getMetaData`–获取给定文件的元数据
*   `containsField`–确定文档是否包含具有给定名称的字段
*   `get`–按名称从对象中获取一个字段
*   `getId`–获取文件的对象 ID
*   `keySet`–获取对象的字段名称

## 5。结论

在本文中，我们研究了 MongoDB 的`GridFS`特性，以及如何使用 Spring Data MongoDB 与它们进行交互。

所有这些例子的实现和代码片段**可以在** [**我的 github 项目**](https://web.archive.org/web/20220815044110/https://github.com/eugenp/tutorials/tree/master/persistence-modules/spring-boot-persistence-mongodb) 中找到。