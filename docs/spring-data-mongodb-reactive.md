# 使用 MongoDB 的 Spring 数据反应库

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-data-mongodb-reactive>

## 1。简介

在本教程中，我们将了解如何通过 Spring Data Reactive Repositories 和 MongoDB 使用反应式编程来配置和实现数据库操作。

我们将复习`ReactiveCrud` *仓库、* `ReactiveMongoRepository` *、*以及`ReactiveMongoTemplate.`的基本用法

尽管这些实现使用了[反应式编程](/web/20220701012946/https://www.baeldung.com/spring-5-functional-web)，但这并不是本教程的主要焦点。

## 2。环境

为了使用反应式 MongoDB，我们需要将依赖项添加到我们的`pom.xml.`

我们还将添加一个嵌入式 MongoDB 进行测试:

```java
<dependencies>
    // ...
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-mongodb-reactive</artifactId>
    </dependency>
    <dependency>
        <groupId>de.flapdoodle.embed</groupId>
        <artifactId>de.flapdoodle.embed.mongo</artifactId>
        <scope>test</scope>
    </dependency>
</dependencies>
```

## 3。配置

为了激活响应式支持，我们需要使用`@EnableReactiveMongoRepositories`以及一些基础设施设置:

```java
@EnableReactiveMongoRepositories
public class MongoReactiveApplication
  extends AbstractReactiveMongoConfiguration {

    @Bean
    public MongoClient mongoClient() {
        return MongoClients.create();
    }

    @Override
    protected String getDatabaseName() {
        return "reactive";
    }
}
```

请注意，如果我们使用的是独立的 MongoDB 安装，那么上述内容是必要的。但是，在我们的示例中，我们将 Spring Boot 与嵌入式 MongoDB 结合使用，因此上述配置是不必要的。

## 4。创造一个`Document`

对于下面的例子，让我们创建一个`Account`类并用`@Document`对其进行注释，以便在数据库操作中使用它:

```java
@Document
public class Account {

    @Id
    private String id;
    private String owner;
    private Double value;

    // getters and setters
}
```

## 5。使用反应库

我们已经熟悉了[存储库编程模型](/web/20220701012946/https://www.baeldung.com/spring-data-mongodb-tutorial)，已经定义了 CRUD 方法，还支持其他一些常见的东西。

现在有了反应模型，我们得到了相同的方法和规范，除了我们将以反应的方式处理结果和参数。

### 5.1。`ReactiveCrudRepository`

我们可以像阻塞`CrudRepository`一样使用这个存储库:

```java
@Repository
public interface AccountCrudRepository 
  extends ReactiveCrudRepository<Account, String> {

    Flux<Account> findAllByValue(String value);
    Mono<Account> findFirstByOwner(Mono<String> owner);
}
```

我们可以传递不同类型的参数，比如简单的(`String`)、包装的(`Optional`、`Stream`)或反应的(`Mono`、`Flux`)，就像我们在`findFirstByOwner()`方法中看到的那样。

### 5.2。`ReactiveMongoRepository`

还有`ReactiveMongoRepository`接口，它继承了`ReactiveCrudRepository`并增加了一些新的查询方法:

```java
@Repository
public interface AccountReactiveRepository 
  extends ReactiveMongoRepository<Account, String> { }
```

使用`ReactiveMongoRepository`，我们可以通过例子进行查询:

```java
Flux<Account> accountFlux = repository
  .findAll(Example.of(new Account(null, "owner", null)));
```

因此，我们将获得与传递的示例相同的每个`Account`。

随着我们的存储库的创建，他们已经定义了一些方法来执行一些我们不需要实现的数据库操作:

```java
Mono<Account> accountMono 
  = repository.save(new Account(null, "owner", 12.3));
Mono<Account> accountMono2 = repository
  .findById("123456");
```

### 5.3。`RxJava2CrudRepository`

对于`RxJava2CrudRepository,`，我们有与`ReactiveCrudRepository,`相同的行为，但是结果和参数类型来自`RxJava`:

```java
@Repository
public interface AccountRxJavaRepository 
  extends RxJava2CrudRepository<Account, String> {

    Observable<Account> findAllByValue(Double value);
    Single<Account> findFirstByOwner(Single<String> owner);
}
```

### 5.4。测试我们的基本操作

为了测试我们的存储库方法，我们将使用测试订阅者:

```java
@Test
public void givenValue_whenFindAllByValue_thenFindAccount() {
    repository.save(new Account(null, "Bill", 12.3)).block();
    Flux<Account> accountFlux = repository.findAllByValue(12.3);

    StepVerifier
      .create(accountFlux)
      .assertNext(account -> {
          assertEquals("Bill", account.getOwner());
          assertEquals(Double.valueOf(12.3) , account.getValue());
          assertNotNull(account.getId());
      })
      .expectComplete()
      .verify();
}

@Test
public void givenOwner_whenFindFirstByOwner_thenFindAccount() {
    repository.save(new Account(null, "Bill", 12.3)).block();
    Mono<Account> accountMono = repository
      .findFirstByOwner(Mono.just("Bill"));

    StepVerifier
      .create(accountMono)
      .assertNext(account -> {
          assertEquals("Bill", account.getOwner());
          assertEquals(Double.valueOf(12.3) , account.getValue());
          assertNotNull(account.getId());
      })
      .expectComplete()
      .verify();
}

@Test
public void givenAccount_whenSave_thenSaveAccount() {
    Mono<Account> accountMono = repository.save(new Account(null, "Bill", 12.3));

    StepVerifier
      .create(accountMono)
      .assertNext(account -> assertNotNull(account.getId()))
      .expectComplete()
      .verify();
}
```

## 6。`ReactiveMongoTemplate`

除了存储库方法，我们还有 `ReactiveMongoTemplate`。

首先，我们需要将`ReactiveMongoTemplate`注册为一个 bean:

```java
@Configuration
public class ReactiveMongoConfig {

    @Autowired
    MongoClient mongoClient;

    @Bean
    public ReactiveMongoTemplate reactiveMongoTemplate() {
        return new ReactiveMongoTemplate(mongoClient, "test");
    }
}
```

然后，我们可以将这个 bean 注入到我们的服务中来执行数据库操作:

```java
@Service
public class AccountTemplateOperations {

    @Autowired
    ReactiveMongoTemplate template;

    public Mono<Account> findById(String id) {
        return template.findById(id, Account.class);
    }

    public Flux<Account> findAll() {
        return template.findAll(Account.class);
    } 
    public Mono<Account> save(Mono<Account> account) {
        return template.save(account);
    }
}
```

`ReactiveMongoTemplate`也有许多与我们的领域无关的方法，你可以在[文档](https://web.archive.org/web/20220701012946/https://docs.spring.io/spring-data/mongodb/docs/current/api/org/springframework/data/mongodb/core/ReactiveMongoTemplate.html)中查看它们。

## 7。结论

在这篇简短的教程中，我们已经介绍了使用 MongoDB with Spring Data Reactive Repositories framework 进行反应式编程时如何使用存储库和模板。

GitHub 上的[提供了示例的完整源代码。](https://web.archive.org/web/20220701012946/https://github.com/eugenp/tutorials/tree/master/persistence-modules/spring-data-mongodb-reactive)