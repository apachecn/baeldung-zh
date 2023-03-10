# spring Data MongoDB——索引、注释和转换器

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-data-mongodb-index-annotations-converter>

## 1。概述

在本教程中，我们将探索 Spring Data MongoDB 的一些核心特性——索引、公共注释和转换器。

## 2。索引

### 2.1。`@Indexed`

这个注释**在 MongoDB 中将字段标记为索引**:

```java
@QueryEntity
@Document
public class User {
    @Indexed
    private String name;

    ... 
}
```

既然`name`字段已经被索引——让我们看看 MongoDB shell 中的索引:

```java
db.user.getIndexes();
```

以下是我们得到的结果:

```java
[
    {
        "v" : 1,
        "key" : {
             "_id" : 1
         },
        "name" : "_id_",
        "ns" : "test.user"
    }
]
```

我们可能会感到惊讶，任何地方都没有`name`场的迹象！

这是因为，**从 Spring Data MongoDB 3.0 开始，自动索引创建被默认关闭**。

然而，我们可以通过在我们的`MongoConfig`中显式覆盖`autoIndexCreation()`方法来改变这种行为:

```java
public class MongoConfig extends AbstractMongoClientConfiguration {

    // rest of the config goes here

    @Override
    protected boolean autoIndexCreation() {
        return true;
    }
} 
```

让我们再次检查一下 MongoDB shell 中的索引:

```java
[
    {
        "v" : 1,
        "key" : {
             "_id" : 1
         },
        "name" : "_id_",
        "ns" : "test.user"
    },
    {
         "v" : 1,
         "key" : {
             "name" : 1
          },
          "name" : "name",
          "ns" : "test.user"
     }
]
```

正如我们所看到的，这一次，我们有两个索引——其中一个是`_id`——它是由于`@Id`注释而默认创建的，第二个是我们的`name`字段。

或者，**如果我们使用 Spring Boot，我们可以将`spring.data.mongodb.auto-index-creation`属性设置为`true`。**

### 2.2。以编程方式创建索引

我们还可以通过编程方式创建索引:

```java
mongoOps.indexOps(User.class).
  ensureIndex(new Index().on("name", Direction.ASC)); 
```

我们现在已经为字段`name`创建了一个索引，结果将与上一节相同。

### 2.3。复合指数

MongoDB 支持复合索引，其中单个索引结构保存对多个字段的引用。

让我们看一个使用复合索引的简单例子:

```java
@QueryEntity
@Document
@CompoundIndexes({
    @CompoundIndex(name = "email_age", def = "{'email.id' : 1, 'age': 1}")
})
public class User {
    //
}
```

我们用`email`和`age`字段创建了一个复合索引。现在让我们来看看实际的索引:

```java
{
    "v" : 1,
    "key" : {
        "email.id" : 1,
        "age" : 1
    },
    "name" : "email_age",
    "ns" : "test.user"
} 
```

注意，`DBRef`字段不能用`@Index`标记——该字段只能是复合索引的一部分。

## 3。常见注释

### 3.1。`@Transient`

正如我们所料，这个简单的注释排除了数据库中的持久化字段:

```java
public class User {

    @Transient
    private Integer yearOfBirth;
```

```java
 // standard getter and setter

}
```

让我们用设置字段`yearOfBirth`插入用户:

```java
User user = new User();
user.setName("Alex");
user.setYearOfBirth(1985);
mongoTemplate.insert(user); 
```

现在，如果我们查看数据库的状态，我们会看到文件`yearOfBirth`没有保存:

```java
{
    "_id" : ObjectId("55d8b30f758fd3c9f374499b"),
    "name" : "Alex",
    "age" : null
}
```

因此，如果我们查询并检查:

```java
mongoTemplate.findOne(Query.query(Criteria.where("name").is("Alex")), User.class).getYearOfBirth()
```

结果会是`null`。

### 3.2。`@Field`

`@Field`表示用于 JSON 文档中字段的键:

```java
@Field("email")
private EmailAddress emailAddress; 
```

现在使用键`email:`将`emailAddress`保存在数据库中

```java
User user = new User();
user.setName("Brendan");
EmailAddress emailAddress = new EmailAddress();
emailAddress.setValue("[[email protected]](/web/20220627173404/https://www.baeldung.com/cdn-cgi/l/email-protection)");
user.setEmailAddress(emailAddress);
mongoTemplate.insert(user); 
```

以及数据库的状态:

```java
{
    "_id" : ObjectId("55d076d80bad441ed114419d"),
    "name" : "Brendan",
    "age" : null,
    "email" : {
        "value" : "[[email protected]](/web/20220627173404/https://www.baeldung.com/cdn-cgi/l/email-protection)"
    }
}
```

### 3.3。`@PersistenceConstructor` 和 `@Value`

标记一个构造函数，即使是一个包保护的构造函数，作为持久逻辑使用的主构造函数。构造函数参数通过名称映射到检索到的`DBObject`中的键值。

让我们来看看我们的`User`类的这个构造函数:

```java
@PersistenceConstructor
public User(String name, @Value("#root.age ?: 0") Integer age, EmailAddress emailAddress) {
    this.name =  name;
    this.age = age;
    this.emailAddress =  emailAddress;
} 
```

注意这里使用了标准的 Spring `@Value`注释。在这个注释的帮助下，我们可以使用 Spring 表达式来转换从数据库中检索到的键值，然后再用它来构造域对象。这是一个非常强大和非常有用的功能。

在我们的例子中，如果没有设置`age`，默认情况下它将被设置为`0`。

现在让我们看看它是如何工作的:

```java
User user = new User();
user.setName("Alex");
mongoTemplate.insert(user);
```

我们的数据库将会是:

```java
{
    "_id" : ObjectId("55d074ca0bad45f744a71318"),
    "name" : "Alex",
    "age" : null
}
```

所以`age`字段是`null`，但是当我们查询文档并检索`age`时:

```java
mongoTemplate.findOne(Query.query(Criteria.where("name").is("Alex")), User.class).getAge();
```

结果将是 0。

## 4。转换器

现在让我们看看 Spring Data MongoDB 中另一个非常有用的特性——转换器，特别是在`MongoConverter`中。

这用于在存储和查询这些对象时处理所有 Java 类型到`DBObjects`的映射。

我们有两个选择——我们可以在早期版本中使用`MappingMongoConverter –` 或`SimpleMongoConverter`(这在 Spring Data MongoDB M3 版中已被弃用，其功能已被移至`MappingMongoConverter` ) `.` 

或者我们可以编写自己的自定义转换器。为此，我们需要实现`Converter` 接口，并在`MongoConfig.`中注册实现

让我们看看**一个简单的例子**。正如我们在这里的一些 JSON 输出中看到的，保存在数据库中的所有对象都有自动保存的字段`_class`。然而，如果我们想在持久化期间跳过那个特定的字段，我们可以使用一个`MappingMongoConverter`来实现。

首先，这是自定义转换器的实现:

```java
@Component
public class UserWriterConverter implements Converter<User, DBObject> {
    @Override
    public DBObject convert(User user) {
        DBObject dbObject = new BasicDBObject();
        dbObject.put("name", user.getName());
        dbObject.put("age", user.getAge());
        if (user.getEmailAddress() != null) {
            DBObject emailDbObject = new BasicDBObject();
            emailDbObject.put("value", user.getEmailAddress().getValue());
            dbObject.put("email", emailDbObject);
        }
        dbObject.removeField("_class");
        return dbObject;
    }
}
```

请注意，我们如何通过在这里直接移除字段来轻松实现不持久化`_class`的目标。

现在我们需要注册自定义转换器:

```java
private List<Converter<?,?>> converters = new ArrayList<Converter<?,?>>();

@Override
public MongoCustomConversions customConversions() {
    converters.add(new UserWriterConverter());
    return new MongoCustomConversions(converters);
}
```

当然，如果我们需要，我们也可以用 XML 配置获得同样的结果:

```java
<bean id="mongoTemplate" 
  class="org.springframework.data.mongodb.core.MongoTemplate">
    <constructor-arg name="mongo" ref="mongo"/>
    <constructor-arg ref="mongoConverter" />
    <constructor-arg name="databaseName" value="test"/>
</bean>

<mongo:mapping-converter id="mongoConverter" base-package="org.baeldung.converter">
    <mongo:custom-converters base-package="com.baeldung.converter" />
</mongo:mapping-converter>
```

现在，当我们保存新用户时:

```java
User user = new User();
user.setName("Chris");
mongoOps.insert(user); 
```

数据库中的结果文档不再包含类信息:

```java
{
    "_id" : ObjectId("55cf09790bad4394db84b853"),
    "name" : "Chris",
    "age" : null
}
```

## 5。结论

在本教程中，我们介绍了使用 Spring Data MongoDB 的一些核心概念——索引、公共注释和转换器。

所有这些例子的实现和代码片段**可以在 GitHub**T5[**上找到。**](https://web.archive.org/web/20220627173404/https://github.com/eugenp/tutorials/tree/master/persistence-modules/spring-data-mongodb)