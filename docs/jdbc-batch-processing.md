# JDBC 的批处理

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/jdbc-batch-processing>

## 1。简介

Java 数据库连接(JDBC)是一个用于与数据库交互的 Java API。批处理将多个查询组合成一个单元，并在一次网络行程中将其传递给数据库。

在本文中，我们将发现如何使用 JDBC 来批量处理 SQL 查询。

关于 JDBC 的更多信息，你可以点击这里查看我们的介绍文章。

## 2。为什么要批处理？

性能和数据一致性是进行批处理的主要动机。

### 2.1。改进的性能

一些用例需要将大量数据插入数据库表中。使用 JDBC 时，无需批处理就能实现这一点的方法之一是顺序执行多个查询。

让我们看一个发送到数据库的顺序查询的例子:

```java
statement.execute("INSERT INTO EMPLOYEE(ID, NAME, DESIGNATION) "
 + "VALUES ('1','EmployeeName1','Designation1')"); 
statement.execute("INSERT INTO EMPLOYEE(ID, NAME, DESIGNATION) "
 + "VALUES ('2','EmployeeName2','Designation2')");
```

这些顺序调用会增加数据库的网络访问次数，从而导致性能下降。

通过使用批处理，这些查询可以在一次调用中发送到数据库，从而提高性能。

### 2.2。数据一致性

在某些情况下，需要将数据推送到多个表中。这导致了一个相互关联的事务，其中被推送的查询序列很重要。

在执行过程中发生的任何错误都会导致先前查询所推送的数据的回滚(如果有的话)。

让我们看一个向多个表添加数据的示例:

```java
statement.execute("INSERT INTO EMPLOYEE(ID, NAME, DESIGNATION) "
 + "VALUES ('1','EmployeeName1','Designation1')"); 
statement.execute("INSERT INTO EMP_ADDRESS(ID, EMP_ID, ADDRESS) "
 + "VALUES ('10','1','Address')"); 
```

当第一条语句成功而第二条语句失败时，上述方法中的一个典型问题就出现了。在这种情况下，第一条语句插入的数据没有回滚，导致数据不一致。

我们可以通过将一个事务跨越多个插入/更新来实现数据一致性，然后在最后提交该事务，或者在出现异常的情况下执行回滚，但是在这种情况下，我们仍然会对每个语句重复地访问数据库。

## 3。如何进行批量处理

JDBC 提供了两个类，`Statement`和`PreparedStatement`来执行数据库查询。这两个类都有自己的`addBatch()`和`executeBatch()`方法的实现，为我们提供了批处理功能。

### 3.1。使用`Statement` 进行批量处理

使用 JDBC，在数据库上执行查询最简单的方法是通过`Statement` 对象`.`

首先，使用`addBatch()`我们可以将所有 SQL 查询添加到一个批处理中，然后使用`executeBatch()`执行这些 SQL 查询。

`executeBatch()`的返回类型是一个`int`数组，指示每个 SQL 语句的执行影响了多少记录。

让我们看一个使用语句创建和执行批处理的例子:

```java
Statement statement = connection.createStatement();
statement.addBatch("INSERT INTO EMPLOYEE(ID, NAME, DESIGNATION) "
 + "VALUES ('1','EmployeeName','Designation')");
statement.addBatch("INSERT INTO EMP_ADDRESS(ID, EMP_ID, ADDRESS) "
 + "VALUES ('10','1','Address')");
statement.executeBatch(); 
```

在上面的例子中，我们试图使用`Statement`将记录插入到`EMPLOYEE`和`EMP_ADDRESS`表中。我们可以看到 SQL 查询是如何添加到要执行的批处理中的。

### 3.2。使用`PreparedStatement` 进行批量处理

`PreparedStatement` 是另一个用于执行 SQL 查询的类`.` 它支持 SQL 语句的重用，并要求我们为每次更新/插入设置新的参数。

让我们看一个使用`PreparedStatement.` 的例子。首先，我们使用一个编码为`String:`的 SQL 查询来设置语句

```java
String[] EMPLOYEES = new String[]{"Zuck","Mike","Larry","Musk","Steve"};
String[] DESIGNATIONS = new String[]{"CFO","CSO","CTO","CEO","CMO"};

String insertEmployeeSQL = "INSERT INTO EMPLOYEE(ID, NAME, DESIGNATION) "
 + "VALUES (?,?,?)";
PreparedStatement employeeStmt = connection.prepareStatement(insertEmployeeSQL);
```

接下来，我们遍历一组`String`值，并将一个新配置的查询添加到批处理中。

一旦循环完成，我们就执行批处理:

```java
for(int i = 0; i < EMPLOYEES.length; i++){
    String employeeId = UUID.randomUUID().toString();
    employeeStmt.setString(1,employeeId);
    employeeStmt.setString(2,EMPLOYEES[i]);
    employeeStmt.setString(3,DESIGNATIONS[i]);
    employeeStmt.addBatch();
}
employeeStmt.executeBatch(); 
```

在上面显示的例子中，我们使用`PreparedStatement.` 将记录插入到`EMPLOYEE`表中，我们可以看到如何在查询中设置要插入的值，然后将其添加到要执行的批处理中。

## 4。结论

在本文中，我们看到了在使用 JDBC 与数据库交互时，SQL 查询的批处理是多么重要。

和往常一样，与本文相关的代码可以在 Github 上找到[。](https://web.archive.org/web/20220525123415/https://github.com/eugenp/tutorials/tree/master/persistence-modules/core-java-persistence)