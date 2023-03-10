# sql2o JDBC 包装器指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-sql2o>

## 1。简介

在本教程中，我们将了解一下 [Sql2o](https://web.archive.org/web/20221206170835/https://www.sql2o.org/) ，这是一个小型快速的库，用于在惯用 Java 中访问关系数据库。

值得一提的是，尽管 Sql2o 通过将查询结果映射到 POJOs(普通旧 Java 对象)来工作，但它并不是一个完整的 ORM 解决方案，如 Hibernate。

## 2。Sql2o 设置

**Sql2o 是一个单独的 jar 文件，我们可以很容易地将它添加到我们项目的依赖项中:**

```java
<dependency>
    <groupId>org.sql2o</groupId>
    <artifactId>sql2o</artifactId>
    <version>1.6.0</version>
</dependency>
```

在我们的例子中，我们还将使用嵌入式数据库 HSQL 为了跟进，我们也可以包括它:

```java
<dependency>
    <groupId>org.hsqldb</groupId>
    <artifactId>hsqldb</artifactId>
    <version>2.4.0</version>
    <scope>test</scope>
</dependency>
```

Maven Central 托管最新版本的`[sql2o](https://web.archive.org/web/20221206170835/https://search.maven.org/search?q=g:org.sql2o%20AND%20a:sql2o&core=gav)`和 [HSQLDB](https://web.archive.org/web/20221206170835/https://search.maven.org/classic/#search|gav|1|g%3A%22org.hsqldb%22%20AND%20a%3A%22hsqldb%22) 。

## 3。连接到数据库

**为了建立连接，我们从`Sql2o` 类的一个实例开始:**

```java
Sql2o sql2o = new Sql2o("jdbc:hsqldb:mem:testDB", "sa", "");
```

这里，我们将连接 URL、用户名和密码指定为构造函数参数。

`Sql2o` 对象是线程安全的，我们可以在整个应用程序中共享它。

### 3.1。使用`DataSource`

**在大多数应用程序中，我们希望使用一个** `**DataSource** `而不是一个原始的`DriverManager` 连接，也许是为了利用连接池，或者指定额外的连接参数。别担心，Sql2o 已经为我们做好了准备:

```java
Sql2o sql2o = new Sql2o(datasource);
```

### 3.2。使用连接

仅仅实例化一个 `Sql2o`对象不会建立到数据库的任何连接。

相反，**我们使用`open` 方法得到一个`Connection` 对象**(注意不是 JDBC `Connection`)。由于`Connection`是`AutoCloseable,` ，我们可以将其包装在 [try-with-resources](/web/20221206170835/https://www.baeldung.com/java-try-with-resources) 块中:

```java
try (Connection connection = sql2o.open()) {
    // use the connection
}
```

## 4。插入和更新语句

现在让我们创建一个数据库并将一些数据放入其中。在整个教程中，我们将使用一个名为`project:`的简单表格

```java
connection.createQuery(
    "create table project "
    + "(id integer identity, name varchar(50), url varchar(100))").executeUpdate();
```

`executeUpdate` 返回`Connection` 对象，这样我们可以链接多个调用。然后，如果我们想知道受影响的行数，我们使用`getResult:`

```java
assertEquals(0, connection.getResult());
```

我们将对所有 DDL、INSERT 和 UPDATE 语句应用我们刚刚看到的模式—`createQuery` 和`executeUpdate –` **。**

### 4.1。获取生成的键值

但是，在某些情况下，我们可能希望取回生成的键值。这些是自动计算的键列的值(比如在某些数据库上使用自动增量)。

我们分两步走。首先，用一个附加参数来`createQuery:`

```java
Query query = connection.createQuery(
    "insert into project (name, url) "
    + "values ('tutorials', 'github.com/eugenp/tutorials')", true);
```

然后，在连接上调用`getKey` :

```java
assertEquals(0, query.executeUpdate().getKey());
```

如果键不止一个，我们使用`getKeys` 来代替，它返回一个数组:

```java
assertEquals(1, query.executeUpdate().getKeys()[0]);
```

## 5。从数据库中提取数据

现在让我们进入问题的核心: **SELECT** **查询和结果集到 Java 对象的映射。**

首先，我们必须用 getters 和 setters 定义一个 POJO 类来表示我们的项目表:

```java
public class Project {
    long id;
    private String name;
    private String url;
    //Standard getters and setters
}
```

然后，像以前一样，我们将编写我们的查询:

```java
Query query = connection.createQuery("select * from project order by id");
```

不过，这次我们要用一个新方法，`executeAndFetch:`

```java
List<Project> list = query.executeAndFetch(Project.class);
```

正如我们所看到的，该方法将结果的类作为参数，Sql2o 将来自数据库的原始结果集的行映射到该参数。

### 5.1。列映射

**Sql2o 通过名称将列映射到 JavaBean 属性，**不区分大小写。

然而，Java 和关系数据库之间的命名约定是不同的。假设我们向项目添加了一个创建日期属性:

```java
public class Project {
    long id;
    private String name;
    private String url;
    private Date creationDate;
    //Standard getters and setters
}
```

在数据库模式中，我们很可能会调用相同的属性`creation_date.`

当然，我们可以在查询中给它起别名:

```java
Query query = connection.createQuery(
    "select name, url, creation_date as creationDate from project");
```

然而，这很乏味，我们失去了使用`select *.`的可能性

另一个选项是指示 Sql2o 将`creation_date` 映射到`creationDate.`，也就是说，我们可以告诉查询关于映射的信息:

```java
connection.createQuery("select * from project")
    .addColumnMapping("creation_date", "creationDate");
```

如果我们在一些查询中谨慎地使用`creationDate` ,这很好；然而，当在大型项目中广泛使用时，一遍又一遍地讲述相同的事实会变得乏味且容易出错。

幸运的是，我们还可以**全局指定映射:**

```java
Map<String, String> mappings = new HashMap<>();
mappings.put("CREATION_DATE", "creationDate");
sql2o.setDefaultColumnMappings(mappings);
```

当然，这将导致`creation_date`的每个实例都被映射到`creationDate`，所以这是努力在我们的数据定义中保持名称一致的另一个原因。

### 5.2。标量结果

有时，我们希望从查询中提取单个标量结果。例如，当我们需要计算记录的数量时。

在这些情况下，定义一个类并迭代一个我们知道包含单个元素的列表是多余的。因此， **Sql2o 包含了`executeScalar` 方法:**

```java
Query query = connection.createQuery(
    "select count(*) from project");
assertEquals(2, query.executeScalar(Integer.class));
```

这里，我们将返回类型指定为`Integer`。然而，这是可选的，我们可以让底层的 JDBC 驱动程序来决定。

### 5.3。复杂结果

相反，有时复杂的查询(比如报告)可能不容易映射到 Java 对象上。我们还可能决定不要编写一个只在单个查询中使用的 Java 类。

因此， **Sql2o 还允许一个到表格数据结构的低级动态映射。**我们使用`executeAndFetchTable` 方法获得:

```java
Query query = connection.createQuery(
    "select * from project order by id");
Table table = query.executeAndFetchTable();
```

然后，我们可以提取地图列表:

```java
List<Map<String, Object>> list = table.asList();
assertEquals("tutorials", list.get(0).get("name"));
```

或者，我们可以将数据映射到一列`Row` 对象上，这些对象是从列名到值的映射，类似于`ResultSet` s:

```java
List<Row> rows = table.rows();
assertEquals("tutorials", rows.get(0).getString("name"));
```

## 6。绑定查询参数

许多 SQL 查询都有一个固定的结构，只有几个参数化的部分。我们可能会天真地编写那些带有字符串连接的部分动态查询。

但是，Sql2o 允许参数化查询，因此:

*   我们避开 [SQL 注入的攻击](/web/20221206170835/https://www.baeldung.com/sql-injection)
*   我们允许数据库缓存经常使用的查询并提高性能
*   最后，我们不再需要对日期和时间等复杂类型进行编码

因此，我们可以在 Sql2o 中使用命名参数来实现上述所有功能。我们用冒号引入参数，并用`addParameter` 方法绑定它们:

```java
Query query = connection.createQuery(
    "insert into project (name, url) values (:name, :url)")
    .addParameter("name", "REST with Spring")
    .addParameter("url", "github.com/eugenp/REST-With-Spring");
assertEquals(1, query.executeUpdate().getResult());
```

### 6.1。从 POJO 绑定

Sql2o 提供了另一种绑定参数的方式:即使用 POJO 作为源来绑定**。当一个查询有许多参数并且它们都指向同一个实体时，这种技术特别适合。那么，让我们来介绍一下**的`bind` 方法:****

```java
Project project = new Project();
project.setName("REST with Spring");
project.setUrl("github.com/eugenp/REST-With-Spring");
connection.createQuery(
    "insert into project (name, url) values (:name, :url)")
    .bind(project)
    .executeUpdate();
assertEquals(1, connection.getResult());
```

## 7。交易和批量查询

对于事务，我们可以将多个 SQL 语句作为单个原子操作来发布。即要么成功，要么批量失败，没有中间结果。事实上，事务是关系数据库的关键特性之一。

为了打开一个事务，我们使用了`beginTransaction` 方法，而不是我们目前使用的`open` 方法:

```java
try (Connection connection = sql2o.beginTransaction()) {
    // here, the transaction is active
}
```

当执行离开块时， **Sql2o 自动回滚事务**,如果它仍然是活动的。

### 7.1。手动提交和回滚

然而，**我们可以用适当的方法显式地提交或回滚事务:**

```java
try (Connection connection = sql2o.beginTransaction()) {
    boolean transactionSuccessful = false;
    // perform some operations
    if(transactionSuccessful) {
        connection.commit();
    } else {
        connection.rollback();
    }
}
```

注意，**`commit` 和`rollback` 都结束事务。**后续语句将在没有事务的情况下运行，因此它们不会在块结束时自动回滚。

但是，我们可以提交或回滚事务而不结束它:

```java
try (Connection connection = sql2o.beginTransaction()) {
    List list = connection.createQuery("select * from project")
        .executeAndFetchTable()
        .asList();
    assertEquals(0, list.size());
    // insert or update some data
    connection.rollback(false);
    // perform some other insert or update queries
}
// implicit rollback
try (Connection connection = sql2o.beginTransaction()) {
    List list = connection.createQuery("select * from project")
        .executeAndFetchTable()
        .asList();
    assertEquals(0, list.size());
}
```

### 7.2。批量操作

当我们需要**用不同的参数多次发出相同的语句时，**批量运行它们可以提供很大的性能优势。

幸运的是，通过组合我们到目前为止描述的两种技术——参数化查询和事务——很容易批量运行它们:

*   首先，我们只创建一次查询
*   然后，我们绑定参数，并为查询的每个实例调用`addToBatch`
*   最后，我们称之为`executeBatch:`

```java
try (Connection connection = sql2o.beginTransaction()) {
    Query query = connection.createQuery(
        "insert into project (name, url) " +
        "values (:name, :url)");
    for (int i = 0; i < 1000; i++) {
        query.addParameter("name", "tutorials" + i);
        query.addParameter("url", "https://github.com/eugenp/tutorials" + i);
        query.addToBatch();
    }
    query.executeBatch();
    connection.commit();
}
try (Connection connection = sql2o.beginTransaction()) {
    assertEquals(
        1000L,
        connection.createQuery("select count(*) from project").executeScalar());
}
```

### 7.3。延迟获取

相反，当单个查询返回大量结果时，将它们全部转换并存储在一个列表中会占用大量内存。

因此，Sql2o 支持惰性模式，在这种模式下，每次返回并映射一行:

```java
Query query = connection.createQuery("select * from project");
try (ResultSetIterable<Project> projects = query.executeAndFetchLazy(Project.class)) {
    for(Project p : projects) {
        // do something with the project
    }
}
```

请注意，`ResultSetIterable` 是`AutoCloseable` ,用于和`try-with-resources`一起使用，以在完成时关闭底层的`ResultSet` 。

## 8。结论

在本教程中，我们概述了 Sql2o 库及其最常见的使用模式。更多信息可以在 GitHub 上的 [Sql20 wiki 中找到。](https://web.archive.org/web/20221206170835/https://github.com/aaberg/sql2o/wiki)

此外，所有这些示例和代码片段的实现都可以在 GitHub 项目中找到，这是一个 Maven 项目，因此它应该很容易导入并按原样运行。