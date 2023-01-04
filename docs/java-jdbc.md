# JDBC 简介

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-jdbc>

## 1。概述

在本文中，我们将了解 JDBC (Java 数据库连接),这是一个用于连接和执行数据库查询的 API。

只要提供合适的驱动程序，JDBC 可以使用任何数据库。

## 2。JDBC 车手

JDBC 驱动程序是一种 JDBC API 实现，用于连接特定类型的数据库。有几种类型的 JDBC 驱动程序:

*   类型 1–包含到另一个数据访问 API 的映射；JDBC-ODBC 驱动程序就是一个例子
*   类型 2–是使用目标数据库的客户端库的实现；也称为本机 API 驱动程序
*   第 3 类——使用中间件将 JDBC 调用转换为特定于数据库的调用；也称为网络协议驱动程序
*   类型 4–通过将 JDBC 调用转换为特定于数据库的调用来直接连接到数据库；称为数据库协议驱动程序或瘦驱动程序，

最常用的类型是 type 4，因为它具有独立于平台的优势。与其他类型相比，直接连接到数据库服务器可以提供更好的性能。这种驱动程序的缺点是它是特定于数据库的，因为每个数据库都有自己特定的协议。

## 3。连接到数据库

要连接到数据库，我们只需初始化驱动程序并打开一个数据库连接。

### 3.1。注册驱动程序

对于我们的例子，我们将使用类型 4 数据库协议驱动程序。

由于我们使用的是 MySQL 数据库，我们需要 [`mysql-connector-java`](https://web.archive.org/web/20221208095137/https://search.maven.org/classic/#search%7Cga%7C1%7Ca%3A%22mysql-connector-java%22%20AND%20g%3A%22mysql%22) 依赖关系:

```java
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <version>6.0.6</version>
</dependency>
```

接下来，让我们使用`Class.forName()`方法注册驱动程序，该方法动态加载驱动程序类:

```java
Class.forName("com.mysql.cj.jdbc.Driver");
```

在旧版本的 JDBC 中，在获得连接之前，我们首先必须通过调用`Class.forName`方法来初始化 JDBC 驱动程序。[从 JDBC 4.0](/web/20221208095137/https://www.baeldung.com/java-jdbc-loading-drivers) ，**开始，所有在类路径中找到的驱动都被自动加载**。因此，在现代环境中，我们不需要这个`Class.forName `零件。

### 3.2。创建连接

要打开一个连接，我们可以使用`DriverManager`类的`getConnection()`方法。这个方法需要一个连接 URL `String` 参数:

```java
try (Connection con = DriverManager
  .getConnection("jdbc:mysql://localhost:3306/myDb", "user1", "pass")) {
    // use con here
}
```

**由于`Connection `是一个`[AutoCloseable](https://web.archive.org/web/20221208095137/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/AutoCloseable.html) `资源，我们应该在`[try-with-resources](/web/20221208095137/https://www.baeldung.com/java-try-with-resources) `块**中使用它。

连接 URL 的语法取决于所使用的数据库类型。让我们来看几个例子:

```java
jdbc:mysql://localhost:3306/myDb?user=user1&password;=pass
```

```java
jdbc:postgresql://localhost/myDb
```

```java
jdbc:hsqldb:mem:myDb
```

要连接到指定的`myDb`数据库，我们必须创建数据库和用户，并添加必要的访问权限:

```java
CREATE DATABASE myDb;
CREATE USER 'user1' IDENTIFIED BY 'pass';
GRANT ALL on myDb.* TO 'user1';
```

## 4。执行 SQL 语句

向数据库发送 SQL 指令，我们可以使用类型为`Statement`、`PreparedStatement,`或`CallableStatement,` 的实例，这些实例可以通过使用`Connection`对象获得。

### 4.1。`Statement`

`Statement`接口包含执行 SQL 命令的基本函数。

首先，让我们创建一个`Statement`对象:

```java
try (Statement stmt = con.createStatement()) {
    // use stmt here
}
```

同样，我们应该在自动资源管理的`try-with-resources `块中使用`Statement` s。

总之，可以通过使用三种方法来执行 SQL 指令:

*   `executeQuery()`用于选择指令
*   `executeUpdate()`用于更新数据或数据库结构
*   当结果未知时,`execute()`可用于上述两种情况

让我们使用`execute()`方法向数据库添加一个`students`表:

```java
String tableSql = "CREATE TABLE IF NOT EXISTS employees" 
  + "(emp_id int PRIMARY KEY AUTO_INCREMENT, name varchar(30),"
  + "position varchar(30), salary double)";
stmt.execute(tableSql);
```

**当使用`execute()`方法更新数据时，`stmt.getUpdateCount()`方法返回受影响的行数。**

如果结果为 0，则没有行受到影响，或者它是一个数据库结构更新命令。

如果该值为-1，则该命令是一个选择查询；然后我们可以使用`stmt.getResultSet()`获得结果。

接下来，让我们使用`executeUpdate()`方法向表中添加一条记录:

```java
String insertSql = "INSERT INTO employees(name, position, salary)"
  + " VALUES('john', 'developer', 2000)";
stmt.executeUpdate(insertSql);
```

对于更新行的命令，该方法返回受影响的行数；对于更新数据库结构的命令，该方法返回 0。

我们可以使用返回类型为`ResultSet`的对象的`executeQuery()`方法从表中检索记录:

```java
String selectSql = "SELECT * FROM employees"; 
try (ResultSet resultSet = stmt.executeQuery(selectSql)) {
    // use resultSet here
}
```

我们应该确保在使用后关闭`ResultSet `实例。否则，我们可能会在比预期更长的时间内保持底层游标打开。要做到这一点，建议使用一个`try-with-resources `块，就像上面的例子一样。

### 4.2。`PreparedStatement`

`PreparedStatement`对象包含预编译的 SQL 序列。它们可以有一个或多个用问号表示的参数。

让我们创建一个`PreparedStatement`，它根据给定的参数更新`employees`表中的记录:

```java
String updatePositionSql = "UPDATE employees SET position=? WHERE emp_id=?";
try (PreparedStatement pstmt = con.prepareStatement(updatePositionSql)) {
    // use pstmt here
}
```

要向`PreparedStatement`添加参数，我们可以使用简单的 setter–`setX()`–其中 X 是参数的类型，方法参数是参数的顺序和值:

```java
pstmt.setString(1, "lead developer");
pstmt.setInt(2, 1);
```

该语句用前面描述的三种方法之一执行:`executeQuery(), executeUpdate(), execute()`不带 SQL `String`参数:

```java
int rowsAffected = pstmt.executeUpdate();
```

### 4.3。`CallableStatement`

`CallableStatement`接口允许调用存储过程。

要创建一个`CallableStatement`对象，我们可以使用`Connection`的`prepareCall()`方法:

```java
String preparedSql = "{call insertEmployee(?,?,?,?)}";
try (CallableStatement cstmt = con.prepareCall(preparedSql)) {
    // use cstmt here
}
```

使用`setX()`方法，像在`PreparedStatement`接口中一样设置存储过程的输入参数值:

```java
cstmt.setString(2, "ana");
cstmt.setString(3, "tester");
cstmt.setDouble(4, 2000);
```

如果存储过程有输出参数，我们需要使用`registerOutParameter()`方法添加它们:

```java
cstmt.registerOutParameter(1, Types.INTEGER);
```

然后，让我们执行该语句，并使用相应的`getX()`方法检索返回值:

```java
cstmt.execute();
int new_id = cstmt.getInt(1);
```

例如，我们需要在 MySql 数据库中创建存储过程:

```java
delimiter //
CREATE PROCEDURE insertEmployee(OUT emp_id int, 
  IN emp_name varchar(30), IN position varchar(30), IN salary double) 
BEGIN
INSERT INTO employees(name, position,salary) VALUES (emp_name,position,salary);
SET emp_id = LAST_INSERT_ID();
END //
delimiter ;
```

上面的`insertEmployee`过程将使用给定的参数将新记录插入到`employees`表中，并在`emp_id` out 参数中返回新记录的 id。

为了能够从 Java 运行存储过程，连接用户需要能够访问存储过程的元数据。这可以通过向用户授予对所有数据库中所有存储过程的权限来实现:

```java
GRANT ALL ON mysql.proc TO 'user1';
```

或者，我们可以打开属性`noAccessToProcedureBodies`设置为`true`的连接:

```java
con = DriverManager.getConnection(
  "jdbc:mysql://localhost:3306/myDb?noAccessToProcedureBodies=true", 
  "user1", "pass");
```

这将通知 JDBC API，用户无权读取过程元数据，因此它将创建所有参数作为 INOUT `String`参数。

## 5。解析查询结果

执行查询后，结果由一个`ResultSet`对象表示，该对象的结构类似于一个表，有行和列。

### 5.1。`ResultSet` 界面

`ResultSet`使用`next()`方法移动到下一行。

让我们首先创建一个`Employee`类来存储我们检索到的记录:

```java
public class Employee {
    private int id;
    private String name;
    private String position;
    private double salary;

    // standard constructor, getters, setters
}
```

接下来，让我们遍历`ResultSet`并为每条记录创建一个`Employee`对象:

```java
String selectSql = "SELECT * FROM employees"; 
try (ResultSet resultSet = stmt.executeQuery(selectSql)) {
    List<Employee> employees = new ArrayList<>(); 
    while (resultSet.next()) { 
        Employee emp = new Employee(); 
        emp.setId(resultSet.getInt("emp_id")); 
        emp.setName(resultSet.getString("name")); 
        emp.setPosition(resultSet.getString("position")); 
        emp.setSalary(resultSet.getDouble("salary")); 
        employees.add(emp); 
    }
}
```

检索每个表格单元格的值可以使用类型为`getX(`的方法来完成，其中 X 表示单元格数据的类型。

`getX()`方法可以与代表单元格顺序的`int`参数或代表列名的`String`参数一起使用。在我们改变查询中列的顺序的情况下，后一个选项是更可取的。

### 5.2。可更新`ResultSet`

隐式地，一个`ResultSet`对象只能向前遍历，不能被修改。

如果我们想使用`ResultSet`来更新数据并在两个方向上遍历它，我们需要创建带有附加参数的`Statement`对象:

```java
stmt = con.createStatement(
  ResultSet.TYPE_SCROLL_INSENSITIVE, 
  ResultSet.CONCUR_UPDATABLE
);
```

要导航这种类型的`ResultSet`，我们可以使用其中一种方法:

*   `first(), last(), beforeFirst(), beforeLast()`–移动到`ResultSet`的第一行或最后一行或之前的行
*   `next(), previous()`–在`ResultSet`中向前和向后导航
*   `getRow() –`获取当前行号
*   `moveToInsertRow(), moveToCurrentRow()`–移动到新的空行进行插入，如果在新行上，则返回当前行
*   `absolute(int row) –`移动到指定行
*   `relative(int nrRows)`–将光标移动给定的行数

可以使用格式为`updateX()`的方法来更新`ResultSet`，其中 X 是单元格数据的类型。这些方法只更新`ResultSet`对象，不更新数据库表。

要将`ResultSet`更改保存到数据库中，我们必须进一步使用以下方法之一:

*   `updateRow()`–将当前行的更改保存到数据库中
*   `insertRow(), deleteRow()`–添加新行或从数据库中删除当前行
*   `refreshRow()`–用数据库中的任何变化刷新`ResultSet`
*   `cancelRowUpdates()`–取消对当前行的更改

让我们看一个通过更新`employee's`表中的数据来使用这些方法的例子:

```java
try (Statement updatableStmt = con.createStatement(
  ResultSet.TYPE_SCROLL_INSENSITIVE, ResultSet.CONCUR_UPDATABLE)) {
    try (ResultSet updatableResultSet = updatableStmt.executeQuery(selectSql)) {
        updatableResultSet.moveToInsertRow();
        updatableResultSet.updateString("name", "mark");
        updatableResultSet.updateString("position", "analyst");
        updatableResultSet.updateDouble("salary", 2000);
        updatableResultSet.insertRow();
    }
}
```

## 6。解析元数据

JDBC API 允许查找关于数据库的信息，称为元数据。

### 6.1。`DatabaseMetadata`

`DatabaseMetadata`接口可用于获取数据库的一般信息，如表、存储过程或 SQL 方言。

让我们快速看一下如何在数据库表上检索信息:

```java
DatabaseMetaData dbmd = con.getMetaData();
ResultSet tablesResultSet = dbmd.getTables(null, null, "%", null);
while (tablesResultSet.next()) {
    LOG.info(tablesResultSet.getString("TABLE_NAME"));
}
```

### 6.2。`ResultSetMetadata`

该界面可用于查找某个`ResultSet`的信息，如其列的编号和名称:

```java
ResultSetMetaData rsmd = rs.getMetaData();
int nrColumns = rsmd.getColumnCount();

IntStream.range(1, nrColumns).forEach(i -> {
    try {
        LOG.info(rsmd.getColumnName(i));
    } catch (SQLException e) {
        e.printStackTrace();
    }
});
```

## 7。处理交易

默认情况下，每条 SQL 语句都在完成后立即提交。然而，**也可以通过编程**来控制事务。

在我们希望保持数据一致性的情况下，这可能是必要的，例如，当我们只想提交一个事务，如果前一个事务已经成功完成。

首先，我们需要将`Connection`的`autoCommit`属性设置为`false`，然后使用 `commit()`和`rollback()`方法来控制事务。

让我们在 employee `position`列更新之后为`salary`列添加第二个 update 语句，并将它们包装在一个事务中。这样，只有职位成功更新后，工资才会更新:

```java
String updatePositionSql = "UPDATE employees SET position=? WHERE emp_id=?";
PreparedStatement pstmt = con.prepareStatement(updatePositionSql);
pstmt.setString(1, "lead developer");
pstmt.setInt(2, 1);

String updateSalarySql = "UPDATE employees SET salary=? WHERE emp_id=?";
PreparedStatement pstmt2 = con.prepareStatement(updateSalarySql);
pstmt.setDouble(1, 3000);
pstmt.setInt(2, 1);

boolean autoCommit = con.getAutoCommit();
try {
    con.setAutoCommit(false);
    pstmt.executeUpdate();
    pstmt2.executeUpdate();
    con.commit();
} catch (SQLException exc) {
    con.rollback();
} finally {
    con.setAutoCommit(autoCommit);
}
```

为了简洁起见，我们在这里省略了`try-with-resources `块。

## 8.关闭资源

当我们不再使用它时，**我们需要关闭连接来释放数据库资源**。

我们可以使用`close()` API 来做到这一点:

```java
con.close();
```

然而，如果我们正在使用一个`try-with-resources`块中的资源，我们不需要显式调用`close()`方法，因为`try-with-resources `块会自动为我们做这件事。

**对于`Statement`、`PreparedStatement`、`CallableStatement`和`ResultSet`、**也是如此

## 9。结论

在本教程中，我们了解了使用 JDBC API 的基本知识。

和往常一样，例子的完整源代码可以在 GitHub 上找到[。](https://web.archive.org/web/20221208095137/https://github.com/eugenp/tutorials/tree/master/persistence-modules/core-java-persistence)