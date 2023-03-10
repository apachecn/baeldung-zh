# 春天的 JDBC

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-jdbc-jdbctemplate>

## 1。概述

在本教程中，我们将浏览 Spring JDBC 模块的实际用例。

春天 JDBC 的所有课程分为四个独立的包:

*   **`core`**——JDBC 的核心功能。这个包下的一些重要类包括`JdbcTemplate`、 `SimpleJdbcInsert`、`SimpleJdbcCall`和`NamedParameterJdbcTemplate`。
*   **`datasource`** —访问数据源的实用程序类。它还有各种数据源实现，用于在 Jakarta EE 容器之外测试 JDBC 代码。
*   **`object`** —以面向对象的方式访问数据库。它允许运行查询并将结果作为业务对象返回。它还在业务对象的列和属性之间映射查询结果。
*   **`support`** —支持`core`和`object`包下的类，如提供`SQLException`翻译功能

    ## 延伸阅读:

    ## [弹簧安全:探索 JDBC 认证](/web/20220630132202/https://www.baeldung.com/spring-security-jdbc-authentication)

    探索 Spring 提供的使用现有数据源配置执行 JDBC 认证的能力。[阅读更多](/web/20220630132202/https://www.baeldung.com/spring-security-jdbc-authentication) →

    ## [春季数据 JPA 简介](/web/20220630132202/https://www.baeldung.com/the-persistence-layer-with-spring-data-jpa)

    Spring 数据 JPA 简介与 Spring 4-Spring config，DAO，手动与生成查询和事务管理。[阅读更多](/web/20220630132202/https://www.baeldung.com/the-persistence-layer-with-spring-data-jpa) →

## 2。配置

让我们从数据源的一些简单配置开始。

我们将使用 MySQL 数据库:

```java
@Configuration
@ComponentScan("com.baeldung.jdbc")
public class SpringJdbcConfig {
    @Bean
    public DataSource mysqlDataSource() {
        DriverManagerDataSource dataSource = new DriverManagerDataSource();
        dataSource.setDriverClassName("com.mysql.jdbc.Driver");
        dataSource.setUrl("jdbc:mysql://localhost:3306/springjdbc");
        dataSource.setUsername("guest_user");
        dataSource.setPassword("guest_password");

        return dataSource;
    }
}
```

或者，我们也可以很好地利用嵌入式数据库进行开发或测试。

下面是一个快速配置，它创建了一个 H2 嵌入式数据库实例，并用简单的 SQL 脚本预先填充它:

```java
@Bean
public DataSource dataSource() {
    return new EmbeddedDatabaseBuilder()
      .setType(EmbeddedDatabaseType.H2)
      .addScript("classpath:jdbc/schema.sql")
      .addScript("classpath:jdbc/test-data.sql").build();
} 
```

最后，同样的事情可以通过对`datasource`使用 XML 配置来完成:

```java
<bean id="dataSource" class="org.apache.commons.dbcp.BasicDataSource" 
  destroy-method="close">
    <property name="driverClassName" value="com.mysql.jdbc.Driver"/>
    <property name="url" value="jdbc:mysql://localhost:3306/springjdbc"/>
    <property name="username" value="guest_user"/>
    <property name="password" value="guest_password"/>
</bean>
```

## 3。`JdbcTemplate`和运行查询

### 3.1。基本查询

JDBC 模板是主要的 API，通过它我们可以访问我们感兴趣的大部分功能:

*   连接的创建和关闭
*   运行语句和存储过程调用
*   迭代`ResultSet` 并返回结果

首先，让我们从一个简单的例子开始，看看 `JdbcTemplate`能做什么:

```java
int result = jdbcTemplate.queryForObject(
    "SELECT COUNT(*) FROM EMPLOYEE", Integer.class); 
```

这里有一个简单的插页:

```java
public int addEmplyee(int id) {
    return jdbcTemplate.update(
      "INSERT INTO EMPLOYEE VALUES (?, ?, ?, ?)", id, "Bill", "Gates", "USA");
}
```

注意使用`?`字符提供参数的标准语法。

接下来，让我们看看这个语法的另一种选择。

### 3.2。带命名参数的查询

为了让**支持命名参数**，我们将使用框架提供的另一个 JDBC 模板——T0。

此外，这包装了`JbdcTemplate`,并提供了传统语法的替代方法。指定参数。

在幕后，它将命名参数替换为 JDBC `?`占位符，并委托给包装的`JDCTemplate` 来运行查询:

```java
SqlParameterSource namedParameters = new MapSqlParameterSource().addValue("id", 1);
return namedParameterJdbcTemplate.queryForObject(
  "SELECT FIRST_NAME FROM EMPLOYEE WHERE ID = :id", namedParameters, String.class);
```

注意我们是如何使用`MapSqlParameterSource`来为命名参数提供值的。

让我们看看如何使用 bean 的属性来确定命名参数:

```java
Employee employee = new Employee();
employee.setFirstName("James");

String SELECT_BY_ID = "SELECT COUNT(*) FROM EMPLOYEE WHERE FIRST_NAME = :firstName";

SqlParameterSource namedParameters = new BeanPropertySqlParameterSource(employee);
return namedParameterJdbcTemplate.queryForObject(
  SELECT_BY_ID, namedParameters, Integer.class);
```

注意我们现在如何使用`BeanPropertySqlParameterSource`实现，而不是像以前那样手动指定命名参数。

### 3.3。将查询结果映射到 Java 对象

另一个非常有用的特性是通过实现 `RowMapper`接口将查询结果映射到 Java 对象的能力。

例如，对于查询返回的每一行，Spring 使用行映射器来填充 java bean:

```java
public class EmployeeRowMapper implements RowMapper<Employee> {
    @Override
    public Employee mapRow(ResultSet rs, int rowNum) throws SQLException {
        Employee employee = new Employee();

        employee.setId(rs.getInt("ID"));
        employee.setFirstName(rs.getString("FIRST_NAME"));
        employee.setLastName(rs.getString("LAST_NAME"));
        employee.setAddress(rs.getString("ADDRESS"));

        return employee;
    }
}
```

随后，我们现在可以将行映射器传递给查询 API，并获得完全填充的 Java 对象:

```java
String query = "SELECT * FROM EMPLOYEE WHERE ID = ?";
Employee employee = jdbcTemplate.queryForObject(
  query, new Object[] { id }, new EmployeeRowMapper());
```

## 4。异常翻译

Spring 自带开箱即用的数据异常层次结构——以`DataAccessException`作为根异常——它将所有底层的原始异常都转化为它。

因此，我们通过不处理低级别的持久性异常来保持理智。我们还受益于 Spring 将底层异常封装在`DataAccessException` 或它的一个子类中。

这也使得异常处理机制独立于我们正在使用的底层数据库。

除了默认的`SQLErrorCodeSQLExceptionTranslator`，我们还可以提供自己的`SQLExceptionTranslator`实现。

这里有一个自定义实现的简单示例——当出现重复键冲突时自定义错误消息，这在使用 H2 时会导致[错误代码 23505](https://web.archive.org/web/20220630132202/https://www.h2database.com/javadoc/org/h2/api/ErrorCode.html#c23505) :

```java
public class CustomSQLErrorCodeTranslator extends SQLErrorCodeSQLExceptionTranslator {
    @Override
    protected DataAccessException
      customTranslate(String task, String sql, SQLException sqlException) {
        if (sqlException.getErrorCode() == 23505) {
          return new DuplicateKeyException(
            "Custom Exception translator - Integrity constraint violation.", sqlException);
        }
        return null;
    }
}
```

要使用这个定制的异常翻译器，我们需要通过调用`setExceptionTranslator()` 方法将它传递给`JdbcTemplate`:

```java
CustomSQLErrorCodeTranslator customSQLErrorCodeTranslator = 
  new CustomSQLErrorCodeTranslator();
jdbcTemplate.setExceptionTranslator(customSQLErrorCodeTranslator);
```

## 5。使用简单 Jdbc 类的 JDBC 操作

类提供了一种配置和运行 SQL 语句的简单方法。这些类使用数据库元数据来构建基本查询。因此，`SimpleJdbcInsert`和`SimpleJdbcCall` 类提供了一种更简单的方式来运行插入和存储过程调用。

## 5.1。`SimpleJdbcInsert`

让我们来看看如何用最少的配置运行简单的 insert 语句。

**INSERT 语句是根据`SimpleJdbcInsert`的配置生成的。**我们只需要提供表名、列名和值。

首先，让我们创建一个`SimpleJdbcInsert`:

```java
SimpleJdbcInsert simpleJdbcInsert = 
  new SimpleJdbcInsert(dataSource).withTableName("EMPLOYEE");
```

接下来，让我们提供列名和值，并运行操作:

```java
public int addEmplyee(Employee emp) {
    Map<String, Object> parameters = new HashMap<String, Object>();
    parameters.put("ID", emp.getId());
    parameters.put("FIRST_NAME", emp.getFirstName());
    parameters.put("LAST_NAME", emp.getLastName());
    parameters.put("ADDRESS", emp.getAddress());

    return simpleJdbcInsert.execute(parameters);
}
```

此外，我们可以使用`executeAndReturnKey()` API 来允许**数据库生成主键**。我们还需要配置实际自动生成的列:

```java
SimpleJdbcInsert simpleJdbcInsert = new SimpleJdbcInsert(dataSource)
                                        .withTableName("EMPLOYEE")
                                        .usingGeneratedKeyColumns("ID");

Number id = simpleJdbcInsert.executeAndReturnKey(parameters);
System.out.println("Generated id - " + id.longValue());
```

最后，我们还可以通过使用`BeanPropertySqlParameterSource`和`MapSqlParameterSource`来传递这些数据。

## 5.2。存储过程用`SimpleJdbcCall`

让我们也来看看运行存储过程。

我们将利用`SimpleJdbcCall`抽象:

```java
SimpleJdbcCall simpleJdbcCall = new SimpleJdbcCall(dataSource)
		                     .withProcedureName("READ_EMPLOYEE"); 
```

```java
public Employee getEmployeeUsingSimpleJdbcCall(int id) {
    SqlParameterSource in = new MapSqlParameterSource().addValue("in_id", id);
    Map<String, Object> out = simpleJdbcCall.execute(in);

    Employee emp = new Employee();
    emp.setFirstName((String) out.get("FIRST_NAME"));
    emp.setLastName((String) out.get("LAST_NAME"));

    return emp;
}
```

## 6。批量操作

另一个简单的用例是一起批处理多个操作。

### 6.1。基本批量操作使用`JdbcTemplate`

使用`JdbcTemplate`， `Batch Operations`可以通过`batchUpdate()` API 运行。

这里有趣的部分是简洁但非常有用的`BatchPreparedStatementSetter` 实现:

```java
public int[] batchUpdateUsingJdbcTemplate(List<Employee> employees) {
    return jdbcTemplate.batchUpdate("INSERT INTO EMPLOYEE VALUES (?, ?, ?, ?)",
        new BatchPreparedStatementSetter() {
            @Override
            public void setValues(PreparedStatement ps, int i) throws SQLException {
                ps.setInt(1, employees.get(i).getId());
                ps.setString(2, employees.get(i).getFirstName());
                ps.setString(3, employees.get(i).getLastName());
                ps.setString(4, employees.get(i).getAddress();
            }
            @Override
            public int getBatchSize() {
                return 50;
            }
        });
}
```

### 6.2。批量操作使用`NamedParameterJdbcTemplate`

我们还可以选择使用`NamedParameterJdbcTemplate`–`batchUpdate()`API 进行批处理操作。

这个 API 比前一个简单。因此，不需要实现任何额外的接口来设置参数，因为它有一个内部准备好的语句设置器来设置参数值。

相反，参数值可以作为`SqlParameterSource`的数组传递给`batchUpdate()`方法。

```java
SqlParameterSource[] batch = SqlParameterSourceUtils.createBatch(employees.toArray());
int[] updateCounts = namedParameterJdbcTemplate.batchUpdate(
    "INSERT INTO EMPLOYEE VALUES (:id, :firstName, :lastName, :address)", batch);
return updateCounts;
```

## 7。春天的 JDBC 和 Spring Boot

Spring Boot 为在关系数据库中使用 JDBC 提供了一个开端。

与每个 Spring Boot 入门者一样，这个帮助我们快速启动并运行我们的应用程序。

### 7.1。Maven 依赖关系

我们需要将 [`spring-boot-starter-jdbc`](https://web.archive.org/web/20220630132202/https://search.maven.org/search?q=a:spring-boot-starter-jdbc) 依赖项作为主依赖项。我们还需要一个我们将使用的数据库的依赖项。在我们的例子中，这是`MySQL`:

```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-jdbc</artifactId>
</dependency>
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <scope>runtime</scope>
</dependency>
```

### 7.2。配置

Spring Boot 为我们自动配置了数据源。我们只需要在一个`properties`文件中提供属性:

```java
spring.datasource.url=jdbc:mysql://localhost:3306/springjdbc
spring.datasource.username=guest_user
spring.datasource.password=guest_password
```

仅此而已。我们的应用程序只需要做这些配置就可以启动并运行。我们现在可以将它用于其他数据库操作。

我们在上一节中看到的标准 Spring 应用程序的显式配置现在包含在 Spring Boot 自动配置中。

## 8。结论

在本文中，我们研究了 Spring 框架中的 JDBC 抽象。我们通过实例介绍了 Spring JDBC 提供的各种功能。

我们还研究了如何使用 Spring Boot·JDBC 首发快速开始春季 JDBC。

GitHub 上的[提供了示例的源代码。](https://web.archive.org/web/20220630132202/https://github.com/eugenp/tutorials/tree/master/persistence-modules/spring-jdbc)