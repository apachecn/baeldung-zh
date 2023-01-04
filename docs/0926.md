# JDBC 结果集界面指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/jdbc-resultset>

## 1.概观

[Java 数据库连接(JDBC) API](/web/20220627092439/https://www.baeldung.com/java-jdbc) 提供了从 Java 应用程序对数据库的访问。我们可以使用 JDBC 连接到任何数据库，只要支持的 JDBC 驱动程序可用。

**`ResultSet`是执行数据库查询生成的数据表。**在本教程中，我们将深入了解一下 [`ResultSet` API](https://web.archive.org/web/20220627092439/https://docs.oracle.com/en/java/javase/11/docs/api/java.sql/java/sql/ResultSet.html) 。

## 2.生成一个`ResultSet`

首先，我们通过在任何实现了`Statement`接口的对象上调用`executeQuery()`来检索一个`ResultSet`。`PreparedStatement`和`CallableStatement`都是`Statement`的子接口:

```
PreparedStatement pstmt = dbConnection.prepareStatement("select * from employees");
ResultSet rs = pstmt.executeQuery();
```

`ResultSet`对象维护一个指向结果集当前行的光标。我们将在我们的`ResultSet`上使用`next()`来遍历记录。

接下来，我们将**使用`getX()`方法，同时遍历结果，从数据库列**中获取值，其中`X`是列的数据类型。事实上，我们将为`getX()`方法提供数据库列名:

```
while(rs.next()) {
    String name = rs.getString("name");
    Integer empId = rs.getInt("emp_id");
    Double salary = rs.getDouble("salary");
    String position = rs.getString("position");
} 
```

同样，**列的索引号可以与`getX()`方法**一起使用，而不是列名。索引号是 SQL select 语句中的列序列。

如果 select 语句没有列出列名，则索引号是表中列的顺序。列索引编号从 1 开始:

```
Integer empId = rs.getInt(1);
String name = rs.getString(2);
String position = rs.getString(3);
Double salary = rs.getDouble(4); 
```

## 3.从`ResultSet`中检索元数据

在这一节中，我们将看到如何在`ResultSet`中检索关于列属性和类型的信息。

首先，让我们使用`ResultSet`上的`getMetaData()`方法来获得 [`ResultSetMetaData`](https://web.archive.org/web/20220627092439/https://docs.oracle.com/en/java/javase/11/docs/api/java.sql/java/sql/ResultSetMetaData.html) :

```
ResultSetMetaData metaData = rs.getMetaData();
```

接下来，让我们获取`ResultSet`中的列数:

```
Integer columnCount = metaData.getColumnCount();
```

此外，我们可以在元数据对象上使用以下任何方法来检索每一列的属性:

*   `getColumnName(int columnNumber)` `–`获取列的名称
*   `getColumnLabel(int columnNumber)` `–`访问列的标签，该标签在 SQL 查询中的`AS`之后指定
*   `getTableName(int columnNumber)` `–`获取该列所属的表名
*   `getColumnClassName(int columnNumber)` `–`获取该列的 Java 数据类型
*   `getColumnTypeName(int columnNumber)` `–`获取数据库中该列的数据类型
*   `getColumnType(int columnNumber)` `–`获取该列的 SQL 数据类型
*   `isAutoIncrement(int columnNumber)` `–`表示该列是否自动递增
*   `isCaseSensitive(int columnNumber)` `–`指定列大小写是否重要
*   `isSearchable(int columnNumber)` `–`建议我们是否可以使用 SQL 查询的`where`子句中的列
*   `isCurrency(int columnNumber)` `–`表示该列是否包含现金值
*   如果列不能为空，则`isNullable(int columnNumber)` `–`返回`zero`，如果列可以包含空值，则返回`one`，如果列的可空性未知，则返回`two`
*   如果列中的值有符号，则`isSigned(int columnNumber)` `–`返回`true`，否则返回`false`

让我们遍历这些列以获取它们的属性:

```
for (int columnNumber = 1; columnNumber <= columnCount; columnNumber++) {
    String catalogName = metaData.getCatalogName(columnNumber);
    String className = metaData.getColumnClassName(columnNumber);
    String label = metaData.getColumnLabel(columnNumber);
    String name = metaData.getColumnName(columnNumber);
    String typeName = metaData.getColumnTypeName(columnNumber);
    int type = metaData.getColumnType(columnNumber);
    String tableName = metaData.getTableName(columnNumber);
    String schemaName = metaData.getSchemaName(columnNumber);
    boolean isAutoIncrement = metaData.isAutoIncrement(columnNumber);
    boolean isCaseSensitive = metaData.isCaseSensitive(columnNumber);
    boolean isCurrency = metaData.isCurrency(columnNumber);
    boolean isDefiniteWritable = metaData.isDefinitelyWritable(columnNumber);
    boolean isReadOnly = metaData.isReadOnly(columnNumber);
    boolean isSearchable = metaData.isSearchable(columnNumber);
    boolean isReadable = metaData.isReadOnly(columnNumber);
    boolean isSigned = metaData.isSigned(columnNumber);
    boolean isWritable = metaData.isWritable(columnNumber);
    int nullable = metaData.isNullable(columnNumber);
}
```

## 4.导航`ResultSet`

当我们获得一个`ResultSet`时，光标的位置在第一行之前。此外，默认情况下，`ResultSet`仅向前移动。但是，我们可以使用可滚动的`ResultSet`用于其他导航选项。

在本节中，我们将讨论各种导航选项。

### 4.1.`ResultSet`类型

`ResultSet`类型表示我们将如何浏览数据集:

*   `TYPE_FORWARD_ONLY –`默认选项，光标从起点移动到终点
*   我们的光标可以在数据集中向前和向后移动；如果在数据集中移动时对基础数据进行了更改，则忽略这些更改；数据集包含数据库查询返回结果时的数据
*   `TYPE_SCROLL_SENSITIVE –` 类似于滚动不敏感类型，但是对于这种类型，数据集会立即反映底层数据的任何更改

并非所有的数据库都支持所有的`ResultSet`类型。因此，让我们通过在我们的`DatabaseMetaData`对象上使用`supportsResultSetType`来检查该类型是否受支持:

```
DatabaseMetaData dbmd = dbConnection.getMetaData();
boolean isSupported = dbmd.supportsResultSetType(ResultSet.TYPE_SCROLL_INSENSITIVE);
```

### 4.2.可滚动结果集

为了得到一个可滚动的`ResultSet`，我们需要**在准备`Statement`** 时传递一些额外的参数。

例如，我们将通过使用`TYPE_SCROLL_INSENSITIVE`或`TYPE_SCROLL_SENSITIVE`作为`ResultSet`类型来获得一个可滚动的`ResultSet`:

```
PreparedStatement pstmt = dbConnection.prepareStatement(
  "select * from employees",
  ResultSet.TYPE_SCROLL_INSENSITIVE,
  ResultSet.CONCUR_UPDATABLE); 
ResultSet rs = pstmt.executeQuery(); 
```

### 4.3.导航选项

我们可以在可滚动的`ResultSet`上使用以下任何选项:

*   `next()`–从当前位置前进到下一行
*   `previous()`–遍历到上一行
*   `first() –`导航到`ResultSet`的第一行
*   `last() –`跳到最后一行
*   `beforeFirst() –`移动到起点；在调用这个方法之后，在我们的`ResultSet`上调用`next()`会从我们的`ResultSet`返回第一行
*   `afterLast() –`跳跃到终点；在执行这个方法后调用`previous() on our ResultSet `会从我们的`ResultSet`返回最后一行
*   `relative(int numOfRows) –`通过`numOfRows`从当前位置前进或后退
*   `absolute(int rowNumber) –`跳到指定的`rowNumber`

让我们看一些例子:

```
PreparedStatement pstmt = dbConnection.prepareStatement(
  "select * from employees",
  ResultSet.TYPE_SCROLL_SENSITIVE,
  ResultSet.CONCUR_UPDATABLE);
ResultSet rs = pstmt.executeQuery();

while (rs.next()) {
    // iterate through the results from first to last
}
rs.beforeFirst(); // jumps back to the starting point, before the first row
rs.afterLast(); // jumps to the end of resultset

rs.first(); // navigates to the first row
rs.last(); // goes to the last row

rs.absolute(2); //jumps to 2nd row

rs.relative(-1); // jumps to the previous row
rs.relative(2); // jumps forward two rows

while (rs.previous()) {
    // iterates from current row to the first row in backward direction
} 
```

### 4.4.`ResultSet`行数

让我们使用`getRow()`来获取我们的`ResultSet`的当前行号。

首先，我们将导航到`ResultSet`的最后一行，然后使用`getRow()`获得记录数:

```
rs.last();
int rowCount = rs.getRow();
```

## 5.更新`ResultSet`中的数据

默认情况下，`ResultSet`是只读的。然而，我们可以使用可更新的`ResultSet`来插入、更新和删除行。

### 5.1.`ResultSet`并发

并发模式表明我们的`ResultSet`是否可以更新数据。

`CONCUR_READ_ONLY`选项是默认选项，如果我们不需要使用`ResultSet`更新数据，就应该使用这个选项。

然而，如果我们需要更新我们的`ResultSet`中的数据，那么应该使用`CONCUR_UPDATABLE`选项。

**并非所有数据库都支持所有`ResultSet`类型**的所有并发模式。因此，我们需要使用`supportsResultSetConcurrency()`方法检查我们想要的类型和并发模式是否受支持:

```
DatabaseMetaData dbmd = dbConnection.getMetaData(); 
boolean isSupported = dbmd.supportsResultSetConcurrency(
  ResultSet.TYPE_SCROLL_SENSITIVE, ResultSet.CONCUR_UPDATABLE); 
```

### 5.2.获取可更新的`ResultSet`

为了获得可更新的`ResultSet`，我们需要在准备`Statement`时传递一个额外的参数。为此，让我们在创建语句时使用`CONCUR_UPDATABLE`作为第三个参数:

```
PreparedStatement pstmt = dbConnection.prepareStatement(
  "select * from employees",
  ResultSet.TYPE_SCROLL_SENSITIVE,
  ResultSet.CONCUR_UPDATABLE);
ResultSet rs = pstmt.executeQuery();
```

### 5.3.更新行

在这一节中，我们将使用上一节中创建的可更新的`ResultSet` 来更新一行。

我们可以通过调用`updateX()`方法，传递要更新的列名和值来更新一行中的数据。我们可以使用任何支持的数据类型来代替`updateX()`方法中的`X`。

让我们更新类型为`double`的`“salary”`列:

```
rs.updateDouble("salary", 1100.0);
```

注意，这只是更新了`ResultSet`中的数据，但是修改还没有保存回数据库。

最后，让我们调用`updateRow()`来**保存对数据库**的更新:

```
rs.updateRow(); 
```

我们可以将列索引传递给`updateX()`方法，而不是列名。这类似于使用列索引通过`getX()`方法获取值。将列名或索引传递给`updateX()`方法会产生相同的结果:

```
rs.updateDouble(4, 1100.0);
rs.updateRow(); 
```

### 5.4.插入一行

现在，让我们使用可更新的`ResultSet`插入一个新行。

首先，我们将使用`moveToInsertRow()`移动光标来插入一个新行:

```
rs.moveToInsertRow();
```

接下来，我们必须调用`updateX()`方法将信息添加到行中。我们需要为数据库表中的所有列提供数据。如果我们不为每一列提供数据，则使用默认的列值:

```
rs.updateString("name", "Venkat"); 
rs.updateString("position", "DBA"); 
rs.updateDouble("salary", 925.0);
```

然后，让我们调用`insertRow()`向数据库中插入一个新行:

```
rs.insertRow();
```

最后，让我们使用`moveToCurrentRow().`这将把光标位置带回到我们开始使用`moveToInsertRow()`方法插入新行之前所在的行:

```
rs.moveToCurrentRow();
```

### 5.5.删除一行

在本节中，我们将使用可更新的`ResultSet`删除一行。

首先，我们将导航到要删除的行。然后，我们将调用`deleteRow() `方法来删除当前行:

```
rs.absolute(2);
rs.deleteRow();
```

## 6.保持能力

可持有性决定了我们的`ResultSet`在数据库事务结束时是打开还是关闭。

### 6.1。可持有性类型

如果事务提交后不需要`ResultSet`，则使用`CLOSE_CURSORS_AT_COMMIT`。

使用`HOLD_CURSORS_OVER_COMMIT`创建一个可保持的`ResultSet`。即使提交了数据库事务，可保持的`ResultSet`也不会关闭。

并非所有的数据库都支持所有的可持有性类型。

所以，让我们用`DatabaseMetaData`对象上的`supportsResultSetHoldability()`来检查是否支持可持有性类型。然后，我们将使用`getResultSetHoldability()`获得数据库的默认可持有性:

```
boolean isCloseCursorSupported
  = dbmd.supportsResultSetHoldability(ResultSet.CLOSE_CURSORS_AT_COMMIT);
boolean isOpenCursorSupported
  = dbmd.supportsResultSetHoldability(ResultSet.HOLD_CURSORS_OVER_COMMIT);
boolean defaultHoldability
  = dbmd.getResultSetHoldability();
```

### 6.2.可持有的`ResultSet`

为了创建一个可持有的`ResultSet`，我们需要在创建一个`Statement`时将`holdability`类型指定为最后一个参数。此参数在并发模式后指定。

注意，如果我们使用 Microsoft SQL Server (MSSQL)，我们必须在数据库连接上设置可保持性，而不是在`ResultSet`上:

```
dbConnection.setHoldability(ResultSet.HOLD_CURSORS_OVER_COMMIT);
```

让我们来看看实际情况。首先，让我们创建一个`Statement`，将可持有性设置为`HOLD_CURSORS_OVER_COMMIT`:

```
Statement pstmt = dbConnection.createStatement(
  ResultSet.TYPE_SCROLL_SENSITIVE, 
  ResultSet.CONCUR_UPDATABLE, 
  ResultSet.HOLD_CURSORS_OVER_COMMIT)
```

现在，让我们在检索数据时更新一行。这类似于我们之前讨论的更新示例，除了在将更新事务提交给数据库之后，我们将继续遍历`ResultSet`。这在 MySQL 和 MSSQL 数据库上都能很好地工作:

```
dbConnection.setAutoCommit(false);
ResultSet rs = pstmt.executeQuery("select * from employees");
while (rs.next()) {
    if(rs.getString("name").equalsIgnoreCase("john")) {
        rs.updateString("name", "John Doe");
        rs.updateRow();
        dbConnection.commit();
    }                
}
rs.last(); 
```

值得注意的是，MySQL 只支持`HOLD_CURSORS_OVER_COMMIT`。所以，即使我们使用`CLOSE_CURSORS_AT_COMMIT`，它也会被忽略。

MSSQL 数据库支持`CLOSE_CURSORS_AT_COMMIT`。这意味着当我们提交事务时,`ResultSet`将被关闭。因此，在提交事务后尝试访问`ResultSet`会导致“游标未打开错误”。因此，我们无法从`ResultSet`中检索进一步的记录。

## 7.获取大小

通常，当将数据加载到`ResultSet`中时，数据库驱动程序决定从数据库中获取的行数。例如，在 MySQL 数据库上，`ResultSet`通常会将所有记录一次加载到内存中。

然而，有时我们可能需要处理大量的记录，这些记录不适合我们的 JVM 内存。在这种情况下，我们可以在我们的`Statement`或`ResultSet`对象上使用 fetch size 属性来限制最初返回的记录数。

每当需要额外的结果时，`ResultSet`从数据库中获取另一批记录。使用 fetch size 属性，我们可以**向数据库驱动程序提供每次数据库访问要获取的行数的建议**。我们指定的获取大小将应用于后续的数据库行程。

如果我们没有为我们的`ResultSet`指定获取大小，那么就使用`Statement`的获取大小。如果我们没有为`Statement`或`ResultSet`指定获取大小，那么使用数据库默认值。

### 7.1.在`Statement`上使用提取大小

现在，让我们看看`Statement`上的读取大小。我们将把`Statement`的获取大小设置为 10 条记录。如果我们的查询返回 100 条记录，那么将有 10 次数据库往返，每次加载 10 条记录:

```
PreparedStatement pstmt = dbConnection.prepareStatement(
  "select * from employees", 
  ResultSet.TYPE_SCROLL_SENSITIVE, 
  ResultSet.CONCUR_READ_ONLY);
pstmt.setFetchSize(10);

ResultSet rs = pstmt.executeQuery();

while (rs.next()) {
    // iterate through the resultset
}
```

### 7.2.在`ResultSet`上使用提取大小

现在，让我们使用`ResultSet`来改变前面例子中的获取大小。

首先，我们将在我们的`Statement`上使用获取大小。这允许我们的`ResultSet`在执行查询后最初加载 10 条记录。

然后，我们将修改`ResultSet`上的获取大小。这将覆盖我们之前在`Statement`中指定的获取大小。因此，所有后续行程将加载 20 条记录，直到加载完所有记录。

因此，加载所有记录只需要 6 次数据库旅行:

```
PreparedStatement pstmt = dbConnection.prepareStatement(
  "select * from employees", 
  ResultSet.TYPE_SCROLL_SENSITIVE, 
  ResultSet.CONCUR_READ_ONLY);
pstmt.setFetchSize(10);

ResultSet rs = pstmt.executeQuery();

rs.setFetchSize(20); 

while (rs.next()) { 
    // iterate through the resultset 
}
```

最后，我们将看到如何在迭代结果时修改`ResultSet`的获取大小。

类似于前面的例子，我们将首先在我们的`Statement`上将获取大小设置为 10。因此，我们的前 3 次数据库访问每次将加载 10 条记录。

然后，在读取第 30 条记录时，我们将把我们的`ResultSet`上的读取大小修改为 20。因此，接下来的 4 次旅行每次将加载 20 条记录。

因此，我们需要 7 次数据库访问来加载所有 100 条记录:

```
PreparedStatement pstmt = dbConnection.prepareStatement(
  "select * from employees", 
  ResultSet.TYPE_SCROLL_SENSITIVE, 
  ResultSet.CONCUR_READ_ONLY);
pstmt.setFetchSize(10);

ResultSet rs = pstmt.executeQuery();

int rowCount = 0;

while (rs.next()) { 
    // iterate through the resultset 
    if (rowCount == 30) {
        rs.setFetchSize(20); 
    }
    rowCount++;
}
```

## 8.结论

在本文中，我们看到了如何使用`ResultSet` API 从数据库中检索和更新数据。我们讨论的一些高级特性依赖于我们使用的数据库。因此，我们需要在使用它们之前检查对这些特性的支持。

和往常一样，代码可以在 GitHub 的[上获得。](https://web.archive.org/web/20220627092439/https://github.com/eugenp/tutorials/tree/master/persistence-modules/core-java-persistence)