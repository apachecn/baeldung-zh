# Apache Commons DbUtils 指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/apache-commons-dbutils>

## 1。概述

Apache Commons DbUtils 是一个小型的库，它使得使用 JDBC 更加容易。

在本文中，我们将实现一些示例来展示它的特性和功能。

## 2。设置

### 2.1。Maven 依赖关系

首先，我们需要将`commons-dbutils`和`h2`依赖项添加到我们的`pom.xml`中:

```java
<dependency>
    <groupId>commons-dbutils</groupId>
    <artifactId>commons-dbutils</artifactId>
    <version>1.6</version>
</dependency>
<dependency>
    <groupId>com.h2database</groupId>
    <artifactId>h2</artifactId>
    <version>1.4.196</version>
</dependency>
```

你可以在 Maven Central 上找到最新版本的 [commons-dbutils](https://web.archive.org/web/20220628154500/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22commons-dbutils%22%20AND%20a%3A%22commons-dbutils%22) 和 [h2](https://web.archive.org/web/20220628154500/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22com.h2database%22%20AND%20a%3A%22h2%22) 。

### 2.2。测试数据库

有了依赖项之后，让我们创建一个脚本来创建我们将使用的表和记录:

```java
CREATE TABLE employee(
    id int NOT NULL PRIMARY KEY auto_increment,
    firstname varchar(255),
    lastname varchar(255),
    salary double,
    hireddate date,
);

CREATE TABLE email(
    id int NOT NULL PRIMARY KEY auto_increment,
    employeeid int,
    address varchar(255)
);

INSERT INTO employee (firstname,lastname,salary,hireddate)
  VALUES ('John', 'Doe', 10000.10, to_date('01-01-2001','dd-mm-yyyy'));
// ...
INSERT INTO email (employeeid,address)
  VALUES (1, '[[email protected]](/web/20220628154500/https://www.baeldung.com/cdn-cgi/l/email-protection)');
// ...
```

本文中的所有示例测试用例都将使用一个新创建的到 H2 内存数据库的连接:

```java
public class DbUtilsUnitTest {
    private Connection connection;

    @Before
    public void setupDB() throws Exception {
        Class.forName("org.h2.Driver");
        String db
          = "jdbc:h2:mem:;INIT=runscript from 'classpath:/employees.sql'";
        connection = DriverManager.getConnection(db);
    }

    @After
    public void closeBD() {
        DbUtils.closeQuietly(connection);
    }
    // ...
}
```

### 2.3。波霍斯

最后，我们需要两个简单的类:

```java
public class Employee {
    private Integer id;
    private String firstName;
    private String lastName;
    private Double salary;
    private Date hiredDate;

    // standard constructors, getters, and setters
}

public class Email {
    private Integer id;
    private Integer employeeId;
    private String address;

    // standard constructors, getters, and setters
}
```

## 3。简介

DbUtils 库提供了**`QueryRunner`类作为大多数可用功能的主要入口点**。

该类通过接收到数据库的连接、要执行的 SQL 语句以及为查询的占位符提供值的可选参数列表来工作。

正如我们将在后面看到的，一些方法也接收了一个`ResultSetHandler`实现——它负责将`ResultSet`实例转换成我们的应用程序期望的对象。

当然，这个库已经提供了几个处理最常见转换的实现，比如列表、映射和 JavaBeans。

## 4。查询数据

现在我们知道了基本知识，我们准备查询我们的数据库。

让我们从一个简单的例子开始，用一个`MapListHandler`获得数据库中的所有记录，作为一个地图列表:

```java
@Test
public void givenResultHandler_whenExecutingQuery_thenExpectedList()
  throws SQLException {
    MapListHandler beanListHandler = new MapListHandler();

    QueryRunner runner = new QueryRunner();
    List<Map<String, Object>> list
      = runner.query(connection, "SELECT * FROM employee", beanListHandler);

    assertEquals(list.size(), 5);
    assertEquals(list.get(0).get("firstname"), "John");
    assertEquals(list.get(4).get("firstname"), "Christian");
}
```

接下来，下面是一个使用`BeanListHandler`将结果转换成`Employee`实例的例子:

```java
@Test
public void givenResultHandler_whenExecutingQuery_thenEmployeeList()
  throws SQLException {
    BeanListHandler<Employee> beanListHandler
      = new BeanListHandler<>(Employee.class);

    QueryRunner runner = new QueryRunner();
    List<Employee> employeeList
      = runner.query(connection, "SELECT * FROM employee", beanListHandler);

    assertEquals(employeeList.size(), 5);
    assertEquals(employeeList.get(0).getFirstName(), "John");
    assertEquals(employeeList.get(4).getFirstName(), "Christian");
}
```

对于返回单个值的查询，我们可以使用`ScalarHandler`:

```java
@Test
public void givenResultHandler_whenExecutingQuery_thenExpectedScalar()
  throws SQLException {
    ScalarHandler<Long> scalarHandler = new ScalarHandler<>();

    QueryRunner runner = new QueryRunner();
    String query = "SELECT COUNT(*) FROM employee";
    long count
      = runner.query(connection, query, scalarHandler);

    assertEquals(count, 5);
}
```

要了解所有的`ResultSerHandler`实现，可以参考 [`ResultSetHandler`文档](https://web.archive.org/web/20220628154500/https://commons.apache.org/proper/commons-dbutils/apidocs/org/apache/commons/dbutils/ResultSetHandler.html)。

### 4.1。自定义处理程序

当我们需要更多地控制结果如何被转换成对象时，我们也可以创建一个自定义的处理程序来传递给`QueryRunner`的方法。

这可以通过实现`ResultSetHandler`接口或者扩展库提供的一个现有实现来实现。

让我们看看第二种方法是什么样子的。首先，让我们向我们的`Employee`类添加另一个字段:

```java
public class Employee {
    private List<Email> emails;
    // ...
}
```

现在，让我们创建一个扩展`BeanListHandler`类型的类，并为每个雇员设置电子邮件列表:

```java
public class EmployeeHandler extends BeanListHandler<Employee> {

    private Connection connection;

    public EmployeeHandler(Connection con) {
        super(Employee.class);
        this.connection = con;
    }

    @Override
    public List<Employee> handle(ResultSet rs) throws SQLException {
        List<Employee> employees = super.handle(rs);

        QueryRunner runner = new QueryRunner();
        BeanListHandler<Email> handler = new BeanListHandler<>(Email.class);
        String query = "SELECT * FROM email WHERE employeeid = ?";

        for (Employee employee : employees) {
            List<Email> emails
              = runner.query(connection, query, handler, employee.getId());
            employee.setEmails(emails);
        }
        return employees;
    }
}
```

注意，我们在构造函数中期待一个`Connection`对象，这样我们就可以执行查询来获取电子邮件。

最后，让我们测试我们的代码，看看是否一切都按预期运行:

```java
@Test
public void
  givenResultHandler_whenExecutingQuery_thenEmailsSetted()
    throws SQLException {
    EmployeeHandler employeeHandler = new EmployeeHandler(connection);

    QueryRunner runner = new QueryRunner();
    List<Employee> employees
      = runner.query(connection, "SELECT * FROM employee", employeeHandler);

    assertEquals(employees.get(0).getEmails().size(), 2);
    assertEquals(employees.get(2).getEmails().size(), 3);
}
```

### 4.2。定制行处理器

在我们的例子中，`employee`表的列名与我们的`Employee`类的字段名匹配(匹配不区分大小写)。然而，情况并不总是如此——例如，当列名使用下划线来分隔复合词时。

在这些情况下，我们可以利用`RowProcessor`接口及其实现将列名映射到我们的类中适当的字段。

让我们看看这个是什么样子的。首先，让我们创建另一个表，并在其中插入一些记录:

```java
CREATE TABLE employee_legacy (
    id int NOT NULL PRIMARY KEY auto_increment,
    first_name varchar(255),
    last_name varchar(255),
    salary double,
    hired_date date,
);

INSERT INTO employee_legacy (first_name,last_name,salary,hired_date)
  VALUES ('John', 'Doe', 10000.10, to_date('01-01-2001','dd-mm-yyyy'));
// ...
```

现在，让我们修改我们的`EmployeeHandler`类:

```java
public class EmployeeHandler extends BeanListHandler<Employee> {
    // ...
    public EmployeeHandler(Connection con) {
        super(Employee.class,
          new BasicRowProcessor(new BeanProcessor(getColumnsToFieldsMap())));
        // ...
    }
    public static Map<String, String> getColumnsToFieldsMap() {
        Map<String, String> columnsToFieldsMap = new HashMap<>();
        columnsToFieldsMap.put("FIRST_NAME", "firstName");
        columnsToFieldsMap.put("LAST_NAME", "lastName");
        columnsToFieldsMap.put("HIRED_DATE", "hiredDate");
        return columnsToFieldsMap;
    }
    // ...
}
```

注意，我们使用了一个`BeanProcessor`来完成列到字段的实际映射，并且只针对那些需要被寻址的字段。

最后，让我们测试一切正常:

```java
@Test
public void
  givenResultHandler_whenExecutingQuery_thenAllPropertiesSetted()
    throws SQLException {
    EmployeeHandler employeeHandler = new EmployeeHandler(connection);

    QueryRunner runner = new QueryRunner();
    String query = "SELECT * FROM employee_legacy";
    List<Employee> employees
      = runner.query(connection, query, employeeHandler);

    assertEquals((int) employees.get(0).getId(), 1);
    assertEquals(employees.get(0).getFirstName(), "John");
}
```

## 5。插入记录

`QueryRunner`类提供了两种在数据库中创建记录的方法。

第一种方法是使用`update()`方法，并传递 SQL 语句和可选的替换参数列表。方法返回插入的记录数:

```java
@Test
public void whenInserting_thenInserted() throws SQLException {
    QueryRunner runner = new QueryRunner();
    String insertSQL
      = "INSERT INTO employee (firstname,lastname,salary, hireddate) "
        + "VALUES (?, ?, ?, ?)";

    int numRowsInserted
      = runner.update(
        connection, insertSQL, "Leia", "Kane", 60000.60, new Date());

    assertEquals(numRowsInserted, 1);
}
```

第二种方法是使用`insert()`方法，除了 SQL 语句和替换参数，还需要一个`ResultSetHandler`来转换生成的自动生成的键。返回值将是处理程序返回的内容:

```java
@Test
public void
  givenHandler_whenInserting_thenExpectedId() throws SQLException {
    ScalarHandler<Integer> scalarHandler = new ScalarHandler<>();

    QueryRunner runner = new QueryRunner();
    String insertSQL
      = "INSERT INTO employee (firstname,lastname,salary, hireddate) "
        + "VALUES (?, ?, ?, ?)";

    int newId
      = runner.insert(
        connection, insertSQL, scalarHandler,
        "Jenny", "Medici", 60000.60, new Date());

    assertEquals(newId, 6);
}
```

## 6。更新和删除

`QueryRunner`类的`update()`方法也可以用来修改和删除数据库中的记录。

它的用法很简单。以下是如何更新员工工资的示例:

```java
@Test
public void givenSalary_whenUpdating_thenUpdated()
 throws SQLException {
    double salary = 35000;

    QueryRunner runner = new QueryRunner();
    String updateSQL
      = "UPDATE employee SET salary = salary * 1.1 WHERE salary <= ?";
    int numRowsUpdated = runner.update(connection, updateSQL, salary);

    assertEquals(numRowsUpdated, 3);
}
```

下面是另一个删除给定 id 的雇员的例子:

```java
@Test
public void whenDeletingRecord_thenDeleted() throws SQLException {
    QueryRunner runner = new QueryRunner();
    String deleteSQL = "DELETE FROM employee WHERE id = ?";
    int numRowsDeleted = runner.update(connection, deleteSQL, 3);

    assertEquals(numRowsDeleted, 1);
}
```

## 7。异步操作

DbUtils 提供了`AsyncQueryRunner`类来异步执行操作。这个类的方法与`QueryRunner`类的方法有对应关系，除了它们返回一个`Future`实例。

下面是一个获取数据库中所有雇员的示例，等待 10 秒钟即可得到结果:

```java
@Test
public void
  givenAsyncRunner_whenExecutingQuery_thenExpectedList() throws Exception {
    AsyncQueryRunner runner
      = new AsyncQueryRunner(Executors.newCachedThreadPool());

    EmployeeHandler employeeHandler = new EmployeeHandler(connection);
    String query = "SELECT * FROM employee";
    Future<List<Employee>> future
      = runner.query(connection, query, employeeHandler);
    List<Employee> employeeList = future.get(10, TimeUnit.SECONDS);

    assertEquals(employeeList.size(), 5);
}
```

## 8。结论

在本教程中，我们探索了 Apache Commons DbUtils 库最显著的特性。

我们查询数据并将其转换为不同的对象类型，插入记录以获得生成的主键，并根据给定的标准更新和删除数据。我们还利用了`AsyncQueryRunner`类来异步执行查询操作。

和往常一样，本文的完整源代码可以在 Github 上找到[。](https://web.archive.org/web/20220628154500/https://github.com/eugenp/tutorials/tree/master/libraries-apache-commons)