# Spring 数据 MongoDB 事务

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-data-mongodb-transactions>

## 1。概述

从 4.0 版本开始，MongoDB 支持多文档 ACID 事务。并且， **Spring Data Lovelace 现在为这些原生 MongoDB 事务提供支持**。

在本教程中，我们将讨论 Spring Data MongoDB 对同步和反应式事务的支持。

我们还将看看 Spring Data `TransactionTemplate`对非本地事务的支持。

关于这个 Spring 数据模块的介绍，请看一下我们的介绍性文章。

## 2。安装 MongoDB 4.0

首先，我们需要安装最新的 MongoDB 来尝试新的本地事务支持。

要开始，我们必须从 [MongoDB 下载中心](https://web.archive.org/web/20221012162640/https://www.mongodb.com/download-center?initial=true#atlas)下载最新版本。

接下来，我们将使用命令行启动`mongod`服务:

```java
mongod --replSet rs0
```

最后，启动副本集—如果尚未启动:

```java
mongo --eval "rs.initiate()"
```

注意，MongoDB 目前支持副本集上的事务。

## 3。Maven 配置

接下来，我们需要将以下依赖项添加到我们的`pom.xml`中:

```java
<dependency>
    <groupId>org.springframework.data</groupId>
    <artifactId>spring-data-mongodb</artifactId>
    <version>3.0.3.RELEASE</version>
</dependency>
```

该库的最新版本可以在[中央存储库](https://web.archive.org/web/20221012162640/https://search.maven.org/search?q=g:org.springframework.data%20AND%20a:spring-data-mongodb)上找到

## 4。MongoDB 配置

现在，让我们来看看我们的配置:

```java
@Configuration
@EnableMongoRepositories(basePackages = "com.baeldung.repository")
public class MongoConfig extends AbstractMongoClientConfiguration{

    @Bean
    MongoTransactionManager transactionManager(MongoDatabaseFactory dbFactory) {
        return new MongoTransactionManager(dbFactory);
    }

    @Override
    protected String getDatabaseName() {
        return "test";
    }

    @Override
    public MongoClient mongoClient() {
        final ConnectionString connectionString = new ConnectionString("mongodb://localhost:27017/test");
        final MongoClientSettings mongoClientSettings = MongoClientSettings.builder()
            .applyConnectionString(connectionString)
            .build();
        return MongoClients.create(mongoClientSettings);
    }
}
```

注意**我们需要在配置中注册`MongoTransactionManager`** 来启用本地 MongoDB 事务，因为它们在默认情况下是禁用的。

## 5.同步交易

在我们完成配置之后，我们使用原生 MongoDB 事务所需要做的就是用 `**@Transactional**.`对我们的方法进行**注释**

注释方法中的所有内容都将在一个事务中执行:

```java
@Test
@Transactional
public void whenPerformMongoTransaction_thenSuccess() {
    userRepository.save(new User("John", 30));
    userRepository.save(new User("Ringo", 35));
    Query query = new Query().addCriteria(Criteria.where("name").is("John"));
    List<User> users = mongoTemplate.find(query, User.class);

    assertThat(users.size(), is(1));
}
```

注意，我们不能在多文档事务中使用`listCollections`命令，例如:

```java
@Test(expected = MongoTransactionException.class)
@Transactional
public void whenListCollectionDuringMongoTransaction_thenException() {
    if (mongoTemplate.collectionExists(User.class)) {
        mongoTemplate.save(new User("John", 30));
        mongoTemplate.save(new User("Ringo", 35));
    }
}
```

这个例子抛出了一个`MongoTransactionException`，因为我们使用了`collectionExists()`方法。

## 6。`TransactionTemplate`

我们看到了 Spring 数据如何支持新的 MongoDB 原生事务。此外，Spring 数据还提供了非本地选项。

**我们可以使用 Spring 数据`TransactionTemplate`** 执行非本地事务:

```java
@Test
public void givenTransactionTemplate_whenPerformTransaction_thenSuccess() {
    mongoTemplate.setSessionSynchronization(SessionSynchronization.ALWAYS);                                     

    TransactionTemplate transactionTemplate = new TransactionTemplate(mongoTransactionManager);
    transactionTemplate.execute(new TransactionCallbackWithoutResult() {
        @Override
        protected void doInTransactionWithoutResult(TransactionStatus status) {
            mongoTemplate.insert(new User("Kim", 20));
            mongoTemplate.insert(new User("Jack", 45));
        };
    });

    Query query = new Query().addCriteria(Criteria.where("name").is("Jack")); 
    List<User> users = mongoTemplate.find(query, User.class);

    assertThat(users.size(), is(1));
}
```

我们需要将`SessionSynchronization`设置为`ALWAYS`来使用非原生的 Spring 数据事务。

## 7。被动交易

最后，我们将看看对 MongoDB 反应事务的 **Spring 数据支持。**

我们需要向`pom.xml`添加更多的依赖项来使用反应式 MongoDB:

```java
<dependency>
    <groupId>org.mongodb</groupId>
    <artifactId>mongodb-driver-reactivestreams</artifactId>
    <version>4.1.0</version>
</dependency>

<dependency>
    <groupId>org.mongodb</groupId>
    <artifactId>mongodb-driver-sync</artifactId>
    <version>4.0.5</version>
</dependency>

<dependency>
    <groupId>io.projectreactor</groupId>
    <artifactId>reactor-test</artifactId>
    <version>3.2.0.RELEASE</version>
    <scope>test</scope>
</dependency>
```

Maven Central 上提供了[MongoDB-driver-react 流](https://web.archive.org/web/20221012162640/https://search.maven.org/search?q=mongodb-driver-reactivestreams)、 [mongodb-driver-sync 和](https://web.archive.org/web/20221012162640/https://search.maven.org/search?q=mongodb-driver-sync) [reactor-test](https://web.archive.org/web/20221012162640/https://search.maven.org/search?q=a:reactor-test%20AND%20g:io.projectreactor) 依赖项。

当然，我们需要配置我们的反应式 MongoDB:

```java
@Configuration
@EnableReactiveMongoRepositories(basePackages 
  = "com.baeldung.reactive.repository")
public class MongoReactiveConfig 
  extends AbstractReactiveMongoConfiguration {

    @Override
    public MongoClient reactiveMongoClient() {
        return MongoClients.create();
    }

    @Override
    protected String getDatabaseName() {
        return "reactive";
    }
}
```

要在 reactive MongoDB 中使用事务，我们需要使用`ReactiveMongoOperations`中的`inTransaction()`方法:

```java
@Autowired
private ReactiveMongoOperations reactiveOps;

@Test
public void whenPerformTransaction_thenSuccess() {
    User user1 = new User("Jane", 23);
    User user2 = new User("John", 34);
    reactiveOps.inTransaction()
      .execute(action -> action.insert(user1)
      .then(action.insert(user2)));
}
```

更多关于 Spring Data 中的反应式知识库的信息可以在[这里](/web/20221012162640/https://www.baeldung.com/spring-data-mongodb-reactive)找到。

## 8。结论

在这篇文章中，我们学习了如何使用 Spring 数据来使用本地和非本地 MongoDB 事务。

GitHub 上的[提供了示例的完整源代码。](https://web.archive.org/web/20221012162640/https://github.com/eugenp/tutorials/tree/master/persistence-modules/spring-data-mongodb)