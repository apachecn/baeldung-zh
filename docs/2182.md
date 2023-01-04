# 使用 JDBC 提取数据库元数据

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/jdbc-database-metadata>

## 1.概观

[JDBC](/web/20221208143841/https://www.baeldung.com/java-jdbc) 提供了一个 Java API 来读取存储在数据库表中的实际数据。除此之外，同样的 API 也可以用来读取数据库的元数据。元数据是指关于数据的数据，例如表名、列名和列类型。

在本教程中，我们将学习如何使用`DatabaseMetaData`接口提取不同类型的元数据。

## 2.`DatabaseMetaData`界面

[`DatabaseMetaData`](https://web.archive.org/web/20221208143841/https://docs.oracle.com/en/java/javase/11/docs/api/java.sql/java/sql/DatabaseMetaData.html) 是一个提供多种方法获取数据库综合信息的界面。**这些信息对于创建允许用户探索不同数据库结构的数据库工具非常有用。**当我们想检查底层数据库是否支持某些特性时，这也很有帮助。

我们需要一个`DatabaseMetaData`的实例来获取这些信息。因此，让我们看看如何通过代码从一个`Connection`对象中获得它:

```
DatabaseMetaData databaseMetaData = connection.getMetaData();
```

这里，`connection`是 `JdbcConnection`的一个实例。因此，`getMetaData()`方法返回一个`JdbcDatabaseMetaData`对象，该对象实现了`DatabaseMetaData` 接口。

在接下来的几节中，我们将使用这个对象来获取不同类型的元数据。之后，我们还将学习如何检查数据库是否支持特定的特性。

## 3.表元数据

有时，我们想知道所有用户定义的表、系统表或视图的名称。此外，我们可能想知道一些关于表格的解释性说明。所有这些都可以通过使用`DatabaseMetaData`对象的`getTables()`方法来完成。

首先，让我们看看如何提取所有现有用户定义表的名称:

```
try(ResultSet resultSet = databaseMetaData.getTables(null, null, null, new String[]{"TABLE"})){ 
  while(resultSet.next()) { 
    String tableName = resultSet.getString("TABLE_NAME"); 
    String remarks = resultSet.getString("REMARKS"); 
  }
}
```

这里，前两个参数是`catalog`和`schema`。第三个参数采用表名模式。例如，如果我们提供“CUST%”，这将包括所有名称以“CUST”开头的表。最后一个参数接受一个包含表类型的`String`数组。**用户自定义表格使用`TABLE`。**

接下来，如果我们想要寻找系统定义的表，我们所要做的就是用“`SYSTEM TABLE`”替换表类型:

```
try(ResultSet resultSet = databaseMetaData.getTables(null, null, null, new String[]{"SYSTEM TABLE"})){
 while(resultSet.next()) { 
    String systemTableName = resultSet.getString("TABLE_NAME"); 
 }
}
```

最后，为了找出所有现有的视图，我们只需将类型从**更改为“`VIEW`** ”。

## 4.列元数据

我们还可以使用同一个`DatabaseMetaData` 对象`.`来提取特定表格的列，让我们来看看实际情况:

```
try(ResultSet columns = databaseMetaData.getColumns(null,null, "CUSTOMER_ADDRESS", null)){
  while(columns.next()) {
    String columnName = columns.getString("COLUMN_NAME");
    String columnSize = columns.getString("COLUMN_SIZE");
    String datatype = columns.getString("DATA_TYPE");
    String isNullable = columns.getString("IS_NULLABLE");
    String isAutoIncrement = columns.getString("IS_AUTOINCREMENT");
  }
}
```

这里，`getColumns()`调用返回一个`ResultSet` ,我们可以迭代它来找到每一列的描述。每个描述包含许多有用的列，如`COLUMN_NAME`、`COLUMN_SIZE`和`DATA_TYPE`。

除了常规列之外，我们还可以找出特定表的主键列:

```
try(ResultSet primaryKeys = databaseMetaData.getPrimaryKeys(null, null, "CUSTOMER_ADDRESS")){ 
 while(primaryKeys.next()){ 
    String primaryKeyColumnName = primaryKeys.getString("COLUMN_NAME"); 
    String primaryKeyName = primaryKeys.getString("PK_NAME"); 
 }
}
```

类似地，我们可以检索外键列的描述以及给定表引用的主键列。让我们看一个例子:

```
try(ResultSet foreignKeys = databaseMetaData.getImportedKeys(null, null, "CUSTOMER_ADDRESS")){
 while(foreignKeys.next()){
    String pkTableName = foreignKeys.getString("PKTABLE_NAME");
    String fkTableName = foreignKeys.getString("FKTABLE_NAME");
    String pkColumnName = foreignKeys.getString("PKCOLUMN_NAME");
    String fkColumnName = foreignKeys.getString("FKCOLUMN_NAME");
 }
}
```

这里，`CUSTOMER_ADDRESS`表有一个外键列`CUST_ID`，它引用了`CUSTOMER`表的`ID`列。上面的代码片段将生成“CUSTOMER”作为主表，并将“CUSTOMER_ADDRESS”作为外部表。

在下一节中，我们将看到如何获取关于用户名和可用模式名的信息。

## 5.用户名和模式元数据

我们还可以获得在获取数据库连接时使用了其凭证的用户的名称:

```
String userName = databaseMetaData.getUserName();
```

类似地，**我们可以使用方法`getSchemas()`来检索数据库中可用模式**的名称:

```
try(ResultSet schemas = databaseMetaData.getSchemas()){
 while (schemas.next()){
    String table_schem = schemas.getString("TABLE_SCHEM");
    String table_catalog = schemas.getString("TABLE_CATALOG");
 }
}
```

在下一节中，我们将看到如何获取一些关于数据库的其他有用信息。

## 6.数据库级元数据

现在，让我们看看如何使用同一个`DatabaseMetaData` 对象获得数据库级别的信息。

例如，我们可以获取数据库产品的名称和版本、JDBC 驱动程序的名称、JDBC 驱动程序的版本号等等。现在让我们看看代码片段:

```
String productName = databaseMetaData.getDatabaseProductName();
String productVersion = databaseMetaData.getDatabaseProductVersion();
String driverName = databaseMetaData.getDriverName();
String driverVersion = databaseMetaData.getDriverVersion();
```

了解这些信息有时会很有用，尤其是当应用程序针对多个数据库产品和版本运行时。例如，某个版本或产品可能缺少某个特定的特性，或者包含一个应用程序需要实现某种解决方法的 bug。

接下来，我们将了解如何知道数据库是否缺少或支持某个特定的特性。

## 7.支持的数据库功能元数据

不同的数据库支持不同的特性。例如，H2 不支持完全外连接，而 MySQL 支持。

那么，我们如何才能知道我们正在使用的数据库是否支持某种特性呢？让我们看一些例子:

```
boolean supportsFullOuterJoins = databaseMetaData.supportsFullOuterJoins();
boolean supportsStoredProcedures = databaseMetaData.supportsStoredProcedures();
boolean supportsTransactions = databaseMetaData.supportsTransactions();
boolean supportsBatchUpdates = databaseMetaData.supportsBatchUpdates();
```

同样，可以查询的特性的完整列表可以在官方 Java 文档中找到。

## 8.结论

在本文中，我们学习了如何使用`DatabaseMetaData`接口来检索元数据和数据库支持的特性。

该项目的完整源代码，包括这里使用的所有代码样本，可以在 GitHub 上找到[。](https://web.archive.org/web/20221208143841/https://github.com/eugenp/tutorials/tree/master/persistence-modules/core-java-persistence)