# 使用来自 Spring 数据的 CassandraTemplate

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-data-cassandratemplate-cqltemplate>

## 1。概述

这是春季数据 Cassandra 文章系列的第二篇文章。在本文中，我们将主要关注数据访问层中的`CassandraTemplate`和 CQL 查询。你可以在系列的[第一篇文章中阅读更多关于 Spring Data Cassandra 的内容。](/web/20221208143856/https://www.baeldung.com/spring-data-cassandra-tutorial)

Cassandra 查询语言(CQL)是 Cassandra 数据库的查询语言，而`CqlTemplate` 是 Spring Data Cassandra 中的底层数据访问模板——它方便地公开了与数据操作相关的操作，以执行 CQL 语句。

`CassandraTemplate` 构建在底层`CqlTemplate`之上，提供了一种简单的方法来查询域对象，并将对象映射到 Cassandra 中持久化的数据结构。

让我们从配置开始，然后深入到使用这两个模板的例子中。

## 延伸阅读:

## [使用 Cassandra、Astra 和 Stargate 构建仪表板](/web/20221208143856/https://www.baeldung.com/cassandra-astra-stargate-dashboard)

了解如何使用 DataStax Astra 构建仪表板，这是一种由 Apache Cassandra 和 Stargate APIs 支持的数据库即服务。[阅读更多](/web/20221208143856/https://www.baeldung.com/cassandra-astra-stargate-dashboard) →

## [用 Cassandra、Astra、REST&graph QL——记录状态更新](/web/20221208143856/https://www.baeldung.com/cassandra-astra-rest-dashboard-updates)

用 Cassandra 存储时序数据的例子。[阅读更多信息](/web/20221208143856/https://www.baeldung.com/cassandra-astra-rest-dashboard-updates) →

## [用卡珊德拉、阿斯特拉和 CQL 构建一个仪表盘——绘制事件数据](/web/20221208143856/https://www.baeldung.com/cassandra-astra-rest-dashboard-map)

了解如何根据存储在阿斯特拉数据库中的数据在交互式地图上显示事件。[阅读更多](/web/20221208143856/https://www.baeldung.com/cassandra-astra-rest-dashboard-map) →

## 2。`CassandraTemplate` 配置

`CassandraTemplate`在 Spring 上下文中是可用的，因为我们的主 Cassandra Spring 配置扩展了 AbstractCassandraConfiguration:

```
@Configuration
@EnableCassandraRepositories(basePackages = "com.baeldung.spring.data.cassandra.repository")
public class CassandraConfig extends AbstractCassandraConfiguration { ... }
```

然后，我们可以简单地连接模板——或者通过它的确切类型 CassandraTemplate，或者作为更通用的接口`CassandraOperations:`

```
@Autowired
private CassandraOperations cassandraTemplate;
```

## 3。数据访问使用`CassandraTemplate`

让我们在数据访问层模块中使用上面定义的`CassandraTemplate`来处理数据持久性。

### 3.1。保存新书

我们可以将一本新书保存到书店:

```
Book javaBook = new Book(
  UUIDs.timeBased(), "Head First Java", "O'Reilly Media",
  ImmutableSet.of("Computer", "Software"));
cassandraTemplate.insert(javaBook);
```

然后，我们可以在数据库中检查插入图书的可用性:

```
Select select = QueryBuilder.select().from("book")
  .where(QueryBuilder.eq("title", "Head First Java"))
  .and(QueryBuilder.eq("publisher", "O'Reilly Media"));
Book retrievedBook = cassandraTemplate.selectOne(select, Book.class);
```

我们在这里使用一个`Select QueryBuilder`，映射到`cassandraTemplate`中的`selectOne()`。我们将在 CQL 查询部分更深入地讨论`QueryBuilder`。

### 3.2。保存多本书

我们可以使用列表一次将多本书保存到书店:

```
Book javaBook = new Book(
  UUIDs.timeBased(), "Head First Java", "O'Reilly Media",
  ImmutableSet.of("Computer", "Software"));
Book dPatternBook = new Book(
  UUIDs.timeBased(), "Head Design Patterns", "O'Reilly Media",
  ImmutableSet.of("Computer", "Software"));
List<Book> bookList = new ArrayList<Book>();
bookList.add(javaBook);
bookList.add(dPatternBook);
cassandraTemplate.insert(bookList);
```

### 3.3。更新现有图书

让我们从插入一本新书开始:

```
Book javaBook = new Book(
  UUIDs.timeBased(), "Head First Java", "O'Reilly Media",
  ImmutableSet.of("Computer", "Software"));
cassandraTemplate.insert(javaBook);
```

我们去拿书吧:

```
Select select = QueryBuilder.select().from("book");
Book retrievedBook = cassandraTemplate.selectOne(select, Book.class);
```

然后我们给检索到的书添加一些额外的标签:

```
retrievedBook.setTags(ImmutableSet.of("Java", "Programming"));
cassandraTemplate.update(retrievedBook);
```

### 3.4。删除插入的书籍

让我们插入一本新书:

```
Book javaBook = new Book(
  UUIDs.timeBased(), "Head First Java", "O'Reilly Media",
  ImmutableSet.of("Computer", "Software"));
cassandraTemplate.insert(javaBook);
```

然后删除该书:

```
cassandraTemplate.delete(javaBook);
```

### 3.5。删除所有书籍

现在让我们插入一些新书:

```
Book javaBook = new Book(
  UUIDs.timeBased(), "Head First Java", "O'Reilly Media",
  ImmutableSet.of("Computer", "Software"));
Book dPatternBook = new Book(
  UUIDs.timeBased(), "Head Design Patterns", "O'Reilly Media", 
  ImmutableSet.of("Computer", "Software"));
cassandraTemplate.insert(javaBook);
cassandraTemplate.insert(dPatternBook);
```

然后删除所有的书:

```
cassandraTemplate.deleteAll(Book.class);
```

## 4。使用 CQL 查询的数据访问

在数据访问层中，总是可以使用 CQL 查询进行数据操作。CQL 查询处理由`CqlTemplate`类执行，允许我们根据需要执行定制查询。

然而，由于`CassandraTemplate` 类是`CqlTemplate`的扩展，我们可以直接使用它来执行这些查询。

让我们看看使用 CQL 查询操作数据的不同方法。

### 4.1。使用`QueryBuilder`

`QueryBuilder` 可用于建立数据库中数据操作的查询。几乎所有的标准操作都可以使用现成的构造块来构建:

```
Insert insertQueryBuider = QueryBuilder.insertInto("book")
 .value("isbn", UUIDs.timeBased())
 .value("title", "Head First Java")
 .value("publisher", "OReilly Media")
 .value("tags", ImmutableSet.of("Software"));
cassandraTemplate.execute(insertQueryBuider);
```

如果仔细观察代码片段，您可能会注意到使用了`execute()` 方法，而不是相关的操作类型(插入、删除等)。).这是因为查询的类型是由`QueryBuilder.`的输出定义的

### 4.2。使用`PreparedStatements`

尽管`PreparedStatements`可用于任何情况，但这种机制通常推荐用于高速摄取的多次插入。

A `PreparedStatement`只准备一次，有助于确保高性能:

```
UUID uuid = UUIDs.timeBased();
String insertPreparedCql = 
  "insert into book (isbn, title, publisher, tags) values (?, ?, ?, ?)";
List<Object> singleBookArgsList = new ArrayList<>();
List<List<?>> bookList = new ArrayList<>();
singleBookArgsList.add(uuid);
singleBookArgsList.add("Head First Java");
singleBookArgsList.add("OReilly Media");
singleBookArgsList.add(ImmutableSet.of("Software"));
bookList.add(singleBookArgsList);
cassandraTemplate.ingest(insertPreparedCql, bookList);
```

### 4.3。使用 CQL 语句

我们可以直接使用 CQL 语句来查询如下数据:

```
UUID uuid = UUIDs.timeBased();
String insertCql = "insert into book (isbn, title, publisher, tags) 
  values (" + uuid + ", 'Head First Java', 'OReilly Media', {'Software'})";
cassandraTemplate.execute(insertCql);
```

## 5。结论

在本文中，我们使用 Spring Data Cassandra 研究了各种数据操作策略，包括`CassandraTemplate`和 CQL 查询。

上述代码片段和示例的实现可以在[我的 GitHub 项目](https://web.archive.org/web/20221208143856/https://github.com/eugenp/tutorials/tree/master/persistence-modules/spring-data-cassandra)中找到——这是一个基于 Maven 的项目，所以它应该很容易导入和运行。