# 用 Spring Data Cassandra 记录查询

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-data-cassandra-logging-queries>

## 1.概观

Apache Cassandra 是一个可伸缩的分布式 NoSQL 数据库。Cassandra 在节点之间传输数据，提供无单点故障的连续可用性。事实上，Cassandra 能够以出色的性能处理大量数据

当开发使用数据库的应用程序时，能够记录和调试执行的查询是非常重要的。在本教程中，我们将学习如何在 Spring Boot 中使用 Apache Cassandra 来记录查询和语句。

在我们的例子中，我们将利用 Spring 数据仓库抽象和 [`Testcontainers`](/web/20221208143830/https://www.baeldung.com/spring-boot-testcontainers-integration-test) 库。我们将看到如何通过 Spring 配置来配置 Cassandra 查询日志。此外，我们将探索数据税务请求记录器。我们可以配置这个内置组件来实现更高级的日志记录。

## 2.设置测试环境

为了演示查询日志，我们需要设置一个测试环境。首先，我们将使用 Apache Cassandra 的 [Spring 数据来设置测试数据。接下来，我们将利用](/web/20221208143830/https://www.baeldung.com/spring-data-cassandra-tutorial) [`Testcontainers`](/web/20221208143830/https://www.baeldung.com/spring-boot-testcontainers-integration-test) 库运行 Cassandra 数据库容器进行集成测试。

### 2.1.卡珊德拉知识库

Spring Data 使我们能够创建基于公共 Spring 接口的 Cassandra 存储库。首先，让我们从定义一个简单的 DAO 类开始:

```java
@Table
public class Person {

    @PrimaryKey
    private UUID id;
    private String firstName;
    private String lastName;

    public Person(UUID id, String firstName, String lastName) {
        this.id = id;
        this.firstName = firstName;
        this.lastName = lastName;
    }

    // getters, setters, equals and hash code
}
```

然后，我们将通过扩展`CassandraRepository`接口为我们的 DAO 定义一个 Spring 数据存储库:

```java
@Repository
public interface PersonRepository extends CassandraRepository<Person, UUID> {}
```

最后，我们将在我们的`application.properties`文件中添加两个属性:

```java
spring.data.cassandra.schema-action=create_if_not_exists
spring.data.cassandra.local-datacenter=datacenter1
```

因此，Spring Data 会自动为我们创建带注释的表。

我们应该注意，不建议生产系统使用`create_if_not_exists`选项。

或者，可以通过从标准根类路径加载一个 [`schema.sql`脚本](/web/20221208143830/https://www.baeldung.com/spring-boot-data-sql-and-schema-sql)来创建表。

### 2.2.卡珊德拉集装箱

下一步，让我们在特定端口上配置和公开一个 Cassandra 容器:

```java
@Container
public static final CassandraContainer cassandra = 
  (CassandraContainer) new CassandraContainer("cassandra:3.11.2").withExposedPorts(9042);
```

在使用容器进行集成测试之前，我们需要[覆盖 Spring 数据所需的测试属性](/web/20221208143830/https://www.baeldung.com/spring-tests-override-properties)来建立与它的连接:

```java
TestPropertyValues.of(
  "spring.data.cassandra.keyspace-name=" + KEYSPACE_NAME,
  "spring.data.cassandra.contact-points=" + cassandra.getContainerIpAddress(),
  "spring.data.cassandra.port=" + cassandra.getMappedPort(9042)
).applyTo(configurableApplicationContext.getEnvironment());

createKeyspace(cassandra.getCluster());
```

最后，在创建任何对象/表之前，我们需要**创建一个 Cassandra keyspace** 。键空间类似于 RDBMS 中的数据库。

### 2.3.集成测试

现在，我们已经准备好开始编写我们的集成测试了。

我们对**记录选择、插入和删除查询**感兴趣。因此，我们将编写几个测试来触发这些不同类型的查询。

首先，我们将编写一个保存和更新个人记录的测试。我们期望这个测试执行两个插入和一个选择数据库查询:

```java
@Test
void givenExistingPersonRecord_whenUpdatingIt_thenRecordIsUpdated() {
    UUID personId = UUIDs.timeBased();
    Person existingPerson = new Person(personId, "Luka", "Modric");
    personRepository.save(existingPerson);
    existingPerson.setFirstName("Marko");
    personRepository.save(existingPerson);

    List<Person> savedPersons = personRepository.findAllById(List.of(personId));
    assertThat(savedPersons.get(0).getFirstName()).isEqualTo("Marko");
}
```

然后，我们将编写一个保存和删除现有人员记录的测试。我们期望这个测试执行一个插入、删除和选择数据库查询:

```java
@Test
void givenExistingPersonRecord_whenDeletingIt_thenRecordIsDeleted() {
    UUID personId = UUIDs.timeBased();
    Person existingPerson = new Person(personId, "Luka", "Modric");

    personRepository.delete(existingPerson);

    List<Person> savedPersons = personRepository.findAllById(List.of(personId));
    assertThat(savedPersons.isEmpty()).isTrue();
}
```

默认情况下，我们不会观察控制台中记录的任何数据库查询。

## 3.春季数据 CQL 测井

对于 Apache Cassandra 或更高版本的 Spring Data，可以让**为 `application.properties`中的`CqlTemplate`类**设置日志级别:

```java
logging.level.org.springframework.data.cassandra.core.cql.CqlTemplate=DEBUG
```

因此，通过将日志级别设置为 DEBUG，我们可以记录所有执行的查询和准备好的语句:

```java
2021-09-25 12:41:58.679 DEBUG 17856 --- [           main] o.s.data.cassandra.core.cql.CqlTemplate:
  Executing CQL statement [CREATE TABLE IF NOT EXISTS person
  (birthdate date, firstname text, id uuid, lastname text, lastpurchaseddate timestamp, lastvisiteddate timestamp, PRIMARY KEY (id));]
2021-09-25 12:42:01.204 DEBUG 17856 --- [           main] o.s.data.cassandra.core.cql.CqlTemplate:
  Preparing statement [INSERT INTO person (birthdate,firstname,id,lastname,lastpurchaseddate,lastvisiteddate)
  VALUES (?,?,?,?,?,?)] using org.springframework[[email protected]](/web/20221208143830/https://www.baeldung.com/cdn-cgi/l/email-protection)4d16975b
2021-09-25 12:42:01.253 DEBUG 17856 --- [           main] o.s.data.cassandra.core.cql.CqlTemplate:
  Executing prepared statement [INSERT INTO person (birthdate,firstname,id,lastname,lastpurchaseddate,lastvisiteddate) VALUES (?,?,?,?,?,?)]
2021-09-25 12:42:01.279 DEBUG 17856 --- [           main] o.s.data.cassandra.core.cql.CqlTemplate:
  Preparing statement [INSERT INTO person (birthdate,firstname,id,lastname,lastpurchaseddate,lastvisiteddate)
  VALUES (?,?,?,?,?,?)] using org.springframework[[email protected]](/web/20221208143830/https://www.baeldung.com/cdn-cgi/l/email-protection)539dd2d0
2021-09-25 12:42:01.290 DEBUG 17856 --- [           main] o.s.data.cassandra.core.cql.CqlTemplate:
  Executing prepared statement [INSERT INTO person (birthdate,firstname,id,lastname,lastpurchaseddate,lastvisiteddate) VALUES (?,?,?,?,?,?)]
2021-09-25 12:42:01.351 DEBUG 17856 --- [           main] o.s.data.cassandra.core.cql.CqlTemplate:
  Preparing statement [SELECT * FROM person WHERE id IN (371bb4a0-1ded-11ec-8cad-934f1aec79e6)]
  using org.springframework[[email protected]](/web/20221208143830/https://www.baeldung.com/cdn-cgi/l/email-protection)3e61cffd
2021-09-25 12:42:01.370 DEBUG 17856 --- [           main] o.s.data.cassandra.core.cql.CqlTemplate:
  Executing prepared statement [SELECT * FROM person WHERE id IN (371bb4a0-1ded-11ec-8cad-934f1aec79e6)]
```

不幸的是，使用这种解决方案，我们不会看到语句中使用的绑定值的输出。

## 4.数据税务请求跟踪器

[DataStax](/web/20221208143830/https://www.baeldung.com/datastax-post) 请求跟踪器是一个会话级的**组件，它会收到每个 Cassandra 请求的结果通知**。

Apache Cassandra 的 DataStax Java 驱动程序附带了一个可选的请求跟踪器实现，可以记录所有请求。

### 4.1.Noop 请求跟踪器

默认的请求跟踪器实现称为`NoopRequestTracker`。因此，它什么也不做:

```java
System.setProperty("datastax-java-driver.advanced.request-tracker.class", "NoopRequestTracker");
```

要设置不同的跟踪器，我们应该在 Cassandra 配置中或通过系统属性指定一个实现`RequestTracker` 的类。

### 4.2.请求记录器

`RequestLogger`是记录每个请求的`RequestTracker` 的**内置实现。**

我们可以通过设置特定的数据和 Java 驱动程序系统属性来启用它:

```java
System.setProperty("datastax-java-driver.advanced.request-tracker.class", "RequestLogger");
System.setProperty("datastax-java-driver.advanced.request-tracker.logs.success.enabled", "true");
System.setProperty("datastax-java-driver.advanced.request-tracker.logs.slow.enabled", "true");
System.setProperty("datastax-java-driver.advanced.request-tracker.logs.error.enabled", "true");
```

在这个例子中，我们启用了所有成功的、缓慢的和失败的请求的日志记录。

现在，当我们运行测试时，我们将在日志中观察所有执行的数据库查询:

```java
2021-09-25 13:06:31.799  INFO 11172 --- [        s0-io-4] c.d.o.d.i.core.tracker.RequestLogger:
  [s0|90232530][Node(endPoint=localhost/[0:0:0:0:0:0:0:1]:49281, hostId=c50413d5-03b6-4037-9c46-29f0c0da595a, hashCode=68c305fe)]
  Success (6 ms) [6 values] INSERT INTO person (birthdate,firstname,id,lastname,lastpurchaseddate,lastvisiteddate)
  VALUES (?,?,?,?,?,?) [birthdate=NULL, firstname='Luka', id=a3ad6890-1df0-11ec-a295-7d319da1858a, lastname='Modric', lastpurchaseddate=NULL, lastvisiteddate=NULL]
2021-09-25 13:06:31.811  INFO 11172 --- [        s0-io-4] c.d.o.d.i.core.tracker.RequestLogger:
  [s0|778232359][Node(endPoint=localhost/[0:0:0:0:0:0:0:1]:49281, hostId=c50413d5-03b6-4037-9c46-29f0c0da595a, hashCode=68c305fe)]
  Success (4 ms) [6 values] INSERT INTO person (birthdate,firstname,id,lastname,lastpurchaseddate,lastvisiteddate)
  VALUES (?,?,?,?,?,?) [birthdate=NULL, firstname='Marko', id=a3ad6890-1df0-11ec-a295-7d319da1858a, lastname='Modric', lastpurchaseddate=NULL, lastvisiteddate=NULL]
2021-09-25 13:06:31.847  INFO 11172 --- [        s0-io-4] c.d.o.d.i.core.tracker.RequestLogger:
  [s0|1947131919][Node(endPoint=localhost/[0:0:0:0:0:0:0:1]:49281, hostId=c50413d5-03b6-4037-9c46-29f0c0da595a, hashCode=68c305fe)]
  Success (5 ms) [0 values] SELECT * FROM person WHERE id IN (a3ad6890-1df0-11ec-a295-7d319da1858a)
```

我们将看到所有请求都记录在类别`com.datastax.oss.driver.internal.core.tracker.RequestLogger`下。

此外，**语句中使用的所有绑定值也会默认记录到**中。

### 4.3.界限值

内置的 **`RequestLogger`是高度可定制的组件**。我们可以使用以下系统属性配置绑定值的输出:

```java
System.setProperty("datastax-java-driver.advanced.request-tracker.logs.show-values", "true");
System.setProperty("datastax-java-driver.advanced.request-tracker.logs.max-value-length", "100");
System.setProperty("datastax-java-driver.advanced.request-tracker.logs.max-values", "100");
```

如果一个值的格式化表示比由`max-value-length`属性定义的值长，它将被截断。

使用`max-values` 属性，我们可以定义要记录的绑定值的最大数量。

### 4.4.附加选项

在第一个例子中，我们启用了慢速请求的日志记录。我们可以**使用`threshold`属性将一个成功的请求归类为慢速**:

```java
System.setProperty("datastax-java-driver.advanced.request-tracker.logs.slow.threshold ", "1 second");
```

默认情况下，会为所有失败的请求记录堆栈跟踪。如果我们禁用它们，我们将只能在日志中看到异常的字符串表示:

```java
System.setProperty("datastax-java-driver.advanced.request-tracker.logs.show-stack-trace", "true");
```

成功和缓慢的请求使用信息日志级别。另一方面，失败的请求使用错误级别。

## 5.结论

在本文中，我们探讨了在使用 Apache Cassandra 和 Spring Boot 时查询和语句的**日志记录。**

在示例中，我们介绍了为 Apache Cassandra 配置 Spring 数据中的日志级别。我们看到 Spring 数据将记录查询，但不记录绑定值。最后，我们探索了数据税务请求跟踪器。这是一个高度可定制的组件，我们可以用它来记录 Cassandra 查询及其绑定值。

和往常一样，源代码可以在 GitHub 上[找到](https://web.archive.org/web/20221208143830/https://github.com/Baeldung/datastax-cassandra/tree/main/spring-cassandra)