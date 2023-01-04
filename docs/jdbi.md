# Jdbi 指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/jdbi>

## 1。简介

在本文中，我们将看看如何使用 [jdbi](https://web.archive.org/web/20221001092901/http://jdbi.org/) 查询关系数据库。

Jdbi 是一个开源 Java 库(Apache license)，它使用 [lambda 表达式](/web/20221001092901/https://www.baeldung.com/java-8-lambda-expressions-tips)和[反射](/web/20221001092901/https://www.baeldung.com/java-reflection)来提供比 [JDBC](/web/20221001092901/https://www.baeldung.com/java-jdbc) 更友好、更高级别的接口来访问数据库。

然而，Jdbi 不是 ORM 尽管它有一个可选的 SQL 对象映射模块，但它没有一个带有附加对象的会话、一个数据库独立层和任何其他典型 ORM 的附加功能。

## 2。Jdbi 设置

Jdbi 由一个核心模块和几个可选模块组成。

要开始，我们只需在依赖项中包含核心模块:

```java
<dependencies>
    <dependency>
        <groupId>org.jdbi</groupId>
        <artifactId>jdbi3-core</artifactId>
        <version>3.1.0</version>
    </dependency>
</dependencies>
```

在本文中，我们将展示使用 HSQL 数据库的例子:

```java
<dependency>
    <groupId>org.hsqldb</groupId>
    <artifactId>hsqldb</artifactId>
    <version>2.4.0</version>
    <scope>test</scope>
</dependency>
```

我们可以在 Maven Central 上找到最新版本的`[jdbi3-core](https://web.archive.org/web/20221001092901/https://search.maven.org/classic/#search|gav|1|g%3A%22org.jdbi%22%20AND%20a%3A%22jdbi3-core%22)`、 [HSQLDB](https://web.archive.org/web/20221001092901/https://search.maven.org/classic/#search|gav|1|g%3A%22org.hsqldb%22%20AND%20a%3A%22hsqldb%22) 和其他 Jdbi 模块。

## 3。连接到数据库

首先，我们需要连接到数据库。为此，我们必须指定连接参数。

起点是`Jdbi` 类:

```java
Jdbi jdbi = Jdbi.create("jdbc:hsqldb:mem:testDB", "sa", "");
```

这里，我们指定连接 URL、用户名，当然还有密码。

### 3.1。附加参数

如果我们需要提供其他参数，我们使用一个接受`Properties` 对象的重载方法:

```java
Properties properties = new Properties();
properties.setProperty("username", "sa");
properties.setProperty("password", "");
Jdbi jdbi = Jdbi.create("jdbc:hsqldb:mem:testDB", properties);
```

在这些例子中，我们将`Jdbi` 实例保存在一个局部变量中。这是因为我们将使用它向数据库发送语句和查询。

事实上，仅仅调用`create` 并不能建立到数据库的任何连接。它只是保存连接参数供以后使用。

### 3.2。使用`DataSource`

如果我们使用`DataSource`连接到数据库，通常情况下，我们可以使用适当的`create` 重载:

```java
Jdbi jdbi = Jdbi.create(datasource);
```

### 3.3。使用手柄

到数据库的实际连接由`Handle` 类的实例表示。

使用句柄并让它们自动关闭的最简单方法是使用 lambda 表达式:

```java
jdbi.useHandle(handle -> {
    doStuffWith(handle);
});
```

当我们不需要返回值时，我们调用`useHandle` 。

否则，我们使用`withHandle`:

```java
jdbi.withHandle(handle -> {
    return computeValue(handle);
});
```

也可以手动打开连接句柄，尽管不推荐这样做；在这种情况下，我们必须在完成后关闭它:

```java
Jdbi jdbi = Jdbi.create("jdbc:hsqldb:mem:testDB", "sa", "");
try (Handle handle = jdbi.open()) {
    doStuffWith(handle);
}
```

幸运的是，正如我们所见，`Handle` 实现了`Closeable`，因此它可以与 [try-with-resources](/web/20221001092901/https://www.baeldung.com/java-try-with-resources) 一起使用。

## 4。简单语句

现在我们知道了如何获得连接，让我们看看如何使用它。

在这一节中，我们将创建一个贯穿整篇文章的简单表格。

为了向数据库发送像`create table`这样的语句，我们使用了`execute` 方法:

```java
handle.execute(
  "create table project "
  + "(id integer identity, name varchar(50), url varchar(100))");
```

`execute` 返回受语句影响的行数:

```java
int updateCount = handle.execute(
  "insert into project values "
  + "(1, 'tutorials', 'github.com/eugenp/tutorials')");

assertEquals(1, updateCount);
```

实际上，execute 只是一个方便的方法。

我们将在后面的章节中研究更复杂的用例，但是在此之前，我们需要学习如何从数据库中提取结果。

## 5。查询数据库

从数据库产生结果的最直接的表达式是 SQL 查询。

要使用 Jdbi 句柄发出查询，我们至少必须:

1.  创建查询
2.  选择如何表示每一行
3.  迭代结果

我们现在来看看上面的每一点。

### 5.1。创建查询

不出所料， **Jdbi 将查询表示为`Query` 类的实例。**

我们可以从句柄中获得一个:

```java
Query query = handle.createQuery("select * from project");
```

### 5.2。映射结果

Jdbi 从 JDBC `ResultSet`中抽象出来，后者有一个相当麻烦的 API。

因此，它提供了几种可能性来访问由查询或其他返回结果的语句产生的列。我们现在来看看最简单的。

我们可以将每一行表示为一张地图:

```java
query.mapToMap();
```

映射的键将是选定的列名。

或者，当查询返回单个列时，我们可以将其映射到所需的 Java 类型:

```java
handle.createQuery("select name from project").mapTo(String.class);
```

Jdbi 内置了许多常见类的映射器。那些特定于某个图书馆或数据库系统的在单独的模块中提供。

当然，我们也可以定义和注册我们的映射器。我们将在后面的部分中讨论它。

最后，我们可以将行映射到 bean 或其他自定义类。同样，我们将在专门的部分看到更高级的选项。

### 5.3。迭代结果

一旦我们通过调用适当的方法**决定了如何映射结果，我们就会收到一个`ResultIterable` 对象。**

然后我们可以用它来迭代结果，一次一行。

在这里，我们将看看最常见的选项。

我们只能将结果累积在一个列表中:

```java
List<Map<String, Object>> results = query.mapToMap().list();
```

或者改成另一种`Collection` 类型:

```java
List<String> results = query.mapTo(String.class).collect(Collectors.toSet());
```

或者我们可以将结果作为一个流进行迭代:

```java
query.mapTo(String.class).useStream((Stream<String> stream) -> {
    doStuffWith(stream)
});
```

这里，为了清楚起见，我们显式地输入了`stream` 变量，但这不是必须的。

### 5.4。得到一个结果

作为特例，当我们期望或者只对一行感兴趣时，我们有两个专用的方法可用。

如果我们想要**最多一个结果**，我们可以使用`findFirst`:

```java
Optional<Map<String, Object>> first = query.mapToMap().findFirst();
```

正如我们所看到的，它返回一个`Optional` 值，只有当查询至少返回一个结果时才会出现。

如果查询返回多行，则只返回第一行。

相反，如果我们想要**且只有一个结果**，我们使用`findOnly`:

```java
Date onlyResult = query.mapTo(Date.class).findOnly();
```

最后，如果有零个或多个结果，`findOnly` 抛出一个`IllegalStateException`。

## 6。绑定参数

通常，**查询有一个固定部分和一个参数化部分。**这有几个优点，包括:

*   安全性:通过避免字符串连接，我们可以防止 SQL 注入
*   轻松:我们不需要记住时间戳等复杂数据类型的确切语法
*   性能:查询的静态部分可以被解析一次并缓存

Jdbi 支持位置参数和命名参数。

我们在查询或语句中插入位置参数作为问号:

```java
Query positionalParamsQuery =
  handle.createQuery("select * from project where name = ?");
```

相反，命名参数以冒号开头:

```java
Query namedParamsQuery =
  handle.createQuery("select * from project where url like :pattern");
```

在这两种情况下，为了设置参数值，我们使用了`bind`方法的一个变体:

```java
positionalParamsQuery.bind(0, "tutorials");
namedParamsQuery.bind("pattern", "%github.com/eugenp/%");
```

注意，与 JDBC 不同，索引从 0 开始。

### 6.1。一次绑定多个命名参数

我们还可以使用一个对象将多个命名参数绑定在一起。

假设我们有这样一个简单的查询:

```java
Query query = handle.createQuery(
  "select id from project where name = :name and url = :url");
Map<String, String> params = new HashMap<>();
params.put("name", "REST with Spring");
params.put("url", "github.com/eugenp/REST-With-Spring");
```

例如，我们可以使用地图:

```java
query.bindMap(params);
```

或者我们可以用不同的方式使用一个物体。例如，在这里，我们绑定一个遵循 JavaBean 约定的对象:

```java
query.bindBean(paramsBean);
```

但是我们也可以绑定一个对象的字段或方法；有关所有支持的选项，请参见 Jdbi 文档。

## 7。发布更复杂的语句

既然我们已经看到了查询、值和参数，我们可以回到语句并应用相同的知识。

回想一下，我们前面看到的`execute` 方法只是一个方便的快捷方式。

事实上，类似于查询， **DDL 和 DML 语句被表示为类`Update.`** 的实例

我们可以通过调用句柄上的方法`createUpdate`来获得一个:

```java
Update update = handle.createUpdate(
  "INSERT INTO PROJECT (NAME, URL) VALUES (:name, :url)");
```

然后，在一个`Update` 上，我们有了在一个`Query`中的所有绑定方法，所以第 6 节。也适用于更新。url

语句被执行时，我们调用，惊喜，`execute`:

```java
int rows = update.execute();
```

正如我们已经看到的，它返回受影响的行数。

### 7.1。提取自动递增的列值

作为一个特例，当我们有一个带有自动生成的列(通常是自动增量或序列)的 insert 语句时，我们可能希望获得生成的值。

然后，我们不叫`execute`，叫`executeAndReturnGeneratedKeys`:

```java
Update update = handle.createUpdate(
  "INSERT INTO PROJECT (NAME, URL) "
  + "VALUES ('tutorials', 'github.com/eugenp/tutorials')");
ResultBearing generatedKeys = update.executeAndReturnGeneratedKeys();
```

**`ResultBearing` 与我们之前看到的`Query` 类**实现的接口相同，所以我们已经知道如何使用它:

```java
generatedKeys.mapToMap()
  .findOnly().get("id");
```

## 8。交易

每当我们必须将多个语句作为单个原子操作来执行时，我们就需要一个事务。

与连接句柄一样，我们通过调用带有闭包的方法来引入事务:

```java
handle.useTransaction((Handle h) -> {
    haveFunWith(h);
});
```

和句柄一样，当闭包返回时，事务会自动关闭。

**然而，我们必须在返回之前提交或回滚事务**:

```java
handle.useTransaction((Handle h) -> {
    h.execute("...");
    h.commit();
});
```

但是，如果闭包抛出异常，Jdbi 会自动回滚事务。

和句柄一样，如果我们想从闭包返回一些东西，我们有一个专用的方法`inTransaction`:

```java
handle.inTransaction((Handle h) -> {
    h.execute("...");
    h.commit();
    return true;
});
```

### 8.1。人工交易管理

虽然在一般情况下不建议这样做，但我们也可以手动`begin` 和`close` 一个事务:

```java
handle.begin();
// ...
handle.commit();
handle.close();
```

## 9。结论和进一步阅读

在本教程中，我们已经介绍了 Jdbi 的核心:**查询、语句和事务。**

我们忽略了一些高级特性，比如定制的行列映射和批处理。

我们还没有讨论任何可选的模块，尤其是 SQL 对象扩展。

Jdbi 文档中详细介绍了所有内容。

所有这些例子和代码片段的实现都可以在 GitHub 项目中找到——这是一个 Maven 项目，所以应该很容易导入和运行。