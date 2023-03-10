# DBUnit 简介

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-dbunit>

## 1.介绍

在本教程中，我们将看看 DBUnit，一个用于在 **Java 中**测试** **关系数据库交互**的单元测试工具。**

我们将看到它如何帮助我们让我们的数据库进入一个已知的状态，并断言一个预期的状态。

## 2.属国

首先，我们可以通过将`dbunit` 依赖项添加到我们的`pom.xml`来将 DBUnit 从 Maven Central 添加到我们的项目中:

```java
<dependency>
  <groupId>org.dbunit</groupId>
  <artifactId>dbunit</artifactId>
  <version>2.7.0</version>
  <scope>test</scope>
</dependency>
```

我们可以在 [Maven Central](https://web.archive.org/web/20220627084342/https://mvnrepository.com/artifact/org.dbunit/dbunit) 上查找最新版本。

## 3.Hello World 示例

接下来，让我们定义一个**数据库模式:**

`schema.sql`:

```java
CREATE TABLE IF NOT EXISTS CLIENTS
(
    `id`         int AUTO_INCREMENT NOT NULL,
    `first_name` varchar(100)       NOT NULL,
    `last_name`  varchar(100)       NOT NULL,
    PRIMARY KEY (`id`)
);

CREATE TABLE IF NOT EXISTS ITEMS
(
    `id`       int AUTO_INCREMENT NOT NULL,
    `title`    varchar(100)       NOT NULL,
    `produced` date,
    `price`    float,
    PRIMARY KEY (`id`)
); 
```

### 3.1.定义初始数据库内容

**DBUnit 让我们以简单的** **声明方式**定义并加载我们的测试数据集。

我们用一个 XML 元素定义每个表行，其中标记名是表名，属性名和值分别映射到列名和值。可以为多个表创建行数据。我们必须实现`DataSourceBasedDBTestCase`的`getDataSet()`方法来定义初始数据集，在这里我们可以使用`FlatXmlDataSetBuilder`来引用我们的 XML 文件:

`data.xml`:

```java
<?xml version="1.0" encoding="UTF-8"?>
<dataset>
    <CLIENTS id='1' first_name='Charles' last_name='Xavier'/>
    <ITEMS id='1' title='Grey T-Shirt' price='17.99' produced='2019-03-20'/>
    <ITEMS id='2' title='Fitted Hat' price='29.99' produced='2019-03-21'/>
    <ITEMS id='3' title='Backpack' price='54.99' produced='2019-03-22'/>
    <ITEMS id='4' title='Earrings' price='14.99' produced='2019-03-23'/>
    <ITEMS id='5' title='Socks' price='9.99'/>
</dataset>
```

### 3.2.初始化数据库连接和模式

现在我们已经有了我们的模式，我们必须初始化我们的数据库。

我们必须扩展`DataSourceBasedDBTestCase`类，并在其`getDataSource()`方法中初始化数据库模式:

`DataSourceDBUnitTest.java`:

```java
public class DataSourceDBUnitTest extends DataSourceBasedDBTestCase {
    @Override
    protected DataSource getDataSource() {
        JdbcDataSource dataSource = new JdbcDataSource();
        dataSource.setURL(
          "jdbc:h2:mem:default;DB_CLOSE_DELAY=-1;init=runscript from 'classpath:schema.sql'");
        dataSource.setUser("sa");
        dataSource.setPassword("sa");
        return dataSource;
    }

    @Override
    protected IDataSet getDataSet() throws Exception {
        return new FlatXmlDataSetBuilder().build(getClass().getClassLoader()
          .getResourceAsStream("data.xml"));
    }
}
```

这里，我们通过连接字符串将一个 SQL 文件传递给了 H2 内存数据库。如果我们想在其他数据库上测试，我们需要为它提供我们的自定义实现。

请记住，在我们的示例中的**，**， **DBUnit 将在每个测试方法执行**之前用给定的测试数据重新初始化数据库。

通过`get` `SetUpOperation `和`get` `TearDownOperation`有多种方式进行配置:

```java
@Override
protected DatabaseOperation getSetUpOperation() {
    return DatabaseOperation.REFRESH;
}

@Override
protected DatabaseOperation getTearDownOperation() {
    return DatabaseOperation.DELETE_ALL;
}
```

`REFRESH`操作告诉 DBUnit 刷新它的所有数据。这将确保所有缓存都被清除，并且我们的单元测试不会受到另一个单元测试的影响。`DELETE_ALL`操作确保所有数据在每个单元测试结束时都被删除。在我们的例子中，我们告诉 DBUnit 在设置期间，使用`getSetUpOperation`方法实现我们将刷新所有缓存。最后，我们使用`getTearDownOperation `方法实现告诉 DBUnit 在拆卸操作期间删除所有数据。

### 3.3.比较预期状态和实际状态

现在，让我们检查我们的实际测试用例。对于第一个测试，我们将保持简单——我们将加载预期的数据集，并将其与从数据库连接中检索的数据集进行比较:

```java
@Test
public void givenDataSetEmptySchema_whenDataSetCreated_thenTablesAreEqual() throws Exception {
    IDataSet expectedDataSet = getDataSet();
    ITable expectedTable = expectedDataSet.getTable("CLIENTS");
    IDataSet databaseDataSet = getConnection().createDataSet();
    ITable actualTable = databaseDataSet.getTable("CLIENTS");
    assertEquals(expectedTable, actualTable);
}
```

## 4.深入探究`Assertions`

在上一节中，我们看到了一个比较表的实际内容和预期数据集的基本示例。现在我们将发现 DBUnit 对定制数据断言的支持。

### 4.1.使用 SQL 查询断言

检查实际状态的一种简单方法是使用 SQL 查询。

在本例中，我们将向 CLIENTS 表中插入一条新记录，然后验证新创建的行的内容。我们在单独的 XML 文件中定义了**的预期输出，并通过 SQL 查询提取了实际的行值:**

```java
@Test
public void givenDataSet_whenInsert_thenTableHasNewClient() throws Exception {
    try (InputStream is = getClass().getClassLoader().getResourceAsStream("dbunit/expected-user.xml")) {
        IDataSet expectedDataSet = new FlatXmlDataSetBuilder().build(is);
        ITable expectedTable = expectedDataSet.getTable("CLIENTS");
        Connection conn = getDataSource().getConnection();

        conn.createStatement()
            .executeUpdate(
            "INSERT INTO CLIENTS (first_name, last_name) VALUES ('John', 'Jansen')");
        ITable actualData = getConnection()
            .createQueryTable(
                "result_name",
                "SELECT * FROM CLIENTS WHERE last_name='Jansen'");

        assertEqualsIgnoreCols(expectedTable, actualData, new String[] { "id" });
    }
}
```

`DBTestCase`祖先类的`getConnection()`方法返回数据源连接的特定于 DBUnit 的表示(一个`IDatabaseConnection`实例)。**`IDatabaseConnection`的`createQueryTable()`方法可用于从数据库**中获取实际数据，以便使用`Assertion.assertEquals()`方法与预期的数据库状态进行比较。传递给`createQueryTable()`的 SQL 查询是我们想要测试的查询。它返回一个我们用来断言的`Table`实例。

### 4.2.忽略列

**有时在数据库测试中，我们希望忽略实际表格**中的一些列。这些通常是自动生成的值，我们不能严格控制，像**生成的主键或当前时间戳**。

**我们可以通过从 SQL 查询的 SELECT 子句中省略列**来做到这一点，但是 DBUnit 提供了一个更方便的实用程序来实现这一点。**使用`DefaultColumnFilter`类的静态方法，我们可以通过排除一些列**从现有实例创建一个新的`ITable`实例，如下所示:

```java
@Test
public void givenDataSet_whenInsert_thenGetResultsAreStillEqualIfIgnoringColumnsWithDifferentProduced()
  throws Exception {
    Connection connection = tester.getConnection().getConnection();
    String[] excludedColumns = { "id", "produced" };
    try (InputStream is = getClass().getClassLoader()
      .getResourceAsStream("dbunit/expected-ignoring-registered_at.xml")) {
        IDataSet expectedDataSet = new FlatXmlDataSetBuilder().build(is);
        ITable expectedTable = excludedColumnsTable(expectedDataSet.getTable("ITEMS"), excludedColumns);

        connection.createStatement()
          .executeUpdate("INSERT INTO ITEMS (title, price, produced)  VALUES('Necklace', 199.99, now())");

        IDataSet databaseDataSet = tester.getConnection().createDataSet();
        ITable actualTable = excludedColumnsTable(databaseDataSet.getTable("ITEMS"), excludedColumns);

        assertEquals(expectedTable, actualTable);
    }
}
```

### 4.3.调查多重故障

如果 DBUnit **发现一个不正确的值，那么它会立即抛出一个`AssertionError`。**

在特定情况下，我们可以使用`DiffCollectingFailureHandler`类，我们可以将它作为第三个参数传递给`Assertion.assertEquals()`方法。

这个失败处理程序将收集所有的失败，而不是在第一次失败时停止，这意味着如果我们使用 `**DiffCollectingFailureHandler**.` ，那么**`Assertion.assertEquals()`方法将总是成功的。因此，我们将必须以编程方式检查处理程序是否发现任何错误:**

```java
@Test
public void givenDataSet_whenInsertUnexpectedData_thenFailOnAllUnexpectedValues() throws Exception {
    try (InputStream is = getClass().getClassLoader()
      .getResourceAsStream("dbunit/expected-multiple-failures.xml")) {
        IDataSet expectedDataSet = new FlatXmlDataSetBuilder().build(is);
        ITable expectedTable = expectedDataSet.getTable("ITEMS");
        Connection conn = getDataSource().getConnection();
        DiffCollectingFailureHandler collectingHandler = new DiffCollectingFailureHandler();

        conn.createStatement()
          .executeUpdate("INSERT INTO ITEMS (title, price) VALUES ('Battery', '1000000')");
        ITable actualData = getConnection().createDataSet().getTable("ITEMS");

        assertEquals(expectedTable, actualData, collectingHandler);
        if (!collectingHandler.getDiffList().isEmpty()) {
            String message = (String) collectingHandler.getDiffList()
                .stream()
                .map(d -> formatDifference((Difference) d))
                .collect(joining("\n"));
            logger.error(() -> message);
        }
    }
}

private static String formatDifference(Difference diff) {
    return "expected value in " + diff.getExpectedTable()
      .getTableMetaData()
      .getTableName() + "." + 
      diff.getColumnName() + " row " + 
      diff.getRowIndex() + ":" + 
      diff.getExpectedValue() + ", but was: " + 
      diff.getActualValue();
}
```

此外，处理程序以`Difference`实例的形式提供失败，这让我们可以格式化错误。

运行测试后，我们得到一个格式化的报告:

```java
java.lang.AssertionError: expected value in ITEMS.price row 5:199.99, but was: 1000000.0
expected value in ITEMS.produced row 5:2019-03-23, but was: null
expected value in ITEMS.title row 5:Necklace, but was: Battery

	at com.baeldung.dbunit.DataSourceDBUnitTest.givenDataSet_whenInsertUnexpectedData_thenFailOnAllUnexpectedValues(DataSourceDBUnitTest.java:91)
```

需要注意的是，在这一点上，我们预计新商品的价格为 199.99 美元，但实际价格是 1000000.0 美元。然后我们看到生产日期是 2019-03-23，但最后，它是空的。最后，我们期待的东西是一条项链，但却得到了一块电池。

## 5.结论

在本文中，我们看到了 DBUnit 如何提供一种定义测试数据的**声明方式来**测试 Java 应用程序的数据访问层。****

与往常一样，GitHub 上的[提供了示例的完整源代码。](https://web.archive.org/web/20220627084342/https://github.com/eugenp/tutorials/tree/master/libraries-testing)