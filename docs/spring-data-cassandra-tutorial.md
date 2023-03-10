# Spring Data Cassandra 简介

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-data-cassandra-tutorial>

## 1。概述

本文是一篇使用 Cassandra 处理 Spring 数据的实用介绍。

我们将从基础开始，通过配置和编码，最终构建一个完整的 Spring Data Cassandra 模块。

## 延伸阅读:

## 使用 Cassandra、Astra 和 Stargate 构建仪表板

Learn how to build a dashboard using DataStax Astra, a database-as-a-service powered by Apache Cassandra and Stargate APIs.[Read more](/web/20220812051945/https://www.baeldung.com/cassandra-astra-stargate-dashboard) →

## [用 Cassandra、Astra、REST & GraphQL 构建一个仪表板——记录状态更新](/web/20220812051945/https://www.baeldung.com/cassandra-astra-rest-dashboard-updates)

An example of using Cassandra to store time-series data.[Read more](/web/20220812051945/https://www.baeldung.com/cassandra-astra-rest-dashboard-updates) →

## [用卡珊德拉、阿斯特拉和 CQL 构建仪表板——映射事件数据](/web/20220812051945/https://www.baeldung.com/cassandra-astra-rest-dashboard-map)

Learn how to display events on an interactive map, based on data stored in an Astra database.[Read more](/web/20220812051945/https://www.baeldung.com/cassandra-astra-rest-dashboard-map) →

## 2。Maven 依赖关系

让我们从定义`pom.xml`中的依赖项开始，用 Maven:

```java
<dependency>
    <groupId>com.datastax.cassandra</groupId>
    <artifactId>cassandra-driver-core</artifactId>
    <version>2.1.9</version>
</dependency>
```

## 3。卡珊德拉的配置

我们将使用 Java 风格的配置来配置 Cassandra 集成。

### 3.1。主配置(弹簧)

我们将为此使用 Java 风格的配置。让我们从主配置类开始——当然是通过类级别 `@Configuration` 驱动的注释:

```java
@Configuration
public class CassandraConfig extends AbstractCassandraConfiguration {

    @Override
    protected String getKeyspaceName() {
        return "testKeySpace";
    }

    @Bean
    public CassandraClusterFactoryBean cluster() {
        CassandraClusterFactoryBean cluster = 
          new CassandraClusterFactoryBean();
        cluster.setContactPoints("127.0.0.1");
        cluster.setPort(9142);
        return cluster;
    }

    @Bean
    public CassandraMappingContext cassandraMapping() 
      throws ClassNotFoundException {
        return new BasicCassandraMappingContext();
    }
}
```

注意新的 bean–`BasicCassandraMappingContext`–带有默认实现。这是在持久实体的对象和持久格式之间映射持久实体所必需的。

由于默认实现足够强大，我们可以直接使用它。

### 3.2。主配置(Spring Boot)

让我们通过`application.properties` : 进行卡珊德拉配置

```java
spring.data.cassandra.keyspace-name=testKeySpace
spring.data.cassandra.port=9142
spring.data.cassandra.contact-points=127.0.0.1
```

我们完成了！这就是我们在使用 Spring Boot 时所需要的。

### 3.3。Cassandra 连接属性

要为 Cassandra 客户端设置连接，我们必须配置三个强制设置。

我们必须设置主机名，以 c `ontactPoints. Port`身份运行的 Cassandra 服务器只是服务器中请求的监听端口。`KeyspaceName` 是定义节点上数据复制的名称空间，它基于 Cassandra 相关的概念。

## 4。卡珊德拉知识库

我们将使用一个`CassandraRepository`作为数据访问层。这个遵循 Spring 数据存储库抽象，它专注于抽象出实现跨不同持久性机制的数据访问层所需的代码。

### 4.1。创建`CassandraRepository`

让我们创建要在配置中使用的`CassandraRepository`:

```java
@Repository
public interface BookRepository extends CassandraRepository<Book> {
    //
}
```

### 4.2。配置为`CassandraRepository`

现在，我们可以扩展 3.1 节中的配置，在 `CassandraConfig:`中添加`@EnableCassandraRepositories`类级注释来标记我们在 4.1 节中创建的 Cassandra 库

```java
@Configuration
@EnableCassandraRepositories(
  basePackages = "com.baeldung.spring.data.cassandra.repository")
public class CassandraConfig extends AbstractCassandraConfiguration {
    //
}
```

## 5。实体

让我们快速浏览一下实体——我们将要使用的模型类。该类是带注释的，并定义了在嵌入式模式下创建元数据 Cassandra 数据表的附加参数。

使用`@Table` 注释，bean 被直接映射到 Cassandra 数据表。每个属性也被定义为一种主键或一个简单的列:

```java
@Table
public class Book {
    @PrimaryKeyColumn(
      name = "isbn", 
      ordinal = 2, 
      type = PrimaryKeyType.CLUSTERED, 
      ordering = Ordering.DESCENDING)
    private UUID id;
    @PrimaryKeyColumn(
      name = "title", ordinal = 0, type = PrimaryKeyType.PARTITIONED)
    private String title;
    @PrimaryKeyColumn(
      name = "publisher", ordinal = 1, type = PrimaryKeyType.PARTITIONED)
    private String publisher;
    @Column
    private Set<String> tags = new HashSet<>();
    // standard getters and setters
}
```

## 6。使用嵌入式服务器进行测试

### 6.1。Maven 依赖关系

如果您想在嵌入式模式下运行 Cassandra(不需要手动安装单独的 Cassandra 服务器)，您需要将与`cassandra-unit`相关的依赖项添加到`pom.xml`:

```java
<dependency>
    <groupId>org.cassandraunit</groupId>
    <artifactId>cassandra-unit-spring</artifactId>
    <version>2.1.9.2</version>
    <scope>test</scope>
    <exclusions>
        <exclusion>
        <groupId>org.cassandraunit</groupId>
        <artifactId>cassandra-unit</artifactId>
        </exclusion>
    </exclusions>
</dependency>
<dependency>
    <groupId>org.cassandraunit</groupId>
    <artifactId>cassandra-unit-shaded</artifactId>
    <version>2.1.9.2</version>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>org.hectorclient</groupId>
    <artifactId>hector-core</artifactId>
    <version>2.0-0</version>
</dependency>
```

有可能使用嵌入式 Cassandra 服务器来测试这个应用程序。最主要的好处是你不想显式安装 Cassandra。

这个嵌入式服务器也兼容 Spring JUnit 测试。在这里，我们可以使用`@RunWith`注释和嵌入式服务器来设置`SpringJUnit4ClassRunner`。因此，在没有外部 Cassandra 服务运行的情况下，实现完整的测试套件是可能的。

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(classes = CassandraConfig.class)
public class BookRepositoryIntegrationTest {
    //
}
```

### 6.2。启动和停止服务器

如果您运行的是外部 Cassandra 服务器，可以忽略这一部分。

我们必须为整个测试套件启动一次服务器，所以服务器启动方法用`@BeforeClass` 标注:

```java
@BeforeClass
public static void startCassandraEmbedded() { 
    EmbeddedCassandraServerHelper.startEmbeddedCassandra(); 
    Cluster cluster = Cluster.builder()
      .addContactPoints("127.0.0.1").withPort(9142).build();
    Session session = cluster.connect(); 
}
```

接下来，我们必须确保服务器在测试套件执行完成后停止:

```java
@AfterClass
public static void stopCassandraEmbedded() {
    EmbeddedCassandraServerHelper.cleanEmbeddedCassandra();
}
```

### 6.3。清理数据表

在每次测试执行之前删除并创建数据表是一个很好的实践，这样可以避免由于在早期测试执行中操作数据而导致的意外结果。

现在，我们可以在服务器启动时创建数据表:

```java
@Before
public void createTable() {
    adminTemplate.createTable(
      true, CqlIdentifier.cqlId(DATA_TABLE_NAME), 
      Book.class, new HashMap<String, Object>());
}
```

并在每个测试用例执行后删除:

```java
@After
public void dropTable() {
    adminTemplate.dropTable(CqlIdentifier.cqlId(DATA_TABLE_NAME));
}
```

## 7。数据访问使用`CassandraRepository`

我们可以直接使用上面创建的`BookRepository`来持久化、操作和获取 Cassandra 数据库中的数据。

### 7.1。保存新书

我们可以将一本新书保存到书店:

```java
Book javaBook = new Book(
  UUIDs.timeBased(), "Head First Java", "O'Reilly Media", 
  ImmutableSet.of("Computer", "Software"));
bookRepository.save(ImmutableSet.of(javaBook));
```

然后，我们可以在数据库中检查插入图书的可用性:

```java
Iterable<Book> books = bookRepository.findByTitleAndPublisher(
  "Head First Java", "O'Reilly Media");
assertEquals(javaBook.getId(), books.iterator().next().getId());
```

### 7.2。更新现有图书

让我们从插入一本新书开始:

```java
Book javaBook = new Book(
  UUIDs.timeBased(), "Head First Java", "O'Reilly Media", 
  ImmutableSet.of("Computer", "Software"));
bookRepository.save(ImmutableSet.of(javaBook));
```

我们按书名取这本书吧:

```java
Iterable<Book> books = bookRepository.findByTitleAndPublisher(
  "Head First Java", "O'Reilly Media");
```

那我们换个书名:

```java
javaBook.setTitle("Head First Java Second Edition");
bookRepository.save(ImmutableSet.of(javaBook));
```

最后，让我们检查数据库中的标题是否已更新:

```java
Iterable<Book> books = bookRepository.findByTitleAndPublisher(
  "Head First Java Second Edition", "O'Reilly Media");
assertEquals(
  javaBook.getTitle(), updateBooks.iterator().next().getTitle());
```

### 7.3。删除现有图书

插入一本新书:

```java
Book javaBook = new Book(
  UUIDs.timeBased(), "Head First Java", "O'Reilly Media",
  ImmutableSet.of("Computer", "Software"));
bookRepository.save(ImmutableSet.of(javaBook));
```

然后删除新输入的图书:

```java
bookRepository.delete(javaBook); 
```

现在我们可以检查删除:

```java
Iterable<Book> books = bookRepository.findByTitleAndPublisher(
  "Head First Java", "O'Reilly Media");
assertNotEquals(javaBook.getId(), books.iterator().next().getId());
```

这将导致从代码中抛出一个 NoSuchElementException，以确保图书被删除。

### 7.4。查找所有书籍

首先插入一本新书:

```java
Book javaBook = new Book(
  UUIDs.timeBased(), "Head First Java", "O'Reilly Media", 
  ImmutableSet.of("Computer", "Software"));
Book dPatternBook = new Book(
  UUIDs.timeBased(), "Head Design Patterns","O'Reilly Media",
  ImmutableSet.of("Computer", "Software"));
bookRepository.save(ImmutableSet.of(javaBook));
bookRepository.save(ImmutableSet.of(dPatternBook));
```

查找所有书籍:

```java
Iterable<Book> books = bookRepository.findAll();
```

然后，我们可以查看数据库中现有图书的数量:

```java
int bookCount = 0;
for (Book book : books) bookCount++;
assertEquals(bookCount, 2);
```

## 8。结论

我们通过使用最常见的方法，使用`CassandraRepository`数据访问机制，对带有 Spring 数据的 Cassandra 进行了基本的实际操作介绍。

上述代码片段和示例的实现可以在[我的 GitHub 项目](https://web.archive.org/web/20220812051945/https://github.com/eugenp/tutorials/tree/master/persistence-modules/spring-data-cassandra)中找到——这是一个基于 Eclipse 的项目，因此它应该很容易导入和运行。