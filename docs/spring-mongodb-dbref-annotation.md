# MongoDB 中@DBRef 的指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-mongodb-dbref-annotation>

## 1。概述

在本教程中，我们将查看 Spring Data MongoDB 的 [`@DBRef`](/web/20220707143816/https://www.baeldung.com/cascading-with-dbref-and-lifecycle-events-in-spring-data-mongodb#dbref) 注释。我们将使用这个注释连接 MongoDB 文档。此外，我们还将看到 MongoDB 数据库引用的类型，并对它们进行比较。

## 2.MongoDB 手册数据库参考

我们讨论的第一种类型称为手动参考。在 MongoDB 中，每个文档都必须有一个 [`id`字段](/web/20220707143816/https://www.baeldung.com/java-mongodb-last-inserted-id#what-is-the-id-of-a-mongodb-document)。因此，我们可以依靠使用它，并用它连接文档。

**使用手动引用时，我们将被引用文档的`_id`存储在另一个文档中。**

稍后，当我们从第一个集合中查询数据时，我们可以启动第二个查询来获取引用的文档。

## 3。Spring Data MongoDB `@DBRef`注解

`DBRef`与手动参考相似，因为它们也包含参考文件的`_id`。**然而，它们在`$ref`字段中包含引用文件的集合，并且可选地在`$db`字段中包含其数据库。**

与手动引用相比，这种方法的优势在于它可以澄清我们引用的是哪个集合。

## 4。应用程序设置

### 4.1。依赖性

首先，我们需要添加将 MongoDB 与 Spring Boot 一起使用所需的依赖项。

让我们将`[spring-boot-starter-data-mongodb](https://web.archive.org/web/20220707143816/https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter-data-mongodb)`添加到我们的`pom.xml`中:

```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-mongodb</artifactId>
</dependency>
```

### 4.2。配置

现在，我们通过向`application.properties`文件添加以下配置来建立连接:

```java
spring.data.mongodb.host=localhost
spring.data.mongodb.port=27017
spring.data.mongodb.database=person_database
```

让我们运行应用程序来测试我们是否能连接到数据库。我们应该在日志中看到类似这样的消息:

```java
Opened connection [connectionId{localValue:2, serverValue:37}] to localhost:27017
```

这意味着应用程序可以成功连接到 MongoDB。

### 4.3。收藏

**在 MongoDB 中，我们使用** **[集合](https://web.archive.org/web/20220707143816/https://docs.mongodb.com/manual/core/databases-and-collections/#collections)来存储单个文档。它们是关系数据库中表的对等物。**

在这个例子中，我们将使用三种不同的数据类型:`Person`、`Dog`和`Cat`。我们会把人和他们的宠物联系起来。

让我们用一些数据在数据库中创建集合。为此，我们可以使用 [Mongo Express](https://web.archive.org/web/20220707143816/https://github.com/mongo-express/mongo-express) ，但是任何其他工具也可以。

首先，让我们创建一个名为`person_database`的数据库，并在其中创建两个名为`Dog`和`Cat.`的集合，我们将在每个集合中插入一个文档。简单来说，它们都只有一个属性:宠物的名字。

让我们将这个文档插入到`Dog`集合中:

```java
{
    _id: ObjectID("622112d71f9dac417b84227d"), 
    name: "Max"
}
```

那么，我们把这个文档插入 *猫* 收藏:

```java
{
    _id: ObjectID("622112781f9dac417b84227b"),
    name: "Loki"
} 
```

现在，让我们创建一个 *人* 集合，并在其中插入一个文档:

```java
{
    _id: ObjectId(),
    name: "Bob",
    pets: [
        {
          "$ref": "Cat",
          "$id": "622112781f9dac417b84227b",
          "$db": ""
        },    
        {
          "$ref": "Dog",
          "$id": "622112d71f9dac417b84227d",
          "$db": ""
        }
    ]
} 
```

我们以阵列的形式提供宠物。**数组的项目需要遵循一个[特定的格式](https://web.archive.org/web/20220707143816/https://docs.mongodb.com/manual/reference/database-references/#format)以便能够将它们用作`DBRef`。**我们需要在`$ref`属性中提供集合的名称。在这种情况下，是`Cat`和`Dog`。然后，我们包括引用文档的 ID。最后，如果我们想引用不同数据库中的集合，我们可以选择在`$db`属性中包含一个数据库名称。

## 5。使用`@DBRef`标注

我们可以将之前创建的集合映射到 Java 类，类似于我们在处理关系数据库时所做的。

为了简化，我们不会为`Dog`和`Cat`数据类型创建单独的类。相反，我们将使用一个包含 ID 和名称的`Pet`类:

```java
public class Pet {
    private String id;
    private String name;

    @Override 
    public String toString() {
        return "Pet [id=" + id + ", name=" + name + "]";
    }

    // standard getters and setters
} 
```

现在，我们将创建 *人* 类，并通过 *@DBRef:* 将一个关联包含到 *宠物* 类

```java
@Document(collection = "Person")
public class Person {

    @Id
    private String id;

    private String name;

    @DBRef
    private List<Pet> pets;

    @Override 
    public String toString() {
        return "Person [id=" + id + ", name=" + name + ", pets=" + pets + "]";
    }

    // standard getters and setters
}
```

接下来，让我们创建一个简单的[存储库](/web/20220707143816/https://www.baeldung.com/spring-data-mongodb-tutorial#using-mongorepository)来查询数据:

```java
public interface PersonRepository extends MongoRepository<Person, String> {}
```

我们将通过创建一个在启动应用程序时执行 MongoDB 查询的 [ApplicationRunner](/web/20220707143816/https://www.baeldung.com/running-setup-logic-on-startup-in-spring#7-spring-boot-applicationrunner) 来测试一切。让我们覆盖`run()`方法并放置一个日志语句，这样我们就可以看到`Person`集合的内容:

```java
@Override
public void run(ApplicationArguments args) throws Exception {
    logger.info("{}", personRepository.findAll());
} 
```

这将生成一个与此类似的日志输出，因为我们在实体类中覆盖了`toString()`方法:

```java
com.baeldung.mongodb.dbref.DbRefTester : [Person [id=62249c5c7ffe83c50ad12700, name=Bob, pets=[Pet [id=622112781f9dac417b84227b, name=Loki], Pet [id=622112d71f9dac417b84227d, name=Max]]]]
```

这意味着我们成功地阅读并加入了来自不同集合的文档。

### 5.1。参考不同的数据库

**`@DBRef`注释接受两个参数。其中一个是`db`参数，可以用来引用其他数据库中的文档。**

这是可选的，这意味着如果我们不提供这个值，应用程序将在同一个数据库中查找引用的文档。

在我们的例子中，如果`Cat`或`Dog`位于另一个名为`pet_database,`的数据库中，我们需要将注释改为:`@DBRef(db = “pet_database”)`。

### 5.2。惰性加载

**注释接受的另一个参数叫做`lazy`。这是一个`boolean`值，它决定引用的文档是否应该延迟加载。**

默认情况下，它是`false`，这意味着当我们查询主实体时，引用将被急切地加载。如果我们打开此功能，引用的文档将不会被加载，直到它们被首先访问。

## 6。结论

在本文中，我们将 MongoDB 手动参考与 Spring 数据 MongoDB `@DBRef.` 进行了比较，我们创建了三个集合并用这个注释将它们连接起来。我们创建了一个 Spring Boot 应用程序，使用一个`MongoRepository`查询这些集合并显示相关文档。

与往常一样，GitHub 上的[提供了示例的源代码。](https://web.archive.org/web/20220707143816/https://github.com/eugenp/tutorials/tree/master/persistence-modules/spring-boot-persistence-mongodb)