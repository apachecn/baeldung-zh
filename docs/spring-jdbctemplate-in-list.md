# 在 JdbcTemplate IN 子句中使用值列表

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-jdbctemplate-in-list>

## 1。简介

在 SQL 语句中，我们可以使用 In 操作符来测试表达式是否匹配列表中的任何值。因此，我们可以使用 IN 运算符来代替多个 or 条件。

在本教程中，我们将学习如何将一列值传递到一个 [Spring JDBC 模板](/web/20220625172019/https://www.baeldung.com/spring-jdbc-jdbctemplate)查询的 In 子句中。

## 2。向`IN`子句传递一个`List`参数

IN 操作符允许我们在 WHERE 子句中指定多个值。例如，我们可以使用它来查找 id 在指定 id 列表中的所有雇员:

```java
SELECT * FROM EMPLOYEE WHERE id IN (1, 2, 3)
```

通常，IN 子句中值的总数是可变的。因此，我们需要创建一个能够支持动态值列表的占位符。

### 2.1。`JdbcTemplate`同

用 [`JdbcTemplate`](https://web.archive.org/web/20220625172019/https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/jdbc/core/JdbcTemplate.html) ，我们可以用“？”字符作为值列表的占位符。“？”的数量字符将与列表的大小相同:

```java
List<Employee> getEmployeesFromIdList(List<Integer> ids) {
    String inSql = String.join(",", Collections.nCopies(ids.size(), "?"));

    List<Employee> employees = jdbcTemplate.query(
      String.format("SELECT * FROM EMPLOYEE WHERE id IN (%s)", inSql), 
      ids.toArray(), 
      (rs, rowNum) -> new Employee(rs.getInt("id"), rs.getString("first_name"), 
        rs.getString("last_name")));

    return employees;
} 
```

在这个方法中，我们首先生成一个包含`ids.size()` ['？'的占位符字符串用逗号分隔的字符](/web/20220625172019/https://www.baeldung.com/java-strings-concatenation)。然后，我们将这个字符串放入 SQL 语句的 IN 子句中。例如，如果我们在`ids`列表中有三个数字，SQL 语句是:

```java
SELECT * FROM EMPLOYEE WHERE id IN (?,?,?)
```

在`query` 方法中，我们将`ids`列表作为参数传递，以匹配 In 子句中的占位符。这样，我们可以根据输入值列表执行动态 SQL 语句。

### 2.2。`NamedParameterJdbcTemplate`同

另一种处理动态值列表的方法是使用 [`NamedParameterJdbcTemplate`](https://web.archive.org/web/20220625172019/https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/jdbc/core/namedparam/NamedParameterJdbcTemplate.html) 。例如，我们可以直接为输入列表创建一个命名参数:

```java
List<Employee> getEmployeesFromIdListNamed(List<Integer> ids) {
    SqlParameterSource parameters = new MapSqlParameterSource("ids", ids);

    List<Employee> employees = namedJdbcTemplate.query(
      "SELECT * FROM EMPLOYEE WHERE id IN (:ids)", 
      parameters, 
      (rs, rowNum) -> new Employee(rs.getInt("id"), rs.getString("first_name"),
        rs.getString("last_name")));

    return employees;
}
```

在这个方法中，我们首先构造一个包含输入 id 列表的 [`MapSqlParameterSource`](https://web.archive.org/web/20220625172019/https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/jdbc/core/namedparam/MapSqlParameterSource.html) 对象。那么我们只使用一个命名参数来表示值的动态列表。

在幕后，`NamedParameterJdbcTemplate`用命名参数代替“？”占位符，并使用`JdbcTemplate`来执行查询。

## 3。`List`搬运大型

当我们在一个列表中有大量的值时，我们应该考虑将它们传递到`JdbcTemplate`查询中的替代方法。

例如，Oracle 数据库不支持 in 子句中超过 1，000 个文字。

一种方法是**为列表**创建一个临时表。然而，不同的数据库可以有不同的方法来创建临时表。例如，我们可以使用`[CREATE GLOBAL TEMPORARY TABLE](https://web.archive.org/web/20220625172019/https://docs.oracle.com/cd/B28359_01/server.111/b28310/tables003.htm#ADMIN11633)`语句在 Oracle 数据库中创建一个临时表。

让我们为 H2 数据库创建一个临时表:

```java
List<Employee> getEmployeesFromLargeIdList(List<Integer> ids) {
    jdbcTemplate.execute("CREATE TEMPORARY TABLE IF NOT EXISTS employee_tmp (id INT NOT NULL)");

    List<Object[]> employeeIds = new ArrayList<>();
    for (Integer id : ids) {
        employeeIds.add(new Object[] { id });
    }
    jdbcTemplate.batchUpdate("INSERT INTO employee_tmp VALUES(?)", employeeIds);

    List<Employee> employees = jdbcTemplate.query(
      "SELECT * FROM EMPLOYEE WHERE id IN (SELECT id FROM employee_tmp)", 
      (rs, rowNum) -> new Employee(rs.getInt("id"), rs.getString("first_name"),
      rs.getString("last_name")));

    jdbcTemplate.update("DELETE FROM employee_tmp");

    return employees;
}
```

这里，我们首先创建一个临时表来保存输入列表的所有值。然后，我们将输入列表的值插入到表中。

在我们生成的 SQL 语句中，**In 子句中的值来自临时表**，我们避免构建带有大量占位符的 IN 子句。

最后，在完成查询后，我们可以清理临时表以备将来使用。

## 4。结论

在本文中，我们演示了如何使用`JdbcTemplate` 和`NamedParameterJdbcTemplate` 为 SQL 查询的 In 子句传递一组值。我们还提供了另一种方法，通过使用临时表来处理大量的列表值。

和往常一样，这篇文章的源代码可以在 GitHub 的[上找到。](https://web.archive.org/web/20220625172019/https://github.com/eugenp/tutorials/tree/master/persistence-modules/spring-jdbc)