# Spring Data MongoDB 简介

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-data-mongodb-tutorial>

## 1。概述

本文将是对 Spring Data MongoDB 的快速而实用的介绍。

我们将使用`MongoTemplate`和`MongoRepository`来复习基础知识，并用实际例子来说明每个操作。

## 延伸阅读:

## [MongoDB 中的地理空间支持](/web/20220628093000/https://www.baeldung.com/mongodb-geospatial-support)

Have a look at how to store, index and search geospatial data with MongoDB[Read more](/web/20220628093000/https://www.baeldung.com/mongodb-geospatial-support) →

## [嵌入式 MongoDB 的 Spring Boot 集成测试](/web/20220628093000/https://www.baeldung.com/spring-boot-embedded-mongodb)

Learn how to use Flapdoodle's embedded MongoDB solution together with Spring Boot to run MongoDB integration tests smoothly.[Read more](/web/20220628093000/https://www.baeldung.com/spring-boot-embedded-mongodb) →

## 2。`MongoTemplate`和 `MongoRepository`

**`MongoTemplate`**遵循 Spring 中的标准模板模式，为底层持久性引擎提供一个现成的基本 API。

存储库遵循了 Spring 以数据为中心的方法，并基于所有 Spring 数据项目中众所周知的访问模式，提供了更加灵活和复杂的 API 操作。

对于这两者，我们需要从定义依赖关系开始——例如，在`pom.xml`中，用 Maven:

```java
<dependency>
    <groupId>org.springframework.data</groupId>
    <artifactId>spring-data-mongodb</artifactId>
    <version>3.0.3.RELEASE</version>
</dependency>
```

要检查是否有新版本的库已经发布，[在这里跟踪发布版本](https://web.archive.org/web/20220628093000/https://search.maven.org/search?q=g:org.springframework.data%20AND%20a:spring-data-mongodb)。

## 3。配置为`MongoTemplate`

### 3.1。XML 配置

让我们从 Mongo 模板的简单 XML 配置开始:

```java
<mongo:mongo-client id="mongoClient" host="localhost" />
<mongo:db-factory id="mongoDbFactory" dbname="test" mongo-client-ref="mongoClient" />
```

我们首先需要定义负责创建 Mongo 实例的工厂 bean。

接下来，我们需要实际定义(和配置)模板 bean:

```java
<bean id="mongoTemplate" class="org.springframework.data.mongodb.core.MongoTemplate"> 
    <constructor-arg ref="mongoDbFactory"/> 
</bean>
```

最后，我们需要定义一个后处理器来翻译任何在`@Repository`注释类中抛出的`MongoExceptions`:

```java
<bean class=
  "org.springframework.dao.annotation.PersistenceExceptionTranslationPostProcessor"/>
```

### 3.2。Java 配置

现在让我们通过扩展 MongoDB 配置`AbstractMongoConfiguration`的基类，使用 Java config 创建一个类似的配置:

```java
@Configuration
public class MongoConfig extends AbstractMongoClientConfiguration {

    @Override
    protected String getDatabaseName() {
        return "test";
    }

    @Override
    public MongoClient mongoClient() {
        ConnectionString connectionString = new ConnectionString("mongodb://localhost:27017/test");
        MongoClientSettings mongoClientSettings = MongoClientSettings.builder()
            .applyConnectionString(connectionString)
            .build();

        return MongoClients.create(mongoClientSettings);
    }

    @Override
    public Collection getMappingBasePackages() {
        return Collections.singleton("com.baeldung");
    }
}
```

注意，我们不需要在前面的配置中定义`MongoTemplate` bean，因为它已经在`AbstractMongoClientConfiguration`中定义了。

我们也可以从头开始使用我们的配置，而无需扩展`AbstractMongoClientConfiguration`:

```java
@Configuration
public class SimpleMongoConfig {

    @Bean
    public MongoClient mongo() {
        ConnectionString connectionString = new ConnectionString("mongodb://localhost:27017/test");
        MongoClientSettings mongoClientSettings = MongoClientSettings.builder()
          .applyConnectionString(connectionString)
          .build();

        return MongoClients.create(mongoClientSettings);
    }

    @Bean
    public MongoTemplate mongoTemplate() throws Exception {
        return new MongoTemplate(mongo(), "test");
    }
}
```

## 4。配置为`MongoRepository`

### 4.1。XML 配置

为了利用定制库(扩展`MongoRepository`)，我们需要继续 3.1 节中的配置。并设置存储库:

```java
<mongo:repositories 
  base-package="com.baeldung.repository" mongo-template-ref="mongoTemplate"/> 
```

### 4.2。Java 配置

类似地，我们将在 3.2 节中已经创建的配置的基础上进行构建。并添加新的注释:

```java
@EnableMongoRepositories(basePackages = "com.baeldung.repository") 
```

### 4.3。创建存储库

配置完成后，我们需要创建一个存储库——扩展现有的`MongoRepository`接口:

```java
public interface UserRepository extends MongoRepository<User, String> {
    // 
}
```

现在我们可以自动连接这个`UserRepository`并使用来自`MongoRepository`的操作或添加自定义操作。

## 5。使用`MongoTemplate`

### 5.1。 `Insert`

让我们从插入操作和一个空数据库开始:

```java
{
}
```

现在，如果我们插入一个新用户:

```java
User user = new User();
user.setName("Jon");
mongoTemplate.insert(user, "user");
```

数据库将如下所示:

```java
{
    "_id" : ObjectId("55b4fda5830b550a8c2ca25a"),
    "_class" : "com.baeldung.model.User",
    "name" : "Jon"
}
```

### 5.2。`Save – Insert`

`save`操作具有保存或更新语义:如果 id 存在，它执行更新，如果不存在，它执行插入。

让我们看看第一个语义——插入。

这里是数据库的初始状态`:`

```java
{
}
```

当我们现在`save`一个新用户:

```java
User user = new User();
user.setName("Albert"); 
mongoTemplate.save(user, "user");
```

实体将被插入到数据库中:

```java
{
    "_id" : ObjectId("55b52bb7830b8c9b544b6ad5"),
    "_class" : "com.baeldung.model.User",
    "name" : "Albert"
}
```

接下来，我们将看看具有更新语义的相同操作— `save`。

### 5.3。`Save – Update`

现在让我们看看带有更新语义的`save`,对现有实体进行操作:

```java
{
    "_id" : ObjectId("55b52bb7830b8c9b544b6ad5"),
    "_class" : "com.baeldung.model.User",
    "name" : "Jack"
}
```

当我们`save`现有用户时，我们将更新它:

```java
user = mongoTemplate.findOne(
  Query.query(Criteria.where("name").is("Jack")), User.class);
user.setName("Jim");
mongoTemplate.save(user, "user");
```

数据库将如下所示:

```java
{
    "_id" : ObjectId("55b52bb7830b8c9b544b6ad5"),
    "_class" : "com.baeldung.model.User",
    "name" : "Jim"
}
```

我们可以看到，在这个特定的例子中，`save`使用了`update`的语义，因为我们使用了一个具有给定的`_id`的对象。

### 5.4。`UpdateFirst`

`updateFirst`更新匹配查询的第一个文档。

让我们从数据库的初始状态开始:

```java
[
    {
        "_id" : ObjectId("55b5ffa5511fee0e45ed614b"),
        "_class" : "com.baeldung.model.User",
        "name" : "Alex"
    },
    {
        "_id" : ObjectId("55b5ffa5511fee0e45ed614c"),
        "_class" : "com.baeldung.model.User",
        "name" : "Alex"
    }
]
```

当我们现在运行`updateFirst`:

```java
Query query = new Query();
query.addCriteria(Criteria.where("name").is("Alex"));
Update update = new Update();
update.set("name", "James");
mongoTemplate.updateFirst(query, update, User.class);
```

只有第一个条目会被更新:

```java
[
    {
        "_id" : ObjectId("55b5ffa5511fee0e45ed614b"),
        "_class" : "com.baeldung.model.User",
        "name" : "James"
    },
    {
        "_id" : ObjectId("55b5ffa5511fee0e45ed614c"),
        "_class" : "com.baeldung.model.User",
        "name" : "Alex"
    }
]
```

### 5.5。`UpdateMulti`

`UpdateMulti` **更新所有匹配给定查询的文档。**

首先，这里是数据库在执行`updateMulti`之前的状态:

```java
[
    {
        "_id" : ObjectId("55b5ffa5511fee0e45ed614b"),
        "_class" : "com.baeldung.model.User",
        "name" : "Eugen"
    },
    {
        "_id" : ObjectId("55b5ffa5511fee0e45ed614c"),
        "_class" : "com.baeldung.model.User",
        "name" : "Eugen"
    }
] 
```

现在让我们运行`updateMulti`操作:

```java
Query query = new Query();
query.addCriteria(Criteria.where("name").is("Eugen"));
Update update = new Update();
update.set("name", "Victor");
mongoTemplate.updateMulti(query, update, User.class);
```

数据库中的两个现有对象都将被更新:

```java
[
    {
        "_id" : ObjectId("55b5ffa5511fee0e45ed614b"),
        "_class" : "com.baeldung.model.User",
        "name" : "Victor"
    },
    {
        "_id" : ObjectId("55b5ffa5511fee0e45ed614c"),
        "_class" : "com.baeldung.model.User",
        "name" : "Victor"
    }
]
```

### 5.6。`FindAndModify`

这个操作类似于`updateMulti`，但是它**返回修改前的对象。**

首先，这是调用`findAndModify`之前数据库的状态:

```java
{
    "_id" : ObjectId("55b5ffa5511fee0e45ed614b"),
    "_class" : "com.baeldung.model.User",
    "name" : "Markus"
} 
```

让我们来看看实际的操作代码:

```java
Query query = new Query();
query.addCriteria(Criteria.where("name").is("Markus"));
Update update = new Update();
update.set("name", "Nick");
User user = mongoTemplate.findAndModify(query, update, User.class);
```

返回的`user object`与数据库中的初始状态具有相同的值。

但是，这是数据库中的新状态:

```java
{
    "_id" : ObjectId("55b5ffa5511fee0e45ed614b"),
    "_class" : "com.baeldung.model.User",
    "name" : "Nick"
}
```

### 5.7。`Upsert`

`upsert`处理**查找和修改 else 创建语义**:如果文档匹配，更新它，或者通过组合查询和更新对象创建一个新文档。

让我们从数据库的初始状态开始:

```java
{
    "_id" : ObjectId("55b5ffa5511fee0e45ed614b"),
    "_class" : "com.baeldung.model.User",
    "name" : "Markus"
}
```

现在让我们运行`upsert`:

```java
Query query = new Query();
query.addCriteria(Criteria.where("name").is("Markus"));
Update update = new Update();
update.set("name", "Nick");
mongoTemplate.upsert(query, update, User.class);
```

以下是操作后数据库的状态:

```java
{
    "_id" : ObjectId("55b5ffa5511fee0e45ed614b"),
    "_class" : "com.baeldung.model.User",
    "name" : "Nick"
}
```

### 5.8。`Remove`

我们将在调用`remove`之前查看数据库的状态:

```java
{
    "_id" : ObjectId("55b5ffa5511fee0e45ed614b"),
    "_class" : "com.baeldung.model.User",
    "name" : "Benn"
}
```

现在让我们运行`remove`:

```java
mongoTemplate.remove(user, "user");
```

结果将如预期的那样:

```java
{
}
```

## 6。使用`MongoRepository`

### 6.1。`Insert`

首先，我们将在运行`insert`之前看到数据库的状态:

```java
{
}
```

现在我们将插入一个新用户:

```java
User user = new User();
user.setName("Jon");
userRepository.insert(user); 
```

这是数据库的最终状态:

```java
{
    "_id" : ObjectId("55b4fda5830b550a8c2ca25a"),
    "_class" : "com.baeldung.model.User",
    "name" : "Jon"
}
```

请注意该操作的工作方式与`MongoTemplate` API 中的`insert`相同。

### 6.2。`Save` – `Insert`

类似地，`save`与 `MongoTemplate` API 中的`save`操作工作相同。

让我们从查看**操作的插入语义**开始。

下面是数据库的初始状态:

```java
{
}
```

现在我们执行`save`操作:

```java
User user = new User();
user.setName("Aaron");
userRepository.save(user);
```

这导致用户被添加到数据库中:

```java
{
    "_id" : ObjectId("55b52bb7830b8c9b544b6ad5"),
    "_class" : "com.baeldung.model.User",
    "name" : "Aaron"
}
```

再次注意`save`如何处理`insert`语义，因为我们插入了一个新对象。

### 6.3。`Save` – `Update`

现在让我们看看相同的操作，但是使用了**更新语义。**

首先，这里是运行新的`save`之前数据库的状态:

```java
{
    "_id" : ObjectId("55b52bb7830b8c9b544b6ad5"),
    "_class" : "com.baeldung.model.User",
    "name" : "Jack"81*6
}
```

现在我们执行操作:

```java
user = mongoTemplate.findOne(
  Query.query(Criteria.where("name").is("Jack")), User.class);
user.setName("Jim");
userRepository.save(user);
```

最后，这里是数据库的状态:

```java
{
    "_id" : ObjectId("55b52bb7830b8c9b544b6ad5"),
    "_class" : "com.baeldung.model.User",
    "name" : "Jim"
}
```

再次注意`save`如何使用`update`语义，因为我们正在使用一个现有的对象。

### 6.4。`Delete`

下面是调用`delete`之前数据库的状态:

```java
{
    "_id" : ObjectId("55b5ffa5511fee0e45ed614b"),
    "_class" : "com.baeldung.model.User",
    "name" : "Benn"
}
```

让我们运行`delete`:

```java
userRepository.delete(user); 
```

这是我们的结果:

```java
{
}
```

### 6.5。`FindOne`

接下来，这是调用`findOne`时数据库的状态:

```java
{
    "_id" : ObjectId("55b5ffa5511fee0e45ed614b"),
    "_class" : "com.baeldung.model.User",
    "name" : "Chris"
}
```

现在让我们执行`findOne`:

```java
userRepository.findOne(user.getId()) 
```

结果将返回现有数据:

```java
{
    "_id" : ObjectId("55b5ffa5511fee0e45ed614b"),
    "_class" : "com.baeldung.model.User",
    "name" : "Chris"
}
```

### 6.6。`Exists`

调用`exists`前数据库的状态:

```java
{
    "_id" : ObjectId("55b5ffa5511fee0e45ed614b"),
    "_class" : "com.baeldung.model.User",
    "name" : "Harris"
}
```

现在让我们运行`exists`，它当然会返回`true`:

```java
boolean isExists = userRepository.exists(user.getId());
```

### 6.7。`FindAll`With `Sort`

调用`findAll`前数据库的状态:

```java
[
    {
        "_id" : ObjectId("55b5ffa5511fee0e45ed614b"),
        "_class" : "com.baeldung.model.User",
        "name" : "Brendan"
    },
    {
       "_id" : ObjectId("67b5ffa5511fee0e45ed614b"),
       "_class" : "com.baeldung.model.User",
       "name" : "Adam"
    }
]
```

现在让我们用 `Sort`运行`findAll` :

```java
List<User> users = userRepository.findAll(Sort.by(Sort.Direction.ASC, "name"));
```

结果将**按姓名升序排序**:

```java
[
    {
        "_id" : ObjectId("67b5ffa5511fee0e45ed614b"),
        "_class" : "com.baeldung.model.User",
        "name" : "Adam"
    },
    {
        "_id" : ObjectId("55b5ffa5511fee0e45ed614b"),
        "_class" : "com.baeldung.model.User",
        "name" : "Brendan"
    }
]
```

### 6.8。`FindAll`With `Pageable`

调用`findAll`前数据库的状态:

```java
[
    {
        "_id" : ObjectId("55b5ffa5511fee0e45ed614b"),
        "_class" : "com.baeldung.model.User",
        "name" : "Brendan"
    },
    {
        "_id" : ObjectId("67b5ffa5511fee0e45ed614b"),
        "_class" : "com.baeldung.model.User",
        "name" : "Adam"
    }
]
```

现在让我们用分页请求来执行`findAll` :

```java
Pageable pageableRequest = PageRequest.of(0, 1);
Page<User> page = userRepository.findAll(pageableRequest);
List<User> users = pages.getContent();
```

产生的`users`列表将只有一个用户:

```java
{
    "_id" : ObjectId("55b5ffa5511fee0e45ed614b"),
    "_class" : "com.baeldung.model.User",
    "name" : "Brendan"
}
```

## 7。注释

最后，让我们看看 Spring 数据用来驱动这些 API 操作的简单注释。

字段级`@Id`标注可以修饰任何类型，包括`long`和`string`:

```java
@Id
private String id;
```

如果`@Id`字段的值不为空，它将按原样存储在数据库中；否则，转换器将假设我们想要在数据库中存储一个`ObjectId`(或者是 `ObjectId`、 `String` 或者是 `BigInteger` 工作)。

我们接下来来看看`@Document`:

```java
@Document
public class User {
    //
}
```

这个注释简单地将一个类标记为需要保存到数据库中的域对象，并允许我们选择要使用的集合的名称。

## 8。结论

本文快速而全面地介绍了如何通过`MongoTemplate` API 和`MongoRepository`使用 MongoDB 和 Spring 数据。

所有这些例子的实现和代码片段**都可以在 GitHub**T5[**上找到。**](https://web.archive.org/web/20220628093000/https://github.com/eugenp/tutorials/tree/master/persistence-modules/spring-data-mongodb)