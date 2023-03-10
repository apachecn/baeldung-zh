# Java 中的 CockroachDB 指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/cockroachdb-java>

## 1。简介

本教程是在 Java 中使用 CockroachDB 的入门指南。

我们将解释关键特性，如何配置本地集群，如何监控它，以及如何使用 Java 与服务器连接和交互的实用指南。

我们先从定义它是什么开始。

## 2。cocroach db

CockroachDB 是一个分布式 SQL 数据库，构建在事务性和一致的键值存储之上。

用 Go 编写并且完全开源，它的主要设计目标是支持 ACID 事务、水平可伸缩性和生存性。根据这些设计目标，它旨在容忍从单个磁盘故障到整个数据中心崩溃的各种情况，将延迟中断降至最低，并且无需手动干预。

因此，**cocroach db 被认为是一个非常适合需要可靠、可用和正确数据的应用程序的解决方案，无论其规模如何。**但是，当非常低的延迟读取和写入至关重要时，这不是第一选择。

### 2.1。主要特征

让我们继续探索 CockroachDB 的一些关键方面:

*   **SQL API 和 PostgreSQL 兼容性**——用于结构化、操作和查询数据
*   **ACID 事务**–支持分布式事务并提供强大的一致性
*   **云就绪**–旨在运行在云中或本地解决方案上，提供不同云提供商之间的轻松迁移，不会造成任何服务中断
*   **水平扩展**—增加容量就像在运行的集群上指向一个新节点一样简单，只需最少的操作员开销
*   **复制**–复制数据以确保可用性，并保证副本之间的一致性
*   **自动修复**—只要大多数副本对于短期故障仍然可用，就可以无缝地继续，而对于长期故障，可以使用未受影响的副本作为源，从丢失的节点自动重新平衡副本

## 3。配置 cocroach db

在我们[安装了 cocroach db](https://web.archive.org/web/20220526052752/https://www.cockroachlabs.com/docs/stable/install-cockroachdb.html)之后，我们可以启动本地集群的第一个节点:

```java
cockroach start --insecure --host=localhost;
```

出于演示的目的，我们使用了`insecure`属性，使得通信不加密，而不需要指定证书的位置。

此时，我们的本地集群已经启动并运行。只有一个节点，我们已经可以连接到它并进行操作，但是**为了更好地利用 CockroachDB 的自动复制、重新平衡和容错功能，我们将再添加两个节点**:

```java
cockroach start --insecure --store=node2 \
  --host=localhost --port=26258 --http-port=8081 \
  --join=localhost:26257;

cockroach start --insecure --store=node3 \
  --host=localhost --port=26259 --http-port=8082 \
  --join=localhost:26257;
```

对于另外两个节点，我们使用`join` 标志将新节点连接到集群，指定第一个节点的地址和端口，在我们的例子中是 localhost:26257。**本地集群上的每个节点都需要唯一的`store`、`port`和`http-port`值。**

当配置 CockroachDB 的分布式集群时，每个节点将位于不同的机器上，因此可以避免指定`port`、`store`和`http-port`，因为缺省值已经足够了。此外，将其他节点加入集群时，应使用第一个节点的实际 IP。

### 3.1。配置数据库和用户

一旦我们的集群启动并运行，通过 CockroachDB 提供的 SQL 控制台，我们需要创建我们的数据库和一个用户。

首先，让我们启动 SQL 控制台:

```java
cockroach sql --insecure;
```

现在，让我们创建我们的`testdb` 数据库，创建一个用户并向该用户添加授权，以便能够执行 CRUD 操作:

```java
CREATE DATABASE testdb;
CREATE USER user17 with password 'qwerty';
GRANT ALL ON DATABASE testdb TO user17;
```

如果我们想要验证数据库是否正确创建，我们可以列出在当前节点中创建的所有数据库:

```java
SHOW DATABASES;
```

最后，如果我们想要验证 CockroachDB 的自动复制特性，我们可以检查另外两个节点中的一个是否正确创建了数据库。为此，我们必须在使用 SQL 控制台时表达`port`标志:

```java
cockroach sql --insecure --port=26258;
```

## 4。监控蟑螂 DB

现在，我们已经启动了本地集群并创建了数据库，**我们可以使用 CockroachDB 管理 UI** 来监控它们:

[![](img/69489017569d398b58c0954e71600c27.png)](/web/20220526052752/https://www.baeldung.com/wp-content/uploads/2018/01/CockroachDB_Monitoring-1024x487.png)

这个管理 UI 与 CockroachDB 捆绑在一起，一旦集群启动并运行，就可以在`http://localhost:8080`访问它。特别是，**它提供了关于集群和数据库配置的详细信息，并通过监控**等指标来帮助我们优化集群性能:

*   **集群运行状况**–关于集群运行状况的基本指标
*   **运行时指标**–关于节点数量、CPU 时间和内存使用的指标
*   **SQL 性能**–关于 SQL 连接、查询和事务的指标
*   **复制详细信息**—关于数据如何在集群中复制的指标
*   **节点详情**–活节点、死节点和退役节点的详情
*   **数据库详细信息**–集群中系统和用户数据库的详细信息

## 5.**项目设置**

给定我们正在运行的本地集群 CockroachDB，为了能够连接到它，我们必须向我们的`pom.xml:`添加一个[附加依赖](https://web.archive.org/web/20220526052752/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22org.postgresql%22%20AND%20a%3A%22postgresql%22)

```java
<dependency>
    <groupId>org.postgresql</groupId>
    <artifactId>postgresql</artifactId>
    <version>42.1.4</version>
</dependency>
```

或者，对于 Gradle 项目:

```java
compile 'org.postgresql:postgresql:42.1.4'
```

## 6。使用 cocroach db

现在我们已经清楚了我们在做什么，并且一切都设置妥当，让我们开始使用它。

得益于 PostgreSQL 的兼容性，**既可以直接与 JDBC 连接，也可以使用 ORM，如 Hibernate** (在撰写本文时(2018 年 1 月)，这两个驱动程序都经过了足够的测试，据开发人员称，它们已经获得了**测试级**支持)。在我们的例子中，我们将使用 JDBC 与数据库进行交互。

为了简单起见，我们将从基本的 CRUD 操作开始，因为它们是最好的开始。

让我们从连接数据库开始。

### 6.1。连接到 cocroach db

要打开与数据库的连接，我们可以使用`DriverManager` 类的`getConnection()`方法。这个方法需要一个连接 URL `String` 参数、一个用户名和一个密码:

```java
Connection con = DriverManager.getConnection(
  "jdbc:postgresql://localhost:26257/testdb", "user17", "qwerty"
);
```

### 6.2。创建表格

有了工作连接，我们就可以开始创建将用于所有 CRUD 操作的`articles`表:

```java
String TABLE_NAME = "articles";
StringBuilder sb = new StringBuilder("CREATE TABLE IF NOT EXISTS ")
  .append(TABLE_NAME)
  .append("(id uuid PRIMARY KEY, ")
  .append("title string,")
  .append("author string)");

String query = sb.toString();
Statement stmt = connection.createStatement();
stmt.execute(query);
```

如果我们想验证表是否被正确创建，我们可以使用`SHOW TABLES`命令:

```java
PreparedStatement preparedStatement = con.prepareStatement("SHOW TABLES");
ResultSet resultSet = preparedStatement.executeQuery();
List tables = new ArrayList<>();
while (resultSet.next()) {
    tables.add(resultSet.getString("Table"));
}

assertTrue(tables.stream().anyMatch(t -> t.equals(TABLE_NAME)));
```

让我们看看如何修改刚刚创建的表。

### 6.3。更改表格

如果我们在创建表的过程中遗漏了一些列，或者因为我们稍后需要它们，我们可以很容易地添加它们:

```java
StringBuilder sb = new StringBuilder("ALTER TABLE ").append(TABLE_NAME)
  .append(" ADD ")
  .append(columnName)
  .append(" ")
  .append(columnType);

String query = sb.toString();
Statement stmt = connection.createStatement();
stmt.execute(query);
```

一旦我们更改了表，我们可以使用`SHOW COLUMNS FROM`命令来验证新列是否被添加:

```java
String query = "SHOW COLUMNS FROM " + TABLE_NAME;
PreparedStatement preparedStatement = con.prepareStatement(query);
ResultSet resultSet = preparedStatement.executeQuery();
List<String> columns = new ArrayList<>();
while (resultSet.next()) {
    columns.add(resultSet.getString("Field"));
}

assertTrue(columns.stream().anyMatch(c -> c.equals(columnName)));
```

### 6.4。删除表格

使用表格时，有时我们需要删除它们，这可以通过几行代码轻松实现:

```java
StringBuilder sb = new StringBuilder("DROP TABLE IF EXISTS ")
  .append(TABLE_NAME);

String query = sb.toString();
Statement stmt = connection.createStatement();
stmt.execute(query);
```

### 6.5。插入数据

一旦我们明确了可以在表上执行的操作，我们现在就可以开始处理数据了。我们可以开始定义`Article`类:

```java
public class Article {

    private UUID id;
    private String title;
    private String author;

    // standard constructor/getters/setters
}
```

现在我们可以看看如何将一个`Article`添加到我们的`articles`表中:

```java
StringBuilder sb = new StringBuilder("INSERT INTO ").append(TABLE_NAME)
  .append("(id, title, author) ")
  .append("VALUES (?,?,?)");

String query = sb.toString();
PreparedStatement preparedStatement = connection.prepareStatement(query);
preparedStatement.setString(1, article.getId().toString());
preparedStatement.setString(2, article.getTitle());
preparedStatement.setString(3, article.getAuthor());
preparedStatement.execute();
```

### 6.6。读取数据

一旦数据存储在表中，我们就想读取这些数据，这很容易实现:

```java
StringBuilder sb = new StringBuilder("SELECT * FROM ")
  .append(TABLE_NAME);

String query = sb.toString();
PreparedStatement preparedStatement = connection.prepareStatement(query);
ResultSet rs = preparedStatement.executeQuery();
```

然而，如果我们不想读取`articles`表中的所有数据，而只想读取一个`Article`，我们可以简单地改变构建`PreparedStatement`的方式:

```java
StringBuilder sb = new StringBuilder("SELECT * FROM ").append(TABLE_NAME)
  .append(" WHERE title = ?");

String query = sb.toString();
PreparedStatement preparedStatement = connection.prepareStatement(query);
preparedStatement.setString(1, title);
ResultSet rs = preparedStatement.executeQuery();
```

### 6.7。删除数据

最后但同样重要的是，如果我们想从表中删除数据，我们可以使用标准的`DELETE FROM`命令删除一组有限的记录:

```java
StringBuilder sb = new StringBuilder("DELETE FROM ").append(TABLE_NAME)
  .append(" WHERE title = ?");

String query = sb.toString();
PreparedStatement preparedStatement = connection.prepareStatement(query);
preparedStatement.setString(1, title);
preparedStatement.execute();
```

或者我们可以用`TRUNCATE`函数删除表中的所有记录:

```java
StringBuilder sb = new StringBuilder("TRUNCATE TABLE ")
  .append(TABLE_NAME);

String query = sb.toString();
Statement stmt = connection.createStatement();
stmt.execute(query);
```

### 6.8。处理交易

一旦连接到数据库，默认情况下，每个 SQL 语句都被视为一个事务，并在执行完成后自动提交。

但是，如果我们希望将两个或更多 SQL 语句组合成一个事务，我们必须以编程方式控制该事务。

首先，我们需要通过将`Connection`的`autoCommit`属性设置为`false`来禁用自动提交模式，然后使用 `commit()`和`rollback()`方法来控制事务。

让我们看看如何在进行多次插入时实现数据一致性:

```java
try {
    con.setAutoCommit(false);

    UUID articleId = UUID.randomUUID();

    Article article = new Article(
      articleId, "Guide to CockroachDB in Java", "baeldung"
    );
    articleRepository.insertArticle(article);

    article = new Article(
      articleId, "A Guide to MongoDB with Java", "baeldung"
    );
    articleRepository.insertArticle(article); // Exception

    con.commit();
} catch (Exception e) {
    con.rollback();
} finally {
    con.setAutoCommit(true);
}
```

在这种情况下，在第二次插入时抛出了一个异常，因为违反了主键约束，因此没有文章被插入到`articles`表中。

## 7。结论

在本文中，我们解释了 CockroachDB 是什么，如何建立一个简单的本地集群，以及如何从 Java 与它进行交互。

一如既往，本文的完整源代码可以在 Github 的[中找到。](https://web.archive.org/web/20220526052752/https://github.com/eugenp/tutorials/tree/master/persistence-modules/java-cockroachdb)