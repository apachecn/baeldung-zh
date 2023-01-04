# 带有 Couchbase 的 Spring 数据反应库

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-data-reactive-couchbase>

## 1.概观

在本教程中，我们将学习如何使用 Spring 数据库在 [Couchbase 上以一种反应式的方式配置和实现数据库操作。](/web/20220626081955/https://www.baeldung.com/spring-data-couchbase)

我们将讲述`ReactiveCrudRepository `和`ReactiveSortingRepository`的基本用法。此外，我们将使用`AbstractReactiveCouchbaseConfiguration`配置我们的测试应用程序。

## 2.Maven 依赖性

首先，让我们添加必要的依赖项:

```java
<dependency>
    <groupId>io.projectreactor</groupId>
    <artifactId>reactor-core</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-couchbase-reactive</artifactId>
</dependency>
```

spring-boot-starter-data-couchbase-reactive 依赖关系包含了我们使用 reactive API 在 couch base 上操作所需的一切。

我们还将包括使用 Project Reactor API 的 [reactor-core](https://web.archive.org/web/20220626081955/https://search.maven.org/search?q=g:io.projectreactor%20AND%20a:reactor-core) 依赖项。

## 3.配置

接下来，让我们定义 Couchbase 和我们的应用程序之间的连接设置。

让我们首先创建一个保存我们属性的类:

```java
@Configuration
public class CouchbaseProperties {

    private List<String> bootstrapHosts;
    private String bucketName;
    private String bucketPassword;
    private int port;

    public CouchbaseProperties(
      @Value("${spring.couchbase.bootstrap-hosts}") List<String> bootstrapHosts, 
      @Value("${spring.couchbase.bucket.name}") String bucketName, 
      @Value("${spring.couchbase.bucket.password}") String bucketPassword, 
      @Value("${spring.couchbase.port}") int port) {
        this.bootstrapHosts = Collections.unmodifiableList(bootstrapHosts);
        this.bucketName = bucketName;
        this.bucketPassword = bucketPassword;
        this.port = port;
    }

    // getters
}
```

为了使用响应式支持，我们应该创建扩展`AbstractReactiveCouchbaseConfiguration`的配置类:

```java
@Configuration
@EnableReactiveCouchbaseRepositories("com.baeldung.couchbase.domain.repository")
public class ReactiveCouchbaseConfiguration extends AbstractReactiveCouchbaseConfiguration {

    private CouchbaseProperties couchbaseProperties;

    public ReactiveCouchbaseConfiguration(CouchbaseProperties couchbaseProperties) {
        this.couchbaseProperties = couchbaseProperties;
    }

    @Override
    protected List<String> getBootstrapHosts() {
        return couchbaseProperties.getBootstrapHosts();
    }

    @Override
    protected String getBucketName() {
        return couchbaseProperties.getBucketName();
    }

    @Override
    protected String getBucketPassword() {
        return couchbaseProperties.getBucketPassword();
    }

    @Override
    public CouchbaseEnvironment couchbaseEnvironment() {
        return DefaultCouchbaseEnvironment
          .builder()
          .bootstrapHttpDirectPort(couchbaseProperties.getPort())
          .build();
    }
}
```

此外，我们已经使用`@EnableReactiveCouchbaseRepositories`来启用我们的反应库，它将在指定的包下。

此外，为了传递 Couchbase 连接端口，我们已经覆盖了`couchbaseEnvironment()`。

## 4.仓库

在本节中，我们将学习如何创建和使用反应式存储库。**默认情况下，“所有”[视图](https://web.archive.org/web/20220626081955/https://docs.couchbase.com/server/6.0/learn/views/views-intro.html)支持大多数 CRUD 操作。定制的存储库方法由 [N1QL](/web/20220626081955/https://www.baeldung.com/n1ql-couchbase) 提供支持。**如果集群不支持 N1QL，初始化时会抛出`UnsupportedCouchbaseFeatureException`。

首先，让我们创建我们的存储库将使用的 POJO 类:

```java
@Document
public class Person {
    @Id private UUID id;
    private String firstName;

   //getters and setters
}
```

### 4.1.基于视图的知识库

现在，我们将为`Person`创建一个存储库:

```java
@Repository
@ViewIndexed(designDoc = ViewPersonRepository.DESIGN_DOCUMENT)
public interface ViewPersonRepository extends ReactiveCrudRepository<Person, UUID> {

    String DESIGN_DOCUMENT = "person";
}
```

存储库扩展了`ReactiveCrudRepository`接口，以便使用 Reactor API 与 Couchbase 交互。

此外，我们可以添加一个定制方法，并使用`@View`注释使其基于视图:

```java
@View(designDocument = ViewPersonRepository.DESIGN_DOCUMENT)
Flux<Person> findByFirstName(String firstName);
```

默认情况下，查询将查找名为`byFirstName`的视图。如果我们想提供一个定制的视图名称，我们必须使用`viewName`参数。

最后，让我们在测试订阅者的帮助下创建一个简单的 CRUD 测试:

```java
@Test
public void shouldSavePerson_findById_thenDeleteIt() {
    final UUID id = UUID.randomUUID();
    final Person person = new Person(id, "John");
    personRepository
      .save(person)
      .subscribe();

    final Mono<Person> byId = personRepository.findById(id);

    StepVerifier
      .create(byId)
      .expectNextMatches(result -> result
        .getId()
        .equals(id))
      .expectComplete()
      .verify();

    personRepository
      .delete(person)
      .subscribe();
}
```

### 4.2.N1QL/基于视图的存储库

现在，我们将为使用 N1QL 查询的`Person`创建反应式存储库:

```java
@Repository
@N1qlPrimaryIndexed
public interface N1QLPersonRepository extends ReactiveCrudRepository<Person, UUID> {
    Flux<Person> findAllByFirstName(String firstName);
}
```

为了使用 Reactor API，存储库扩展了`ReactiveCrudRepository`。此外，我们添加了一个定制的`findAllByFirstName`方法，它创建 N1QL 支持的查询。

之后，让我们添加对`findAllByFirstName`方法的测试:

```java
@Test
public void shouldFindAll_byLastName() {
    final String firstName = "John";
    final Person matchingPerson = new Person(UUID.randomUUID(), firstName);
    final Person nonMatchingPerson = new Person(UUID.randomUUID(), "NotJohn");
    personRepository
      .save(matchingPerson)
      .subscribe();
    personRepository
      .save(nonMatchingPerson)
      .subscribe();

    final Flux<Person> allByFirstName = personRepository.findAllByFirstName(firstName);

    StepVerifier
      .create(allByFirstName)
      .expectNext(matchingPerson)
      .verifyComplete();
}
```

此外，我们将创建一个存储库，允许我们使用排序抽象来检索人员:

```java
@Repository
public interface N1QLSortingPersonRepository extends ReactiveSortingRepository<Person, UUID> {
    Flux<Person> findAllByFirstName(String firstName, Sort sort);
}
```

最后，让我们编写一个测试来检查数据是否真正排序了:

```java
@Test
public void shouldFindAll_sortedByFirstName() {
    final Person firstPerson = new Person(UUID.randomUUID(), "John");
    final Person secondPerson = new Person(UUID.randomUUID(), "Mikki");
    personRepository
      .save(firstPerson)
      .subscribe();
    personRepository
      .save(secondPerson)
      .subscribe();

    final Flux<Person> allByFirstName = personRepository
      .findAll(Sort.by(Sort.Direction.DESC, "firstName"));

    StepVerifier
      .create(allByFirstName)
      .expectNextMatches(person -> person
        .getFirstName()
        .equals(secondPerson.getFirstName()))
      .expectNextMatches(person -> person
        .getFirstName()
        .equals(firstPerson.getFirstName()))
      .verifyComplete();
}
```

## 5.结论

在本文中，我们学习了如何通过 Couchbase 和 Spring Data Reactive framework 使用反应式编程来使用存储库。

和往常一样，这些例子的代码可以在 Github 的[上找到。](https://web.archive.org/web/20220626081955/https://github.com/eugenp/tutorials/tree/master/spring-5-reactive-modules/spring-5-data-reactive)