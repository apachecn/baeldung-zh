# Java 卡桑德拉指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/cassandra-with-java>

## 1。概述

本教程是使用 Java 的 Apache Cassandra 数据库的入门指南。

您将看到对关键概念的解释，以及一个工作示例，该示例涵盖了从 Java 连接并开始使用这个 NoSQL 数据库的基本步骤。

## 延伸阅读:

## [使用 Cassandra、Astra 和 Stargate 构建仪表板](/web/20221126114701/https://www.baeldung.com/cassandra-astra-stargate-dashboard)

了解如何使用 DataStax Astra(一种由 Apache Cassandra 和 Stargate APIs 支持的数据库即服务)构建仪表板。[阅读更多](/web/20221126114701/https://www.baeldung.com/cassandra-astra-stargate-dashboard) →

## [用 Cassandra、Astra、REST&graph QL——记录状态更新](/web/20221126114701/https://www.baeldung.com/cassandra-astra-rest-dashboard-updates)

用 Cassandra 存储时序数据的例子。[阅读更多信息](/web/20221126114701/https://www.baeldung.com/cassandra-astra-rest-dashboard-updates) →

## [用卡珊德拉、阿斯特拉和 CQL 构建一个仪表板——绘制事件数据](/web/20221126114701/https://www.baeldung.com/cassandra-astra-rest-dashboard-map)

了解如何根据存储在阿斯特拉数据库中的数据在交互式地图上显示事件。[阅读更多](/web/20221126114701/https://www.baeldung.com/cassandra-astra-rest-dashboard-map) →

## 2。卡珊德拉

Cassandra 是一个可扩展的 NoSQL 数据库，提供无单点故障的连续可用性，并能够以卓越的性能处理大量数据。

这个数据库使用环形设计，而不是使用主从架构。在环形设计中，没有主节点，所有参与节点都是相同的，并作为对等节点相互通信。

这使得 Cassandra 成为一个可水平扩展的系统，允许在不需要重新配置的情况下增加节点。

### 2.1。关键概念

让我们先来简单了解一下 Cassandra 的一些关键概念:

*   **集群**–以环形架构排列的节点或数据中心的集合。必须为每个集群分配一个名称，该名称随后将由参与的节点使用
*   **键空间**–如果你来自关系数据库，那么模式就是 Cassandra 中相应的键空间。在 Cassandra 中，keyspace 是最外层的数据容器。每个键空间要设置的主要属性是`Replication Factor`、`Replica Placement Strategy`和`Column Families`
*   **列族**–Cassandra 中的列族就像关系数据库中的表。每个列族包含一个由`Map<RowKey, SortedMap>`表示的行集合。密钥提供了一起访问相关数据的能力
*   **列**–Cassandra 中的列是包含列名、值和时间戳的数据结构。与数据结构良好的关系数据库相比，每一行中的列和列数可以变化

## 3。使用 Java 客户端

### 3.1。Maven 依赖关系

我们需要在`pom.xml`中定义以下 Cassandra 依赖，其最新版本可以在[这里](https://web.archive.org/web/20221126114701/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22com.datastax.cassandra%22)找到:

```java
<dependency>
    <groupId>com.datastax.cassandra</groupId>
    <artifactId>cassandra-driver-core</artifactId>
    <version>3.1.0</version>
</dependency>
```

为了用嵌入式数据库服务器测试代码，我们还应该添加`cassandra-unit`依赖项，它的最新版本可以在[这里找到](https://web.archive.org/web/20221126114701/https://search.maven.org/classic/#search%7Cga%7C1%7Ccassandra-unit):

```java
<dependency>
    <groupId>org.cassandraunit</groupId>
    <artifactId>cassandra-unit</artifactId>
    <version>3.0.0.1</version>
</dependency>
```

### 3.2。连接到卡珊德拉

为了从 Java 连接到 Cassandra，我们需要构建一个`Cluster`对象。

需要提供节点的地址作为联系点。如果我们不提供端口号，将使用默认端口(9042)。

这些设置允许驱动程序发现集群的当前拓扑。

```java
public class CassandraConnector {

    private Cluster cluster;

    private Session session;

    public void connect(String node, Integer port) {
        Builder b = Cluster.builder().addContactPoint(node);
        if (port != null) {
            b.withPort(port);
        }
        cluster = b.build();

        session = cluster.connect();
    }

    public Session getSession() {
        return this.session;
    }

    public void close() {
        session.close();
        cluster.close();
    }
}
```

### 3.3。创建密钥空间

让我们创建我们的"`library`"键空间:

```java
public void createKeyspace(
  String keyspaceName, String replicationStrategy, int replicationFactor) {
  StringBuilder sb = 
    new StringBuilder("CREATE KEYSPACE IF NOT EXISTS ")
      .append(keyspaceName).append(" WITH replication = {")
      .append("'class':'").append(replicationStrategy)
      .append("','replication_factor':").append(replicationFactor)
      .append("};");

    String query = sb.toString();
    session.execute(query);
}
```

除了从`keyspaceName`开始，我们还需要定义两个参数，即`replicationFactor` 和`replicationStrategy`。这些参数分别确定副本的数量以及副本将如何在环上分布。

借助复制，Cassandra 通过在多个节点中存储数据副本来确保可靠性和容错能力。

此时，我们可以测试我们的密钥空间是否已经成功创建:

```java
private KeyspaceRepository schemaRepository;
private Session session;

@Before
public void connect() {
    CassandraConnector client = new CassandraConnector();
    client.connect("127.0.0.1", 9142);
    this.session = client.getSession();
    schemaRepository = new KeyspaceRepository(session);
}
```

```java
@Test
public void whenCreatingAKeyspace_thenCreated() {
    String keyspaceName = "library";
    schemaRepository.createKeyspace(keyspaceName, "SimpleStrategy", 1);

    ResultSet result = 
      session.execute("SELECT * FROM system_schema.keyspaces;");

    List<String> matchedKeyspaces = result.all()
      .stream()
      .filter(r -> r.getString(0).equals(keyspaceName.toLowerCase()))
      .map(r -> r.getString(0))
      .collect(Collectors.toList());

    assertEquals(matchedKeyspaces.size(), 1);
    assertTrue(matchedKeyspaces.get(0).equals(keyspaceName.toLowerCase()));
}
```

### 3.4。创建柱族

现在，我们可以将第一个列族“books”添加到现有的键空间中:

```java
private static final String TABLE_NAME = "books";
private Session session;

public void createTable() {
    StringBuilder sb = new StringBuilder("CREATE TABLE IF NOT EXISTS ")
      .append(TABLE_NAME).append("(")
      .append("id uuid PRIMARY KEY, ")
      .append("title text,")
      .append("subject text);");

    String query = sb.toString();
    session.execute(query);
}
```

下面提供了测试柱族是否已创建的代码:

```java
private BookRepository bookRepository;
private Session session;

@Before
public void connect() {
    CassandraConnector client = new CassandraConnector();
    client.connect("127.0.0.1", 9142);
    this.session = client.getSession();
    bookRepository = new BookRepository(session);
}
```

```java
@Test
public void whenCreatingATable_thenCreatedCorrectly() {
    bookRepository.createTable();

    ResultSet result = session.execute(
      "SELECT * FROM " + KEYSPACE_NAME + ".books;");

    List<String> columnNames = 
      result.getColumnDefinitions().asList().stream()
      .map(cl -> cl.getName())
      .collect(Collectors.toList());

    assertEquals(columnNames.size(), 3);
    assertTrue(columnNames.contains("id"));
    assertTrue(columnNames.contains("title"));
    assertTrue(columnNames.contains("subject"));
}
```

### 3.5。改变柱族

一本书也有出版商，但是在创建的表中找不到这样的列。我们可以使用下面的代码来改变表格并添加一个新列:

```java
public void alterTablebooks(String columnName, String columnType) {
    StringBuilder sb = new StringBuilder("ALTER TABLE ")
      .append(TABLE_NAME).append(" ADD ")
      .append(columnName).append(" ")
      .append(columnType).append(";");

    String query = sb.toString();
    session.execute(query);
}
```

让我们确保已经添加了新列`publisher`:

```java
@Test
public void whenAlteringTable_thenAddedColumnExists() {
    bookRepository.createTable();

    bookRepository.alterTablebooks("publisher", "text");

    ResultSet result = session.execute(
      "SELECT * FROM " + KEYSPACE_NAME + "." + "books" + ";");

    boolean columnExists = result.getColumnDefinitions().asList().stream()
      .anyMatch(cl -> cl.getName().equals("publisher"));

    assertTrue(columnExists);
}
```

### 3.6。在列族中插入数据

现在已经创建了`books`表，我们准备开始向表中添加数据:

```java
public void insertbookByTitle(Book book) {
    StringBuilder sb = new StringBuilder("INSERT INTO ")
      .append(TABLE_NAME_BY_TITLE).append("(id, title) ")
      .append("VALUES (").append(book.getId())
      .append(", '").append(book.getTitle()).append("');");

    String query = sb.toString();
    session.execute(query);
}
```

“books”表中添加了一个新行，因此我们可以测试该行是否存在:

```java
@Test
public void whenAddingANewBook_thenBookExists() {
    bookRepository.createTableBooksByTitle();

    String title = "Effective Java";
    Book book = new Book(UUIDs.timeBased(), title, "Programming");
    bookRepository.insertbookByTitle(book);

    Book savedBook = bookRepository.selectByTitle(title);
    assertEquals(book.getTitle(), savedBook.getTitle());
}
```

在上面的测试代码中，我们使用了不同的方法来创建一个名为`booksByTitle:`的表

```java
public void createTableBooksByTitle() {
    StringBuilder sb = new StringBuilder("CREATE TABLE IF NOT EXISTS ")
      .append("booksByTitle").append("(")
      .append("id uuid, ")
      .append("title text,")
      .append("PRIMARY KEY (title, id));");

    String query = sb.toString();
    session.execute(query);
}
```

在 Cassandra 中，最佳实践之一是使用每个查询一个表的模式。这意味着，对于不同的查询，需要不同的表。

在我们的例子中，我们选择了根据书名来选择一本书。为了满足`selectByTitle`查询，我们使用列`title`和`id`创建了一个复合表`PRIMARY KEY`。列`title`是分区键，而`id`列是聚类键。

这样，您的数据模型中的许多表都包含重复数据。这不是这个数据库的缺点。相反，这种做法优化了读取的性能。

让我们看看当前保存在表中的数据:

```java
public List<Book> selectAll() {
    StringBuilder sb = 
      new StringBuilder("SELECT * FROM ").append(TABLE_NAME);

    String query = sb.toString();
    ResultSet rs = session.execute(query);

    List<Book> books = new ArrayList<Book>();

    rs.forEach(r -> {
        books.add(new Book(
          r.getUUID("id"), 
          r.getString("title"),  
          r.getString("subject")));
    });
    return books;
}
```

对返回预期结果的查询的测试:

```java
@Test
public void whenSelectingAll_thenReturnAllRecords() {
    bookRepository.createTable();

    Book book = new Book(
      UUIDs.timeBased(), "Effective Java", "Programming");
    bookRepository.insertbook(book);

    book = new Book(
      UUIDs.timeBased(), "Clean Code", "Programming");
    bookRepository.insertbook(book);

    List<Book> books = bookRepository.selectAll(); 

    assertEquals(2, books.size());
    assertTrue(books.stream().anyMatch(b -> b.getTitle()
      .equals("Effective Java")));
    assertTrue(books.stream().anyMatch(b -> b.getTitle()
      .equals("Clean Code")));
}
```

到目前为止，一切都很好，但有一点必须意识到。我们开始使用表`books,`，但是同时，为了满足`title`列的`select`查询，我们必须创建另一个名为`booksByTitle.`的表

这两个表是相同的，包含重复的列，但是我们只在`booksByTitle`表中插入了数据。因此，两个表中的数据目前不一致。

我们可以使用一个`batch`查询来解决这个问题，这个查询包含两个 insert 语句，每个表一个。一个`batch`查询将多个 DML 语句作为一个操作来执行。

提供了此类查询的一个示例:

```java
public void insertBookBatch(Book book) {
    StringBuilder sb = new StringBuilder("BEGIN BATCH ")
      .append("INSERT INTO ").append(TABLE_NAME)
      .append("(id, title, subject) ")
      .append("VALUES (").append(book.getId()).append(", '")
      .append(book.getTitle()).append("', '")
      .append(book.getSubject()).append("');")
      .append("INSERT INTO ")
      .append(TABLE_NAME_BY_TITLE).append("(id, title) ")
      .append("VALUES (").append(book.getId()).append(", '")
      .append(book.getTitle()).append("');")
      .append("APPLY BATCH;");

    String query = sb.toString();
    session.execute(query);
}
```

我们再次测试批量查询结果，如下所示:

```java
@Test
public void whenAddingANewBookBatch_ThenBookAddedInAllTables() {
    bookRepository.createTable();

    bookRepository.createTableBooksByTitle();

    String title = "Effective Java";
    Book book = new Book(UUIDs.timeBased(), title, "Programming");
    bookRepository.insertBookBatch(book);

    List<Book> books = bookRepository.selectAll();

    assertEquals(1, books.size());
    assertTrue(
      books.stream().anyMatch(
        b -> b.getTitle().equals("Effective Java")));

    List<Book> booksByTitle = bookRepository.selectAllBookByTitle();

    assertEquals(1, booksByTitle.size());
    assertTrue(
      booksByTitle.stream().anyMatch(
        b -> b.getTitle().equals("Effective Java")));
}
```

注意:从 3.0 版本开始，有一个叫做“物化视图”的新特性，我们可以用它来代替`batch`查询。这里有一个关于“物化视图”的详细记录的例子[。](https://web.archive.org/web/20221126114701/http://www.datastax.com/dev/blog/new-in-cassandra-3-0-materialized-views)

### 3.7。删除列族

下面的代码显示了如何删除表:

```java
public void deleteTable() {
    StringBuilder sb = 
      new StringBuilder("DROP TABLE IF EXISTS ").append(TABLE_NAME);

    String query = sb.toString();
    session.execute(query);
}
```

选择一个键空间中不存在的表会导致一个`InvalidQueryException: unconfigured table books`:

```java
@Test(expected = InvalidQueryException.class)
public void whenDeletingATable_thenUnconfiguredTable() {
    bookRepository.createTable();
    bookRepository.deleteTable("books");

    session.execute("SELECT * FROM " + KEYSPACE_NAME + ".books;");
}
```

### 3.8。删除键区

最后，让我们删除键空间:

```java
public void deleteKeyspace(String keyspaceName) {
    StringBuilder sb = 
      new StringBuilder("DROP KEYSPACE ").append(keyspaceName);

    String query = sb.toString();
    session.execute(query);
}
```

并测试密钥空间是否已被删除:

```java
@Test
public void whenDeletingAKeyspace_thenDoesNotExist() {
    String keyspaceName = "library";
    schemaRepository.deleteKeyspace(keyspaceName);

    ResultSet result = 
      session.execute("SELECT * FROM system_schema.keyspaces;");
    boolean isKeyspaceCreated = result.all().stream()
      .anyMatch(r -> r.getString(0).equals(keyspaceName.toLowerCase()));

    assertFalse(isKeyspaceCreated);
}
```

## 4。结论

本教程介绍了用 Java 连接和使用 Cassandra 数据库的基本步骤。还讨论了该数据库的一些关键概念，以帮助您快速入门。

本教程的完整实现可以在 [Github 项目](https://web.archive.org/web/20221126114701/https://github.com/eugenp/tutorials/tree/master/persistence-modules/java-cassandra)中找到。