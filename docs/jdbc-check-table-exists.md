# 如何检查 JDBC 数据库表是否存在

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/jdbc-check-table-exists>

## 1.介绍

在本教程中，我们将了解如何使用 JDBC 和纯 SQL 来检查数据库中是否存在表。

## 2.使用`DatabaseMetaData`

[JDBC](/web/20220626115117/https://www.baeldung.com/java-jdbc) 为我们提供了读写数据库数据的工具。除了存储在表中的实际数据，我们还可以读取描述数据库的元数据。为此，我们将使用可以从 JDBC 连接中获得的 [`DatabaseMetaData`](/web/20220626115117/https://www.baeldung.com/jdbc-database-metadata) 对象:

```java
DatabaseMetaData databaseMetaData = connection.getMetaData();
```

`DatabaseMetaData`提供了很多有用的方法，但是我们只需要一个:`getTables`。**让我们用它来打印所有可用的表格:**

```java
ResultSet resultSet = databaseMetaData.getTables(null, null, null, new String[] {"TABLE"});

while (resultSet.next()) {
    String name = resultSet.getString("TABLE_NAME");
    String schema = resultSet.getString("TABLE_SCHEM");
    System.out.println(name + " on schema " + schema);
}
```

因为我们没有提供前三个参数，所以我们获得了所有目录和模式中的所有表。例如，我们还可以将查询范围缩小到只有一个模式:

```java
ResultSet resultSet = databaseMetaData.getTables(null, "PUBLIC", null, new String[] {"TABLE"});
```

## 3.使用`DatabaseMetaData`检查表格是否存在

如果我们想检查一个表是否存在，我们不需要迭代结果集。我们只需要检查结果集是否不为空。让我们首先创建一个“雇员”表:

```java
connection.createStatement().executeUpdate("create table EMPLOYEE (id int primary key auto_increment, name VARCHAR(255))");
```

现在，我们可以使用元数据对象来断言我们刚刚创建的表确实存在:

```java
boolean tableExists(Connection connection, String tableName) throws SQLException {
    DatabaseMetaData meta = connection.getMetaData();
    ResultSet resultSet = meta.getTables(null, null, tableName, new String[] {"TABLE"});

    return resultSet.next();
}
```

请注意，虽然 SQL 不区分大小写，但是`getTables`方法的实现是区分大小写的。即使我们用小写字母定义一个表，它也会以大写字母存储。正因为如此，`getTables`方法将对大写的表名进行操作，所以我们需要使用“employee”而不是“EMPLOYEE”。

## 4.用 SQL 检查表是否存在

虽然`DatabaseMetaData `很方便，但我们可能需要使用纯 SQL 来实现同样的目标。**为此，我们需要查看位于模式“`information_schema`”中的“`tables`”表。**它是 [SQL-92](https://web.archive.org/web/20220626115117/https://en.wikipedia.org/wiki/SQL-92) 标准的一部分，由大多数主流数据库引擎实现(Oracle 是个明显的例外)。

让我们查询“`tables`”表，并统计提取了多少个结果。如果表存在，我们期望得到 1，如果不存在，我们期望得到 0:

```java
SELECT count(*) FROM information_schema.tables
WHERE table_name = 'EMPLOYEE' 
LIMIT 1;
```

将它与 JDBC 一起使用就是创建一个简单的[预准备语句](/web/20220626115117/https://www.baeldung.com/java-statement-preparedstatement)，然后检查结果计数是否不等于零:

```java
static boolean tableExistsSQL(Connection connection, String tableName) throws SQLException {
    PreparedStatement preparedStatement = connection.prepareStatement("SELECT count(*) "
      + "FROM information_schema.tables "
      + "WHERE table_name = ?"
      + "LIMIT 1;");
    preparedStatement.setString(1, tableName);

    ResultSet resultSet = preparedStatement.executeQuery();
    resultSet.next();
    return resultSet.getInt(1) != 0;
}
```

## 5.结论

在本教程中，我们学习了如何在数据库中查找关于表存在的信息。我们同时使用了 JDBC 的`DatabaseMetaData`和纯 SQL。

像往常一样，GitHub 上的所有代码示例[都可用。](https://web.archive.org/web/20220626115117/https://github.com/eugenp/tutorials/tree/master/persistence-modules/core-java-persistence-2)