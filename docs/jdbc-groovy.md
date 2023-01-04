# JDBC 与 Groovy

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/jdbc-groovy>

## 1。简介

在本文中，我们将看看如何使用惯用的 Groovy 用 [JDBC](/web/20220630125416/https://www.baeldung.com/java-jdbc) 查询关系数据库。

JDBC 虽然级别相对较低，但却是 JVM 上大多数 ORM 和其他高级数据访问库的基础。当然，我们可以在 Groovy 中直接使用 JDBC；然而，它有一个相当麻烦的 API。

幸运的是，Groovy 标准库建立在 JDBC 的基础上，提供了一个干净、简单而又强大的接口。因此，我们将探索 Groovy SQL 模块。

我们将用普通的 Groovy 来研究 JDBC，不考虑任何诸如 Spring 之类的框架，因为我们有其他指南。

## 2。JDBC 和 Groovy 设置

我们必须将`groovy-` sql 模块包含在我们的依赖项中:

```java
<dependency>
    <groupId>org.codehaus.groovy</groupId>
    <artifactId>groovy</artifactId>
    <version>2.4.13</version>
</dependency>
<dependency>
    <groupId>org.codehaus.groovy</groupId>
    <artifactId>groovy-sql</artifactId>
    <version>2.4.13</version>
</dependency>
```

如果我们使用 groovy-all，就没有必要明确列出它:

```java
<dependency>
    <groupId>org.codehaus.groovy</groupId>
    <artifactId>groovy-all</artifactId>
    <version>2.4.13</version>
</dependency>
```

我们可以在 Maven Central 上找到`[groovy](https://web.archive.org/web/20220630125416/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.codehaus.groovy%22%20AND%20a%3A%22groovy%22), [groovy-sql](https://web.archive.org/web/20220630125416/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.codehaus.groovy%22%20AND%20a%3A%22groovy-sql%22)` 和 `[groovy-all](https://web.archive.org/web/20220630125416/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.codehaus.groovy%22%20AND%20a%3A%22groovy-all%22)`的最新版本。

## 3。连接到数据库

为了使用数据库，我们必须做的第一件事是连接到数据库。

让我们来介绍一下`groovy.sql.Sql` 类，我们将使用 Groovy SQL 模块对数据库进行所有操作。

`Sql` 的一个实例代表一个我们想要操作的数据库。

然而， **`Sql`****的一个实例并不是一个单独的数据库连接**。我们稍后再讨论连接，现在先不要担心它们；让我们假设一切都神奇地工作。

### 3.1。指定连接参数

在本文中，我们将使用 HSQL 数据库，这是一种轻量级关系数据库，主要用于测试。

数据库连接需要 URL、驱动程序和访问凭据:

```java
Map dbConnParams = [
  url: 'jdbc:hsqldb:mem:testDB',
  user: 'sa',
  password: '',
  driver: 'org.hsqldb.jdbc.JDBCDriver']
```

这里，我们选择使用`Map`来指定这些，尽管这不是唯一可能的选择。

然后我们可以从`Sql` 类获得一个连接:

```java
def sql = Sql.newInstance(dbConnParams)
```

我们将在接下来的章节中看到如何使用它。

当我们完成时，我们应该总是释放任何相关的资源:

```java
sql.close()
```

### 3.2。使用`DataSource`

使用数据源连接数据库是很常见的，尤其是在应用服务器内部运行的程序中。

此外，当我们想要共享连接或使用 JNDI 时，数据源是最自然的选择。

Groovy 的`Sql`类很好地接受了数据源:

```java
def sql = Sql.newInstance(datasource)
```

### 3.3。自动资源管理

当我们处理完一个`Sql`实例时，记住调用`close()` 是很乏味的；毕竟，机器对事物的记忆比我们好得多。

使用 `Sql` ,我们可以将代码封装在一个闭包中，当控制权离开时，让 Groovy 自动调用`close()` ,即使出现异常:

```java
Sql.withInstance(dbConnParams) {
    Sql sql -> haveFunWith(sql)
}
```

## 4。针对数据库发布语句

现在，我们可以继续有趣的东西。

针对数据库发布语句的最简单、最不专业的方法是`execute` 方法:

```java
sql.execute "create table PROJECT (id integer not null, name varchar(50), url varchar(100))"
```

理论上，它对 DDL/DML 语句和查询都有效；然而，上面的简单表单没有提供获取查询结果的方法。我们将把问题留到以后。

`execute` 方法有几个重载版本，但是，同样，我们将在后面的章节中查看这个方法和其他方法的更高级的用例。

### 4.1。插入数据

对于在少量和简单的场景中插入数据，前面讨论的`execute` 方法非常好。

但是，如果我们已经生成了列(例如，使用序列或自动增量)并且我们想要知道生成的值，那么有一个专用的方法:`executeInsert` `.`

至于`execute`，我们现在将看看可用的最简单的方法重载，将更复杂的变体留到后面的部分。

因此，假设我们有一个带有自动递增主键的表(用 HSQLDB 的说法是 identity):

```java
sql.execute "create table PROJECT (ID IDENTITY, NAME VARCHAR (50), URL VARCHAR (100))"
```

让我们在表中插入一行，并将结果保存在一个变量中:

```java
def ids = sql.executeInsert """
  INSERT INTO PROJECT (NAME, URL) VALUES ('tutorials', 'github.com/eugenp/tutorials')
"""
```

`executeInsert` 的行为和`execute`完全一样，但是它返回什么？

原来返回值是一个矩阵:它的行是插入的行(记住一条语句可以导致多行被插入)，它的列是生成的值。

这听起来很复杂，但是在我们的例子中，这是最常见的例子，只有一行和一个生成的值:

```java
assertEquals(0, ids[0][0])
```

随后的插入将返回生成的值 1:

```java
ids = sql.executeInsert """
  INSERT INTO PROJECT (NAME, URL)
  VALUES ('REST with Spring', 'github.com/eugenp/REST-With-Spring')
"""

assertEquals(1, ids[0][0])
```

### 4.2。更新和删除数据

同样，数据修改和删除也有一个专用的方法:`executeUpdate`。

同样，这与`execute` 的区别仅在于它的返回值，我们只看它最简单的形式。

在这种情况下，返回值是一个整数，即受影响的行数:

```java
def count = sql.executeUpdate("UPDATE PROJECT SET URL = 'https://' + URL")

assertEquals(2, count)
```

## 5。查询数据库

当我们查询数据库时，事情开始变得很棒。

和 JDBC 这个阶层打交道并不有趣。幸运的是，Groovy 为所有这些提供了一个很好的抽象。

### 5.1。迭代查询结果

虽然循环是如此古老的风格……我们现在都喜欢闭包。

Groovy 适合我们的口味:

```java
sql.eachRow("SELECT * FROM PROJECT") { GroovyResultSet rs ->
    haveFunWith(rs)
}
```

`eachRow` 方法对数据库发出查询，并对每一行调用闭包。

正如我们所看到的，**一行由一个`GroovyResultSet`** 实例表示，它是普通旧`ResultSet` 的扩展，增加了一些好东西。请继续阅读，了解更多相关信息。

### 5.2。访问结果集

除了所有的`ResultSet` 方法，`GroovyResultSet` 还提供了一些方便的实用程序。

主要是，它公开匹配列名的命名属性:

```java
sql.eachRow("SELECT * FROM PROJECT") { rs ->
    assertNotNull(rs.name)
    assertNotNull(rs.URL)
}
```

注意属性名是不区分大小写的。

`GroovyResultSet`还提供使用从零开始的索引来访问列:

```java
sql.eachRow("SELECT * FROM PROJECT") { rs ->
    assertNotNull(rs[0])
    assertNotNull(rs[1])
    assertNotNull(rs[2])
}
```

### 5.3。分页

我们可以很容易地对结果进行分页，也就是说，只加载从某个偏移量开始直到某个最大行数的子集。例如，这是 web 应用程序中的一个常见问题。

`eachRow` 和相关方法具有接受偏移量和最大返回行数的重载:

```java
def offset = 1
def maxResults = 1
def rows = sql.rows('SELECT * FROM PROJECT ORDER BY NAME', offset, maxResults)

assertEquals(1, rows.size())
assertEquals('REST with Spring', rows[0].name)
```

这里，`rows` 方法返回一个行列表，而不是像`eachRow`那样遍历它们。

## 6。参数化查询和语句

通常情况下，查询和语句在编译时并不完全固定；它们通常以参数的形式包含静态部分和动态部分。

如果你正在考虑字符串连接，现在停下来，去看看 SQL 注入吧！

我们之前提到过，我们在前面章节中看到的方法有许多重载，适用于各种场景。

让我们介绍一下那些处理 SQL 查询和语句中的参数的重载。

### 6.1。带占位符的字符串

在类似于普通 JDBC 的风格中，我们可以使用位置参数:

```java
sql.execute(
    'INSERT INTO PROJECT (NAME, URL) VALUES (?, ?)',
    'tutorials', 'github.com/eugenp/tutorials')
```

或者我们可以在地图中使用命名参数:

```java
sql.execute(
    'INSERT INTO PROJECT (NAME, URL) VALUES (:name, :url)',
    [name: 'REST with Spring', url: 'github.com/eugenp/REST-With-Spring'])
```

这适用于`execute`、`executeUpdate`、`rows`和`eachRow`。`executeInsert` 也支持参数，但是它的签名有点不同，也更复杂。

### 6.2。Groovy 字符串

我们也可以使用带有占位符的 GStrings 来选择 Groovier 样式。

我们看到的所有方法都没有以通常的方式替换 GStrings 中的占位符；相反，他们将它们作为 JDBC 参数插入，确保 SQL 语法正确保留，不需要引用或转义任何内容，因此没有注入的风险。

这是非常好的，安全和时髦的:

```java
def name = 'REST with Spring'
def url = 'github.com/eugenp/REST-With-Spring'
sql.execute "INSERT INTO PROJECT (NAME, URL) VALUES (${name}, ${url})"
```

## 7 .**。交易和连接**

到目前为止，我们已经跳过了一个非常重要的问题:事务。

事实上，我们也没有谈到 Groovy 的`Sql`如何管理连接。

### 7.1。短期连接

在目前给出的例子中，**每个查询或语句都使用一个新的专用连接发送到数据库。** `Sql` 操作一终止就关闭连接。

当然，如果我们使用连接池，对性能的影响可能很小。

然而，**如果我们想将多个 DML 语句和查询作为一个单一的原子操作**来发布，我们需要一个事务。

此外，为了使事务首先成为可能，我们需要一个跨越多个语句和查询的连接。

### 7.2。具有缓存连接的事务

Groovy SQL 不允许我们显式地创建或访问事务。

相反，我们使用带有闭包的`withTransaction` 方法:

```java
sql.withTransaction {
    sql.execute """
        INSERT INTO PROJECT (NAME, URL)
        VALUES ('tutorials', 'github.com/eugenp/tutorials')
    """
    sql.execute """
        INSERT INTO PROJECT (NAME, URL)
        VALUES ('REST with Spring', 'github.com/eugenp/REST-With-Spring')
    """
}
```

在闭包内部，所有查询和语句都使用一个数据库连接。

此外，当闭包终止时，事务被自动提交，除非它由于异常而提前退出。

然而，我们也可以使用`Sql` 类中的方法手动提交或回滚当前事务:

```java
sql.withTransaction {
    sql.execute """
        INSERT INTO PROJECT (NAME, URL)
        VALUES ('tutorials', 'github.com/eugenp/tutorials')
    """
    sql.commit()
    sql.execute """
        INSERT INTO PROJECT (NAME, URL)
        VALUES ('REST with Spring', 'github.com/eugenp/REST-With-Spring')
    """
    sql.rollback()
}
```

### 7.3。没有事务的缓存连接

最后，为了重用没有上述事务语义的数据库连接，我们使用`cacheConnection`:

```java
sql.cacheConnection {
    sql.execute """
        INSERT INTO PROJECT (NAME, URL)
        VALUES ('tutorials', 'github.com/eugenp/tutorials')
    """
    throw new Exception('This does not roll back')
}
```

## 8。结论和进一步阅读

在本文中，我们研究了 Groovy SQL 模块，以及它如何用闭包和 Groovy 字符串增强和简化 JDBC。

我们可以有把握地得出这样的结论:朴素的老 JDBC 在点缀上一点时髦元素后看起来更加现代了！

我们还没有谈到 Groovy SQL 的每一个特性；例如，我们忽略了[批处理](/web/20220630125416/https://www.baeldung.com/jdbc-batch-processing)、存储过程、元数据和其他东西。

有关更多信息，请参见 Groovy 文档。

所有这些例子和代码片段的实现都可以在 GitHub 项目中找到——这是一个 Maven 项目，所以应该很容易导入和运行。