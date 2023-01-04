# Spring JdbcTemplate 单元测试

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-jdbctemplate-testing>

## 1。概述

[Spring `JdbcTemplate`](/web/20220625232250/https://www.baeldung.com/spring-jdbc-jdbctemplate) 是开发者专注于编写 SQL 查询和提取结果的强大工具。它连接到后端数据库并直接执行 SQL 查询。

因此，我们可以使用集成测试来确保我们可以正确地从数据库中提取数据。此外，我们可以编写单元测试来检查相关功能的正确性。

在本教程中，我们将展示如何对`JdbcTemplate`代码进行单元测试。

## 2。`JdbcTemplate`和运行查询

首先，让我们从一个使用`JdbcTemplate`的数据访问对象(DAO)类开始:

```java
public class EmployeeDAO {
    private JdbcTemplate jdbcTemplate;

    public void setDataSource(DataSource dataSource) {
        jdbcTemplate = new JdbcTemplate(dataSource);
    }

    public int getCountOfEmployees() {
        return jdbcTemplate.queryForObject("SELECT COUNT(*) FROM EMPLOYEE", Integer.class);
    }
}
```

我们将一个`DataSource`对象依赖注入到`EmployeeDAO`类中。然后，我们在 setter 方法中创建了`JdbcTemplate`对象。同样，我们在示例方法`getCountOfEmployees().`中使用`JdbcTemplate`

使用`JdbcTemplate`和`.` 的单元测试方法有两种

**我们可以使用内存数据库，例如 [H2 数据库](/web/20220625232250/https://www.baeldung.com/spring-boot-h2-database)作为测试**的数据源。然而，在现实世界的应用程序中，SQL 查询可能有复杂的关系，我们需要创建复杂的设置脚本来测试 SQL 语句。

**或者，我们也可以模拟`JdbcTemplate `对象来测试方法功能。**

## 3。H2 数据库单元测试

**我们可以创建一个连接到 H2 数据库的数据源，并将其注入到`EmployeeDAO`类:**

```java
@Test
public void whenInjectInMemoryDataSource_thenReturnCorrectEmployeeCount() {
    DataSource dataSource = new EmbeddedDatabaseBuilder().setType(EmbeddedDatabaseType.H2)
      .addScript("classpath:jdbc/schema.sql")
      .addScript("classpath:jdbc/test-data.sql")
      .build();

    EmployeeDAO employeeDAO = new EmployeeDAO();
    employeeDAO.setDataSource(dataSource);

    assertEquals(4, employeeDAO.getCountOfEmployees());
}
```

在这个测试中，我们首先在 H2 数据库上构建一个数据源。在构造过程中，我们执行`schema.sql`来创建`EMPLOYEE`表:

```java
CREATE TABLE EMPLOYEE
(
    ID int NOT NULL PRIMARY KEY,
    FIRST_NAME varchar(255),
    LAST_NAME varchar(255),
    ADDRESS varchar(255)
);
```

此外，我们运行`test-data.sql`将测试数据添加到表中:

```java
INSERT INTO EMPLOYEE VALUES (1, 'James', 'Gosling', 'Canada');
INSERT INTO EMPLOYEE VALUES (2, 'Donald', 'Knuth', 'USA');
INSERT INTO EMPLOYEE VALUES (3, 'Linus', 'Torvalds', 'Finland');
INSERT INTO EMPLOYEE VALUES (4, 'Dennis', 'Ritchie', 'USA');
```

然后，我们可以将这个数据源注入到`EmployeeDAO`类中，并在内存中的 H2 数据库上测试`getCountOfEmployees`方法。

## 4。使用模拟对象的单元测试

**我们可以模仿`JdbcTemplate`对象，这样我们就不需要在数据库上运行 SQL 语句:**

```java
public class EmployeeDAOUnitTest {
    @Mock
    JdbcTemplate jdbcTemplate;

    @Test
    public void whenMockJdbcTemplate_thenReturnCorrectEmployeeCount() {
        EmployeeDAO employeeDAO = new EmployeeDAO();
        ReflectionTestUtils.setField(employeeDAO, "jdbcTemplate", jdbcTemplate);
        Mockito.when(jdbcTemplate.queryForObject("SELECT COUNT(*) FROM EMPLOYEE", Integer.class))
          .thenReturn(4);

        assertEquals(4, employeeDAO.getCountOfEmployees());
    }
}
```

在这个单元测试中，我们首先用`@Mock`注释声明一个模拟`JdbcTemplate`对象。然后我们使用`[ReflectionTestUtils](/web/20220625232250/https://www.baeldung.com/spring-reflection-test-utils). `将它注入到`EmployeeDAO` 对象中，同样，我们使用 [`Mockito`](/web/20220625232250/https://www.baeldung.com/mockito-mock-methods) 实用程序来模拟`JdbcTemplate`查询的返回结果。这允许我们在不连接数据库的情况下测试`getCountOfEmployees` 方法的功能。

当我们模拟`JdbcTemplate`查询时，我们使用 SQL 语句字符串的精确匹配。在现实世界的应用程序中，我们可能会创建复杂的 SQL 字符串，并且很难做到完全匹配。因此，我们也可以用 [`anyString()`](/web/20220625232250/https://www.baeldung.com/mockito-argument-matchers) 的方法来绕过字符串检查:

```java
Mockito.when(jdbcTemplate.queryForObject(Mockito.anyString(), Mockito.eq(Integer.class)))
  .thenReturn(3);
assertEquals(3, employeeDAO.getCountOfEmployees());
```

## 5.Spring Boot @ JDBC 测试

最后，如果我们使用 Spring Boot，我们可以使用一个注释来引导一个 H2 数据库和一个`JdbcTemplate` bean: `@JdbcTest`的测试。

让我们用这个注释创建一个测试类:

```java
@JdbcTest
@Sql({"schema.sql", "test-data.sql"})
class EmployeeDAOIntegrationTest {
    @Autowired
    private JdbcTemplate jdbcTemplate;

    @Test
    void whenInjectInMemoryDataSource_thenReturnCorrectEmployeeCount() {
        EmployeeDAO employeeDAO = new EmployeeDAO();
        employeeDAO.setJdbcTemplate(jdbcTemplate);

        assertEquals(4, employeeDAO.getCountOfEmployees());
    }
}
```

我们还可以注意到`@Sql`注释的存在，它允许我们[指定在测试](/web/20220625232250/https://www.baeldung.com/spring-boot-data-sql-and-schema-sql)之前运行的 SQL 文件。

## 6。结论

在本教程中，我们展示了单元测试的多种方法`JdbcTemplate.`

和往常一样，这篇文章的源代码可以在 GitHub 的[上找到。](https://web.archive.org/web/20220625232250/https://github.com/eugenp/tutorials/tree/master/persistence-modules/spring-jdbc)