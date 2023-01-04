# Spring 非 TransientDataAccessException 指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/nontransientdataaccessexception>

## 1。概述

在这个快速教程中，我们将浏览最重要的常见`NonTransientDataAccessException`类型，并举例说明。

## 2。基本异常类

这个主异常类的子类表示与数据访问相关的异常，这些异常被认为是非暂时的或永久的。

简而言之，这意味着——在根本原因被修复之前——对导致异常的方法的所有未来尝试都将失败。

## 3。`DataIntegrityViolationException`

当试图修改数据导致违反完整性约束时，抛出`NonTransientDataAccessException`的这个子类型。

在我们的`Foo`类的例子中，name 列被定义为不允许使用`null` 值:

```java
@Column(nullable = false)
private String name;
```

如果我们试图保存一个实例而不设置名称的值，我们可以预期会抛出一个`DataIntegrityViolationException`:

```java
@Test(expected = DataIntegrityViolationException.class)
public void whenSavingNullValue_thenDataIntegrityException() {
    Foo fooEntity = new Foo();
    fooService.create(fooEntity);
}
```

### 3.1。`DuplicateKeyException`

`DataIntegrityViolationException`的一个子类是`DuplicateKeyException`，当试图保存一个主键已经存在的记录或者一个值已经存在于具有`unique`约束的列中时，比如试图在`foo`表中插入两行具有相同的`id`1:

```java
@Test(expected = DuplicateKeyException.class)
public void whenSavingDuplicateKeyValues_thenDuplicateKeyException() {
    JdbcTemplate jdbcTemplate = new JdbcTemplate(restDataSource);
    jdbcTemplate.execute("insert into foo(id,name) values (1,'a')");
    jdbcTemplate.execute("insert into foo(id,name) values (1,'b')");
}
```

## 4。`DataRetrievalFailureException`

当检索数据过程中出现问题时，会引发此异常，例如查找带有数据库中不存在的标识符的对象。

例如，我们将使用`JdbcTemplate`类，它有一个抛出这个异常的方法:

```java
@Test(expected = DataRetrievalFailureException.class)
public void whenRetrievingNonExistentValue_thenDataRetrievalException() {
    JdbcTemplate jdbcTemplate = new JdbcTemplate(restDataSource);

    jdbcTemplate.queryForObject("select * from foo where id = 3", Integer.class);
}
```

### 4.1。`IncorrectResultSetColumnCountException`

当试图从一个表中检索多个列而没有创建适当的`RowMapper`时，抛出这个异常子类:

```java
@Test(expected = IncorrectResultSetColumnCountException.class)
public void whenRetrievingMultipleColumns_thenIncorrectResultSetColumnCountException() {
    JdbcTemplate jdbcTemplate = new JdbcTemplate(restDataSource);

    jdbcTemplate.execute("insert into foo(id,name) values (1,'a')");
    jdbcTemplate.queryForList("select id,name from foo where id=1", Foo.class);
}
```

### 4.2。`IncorrectResultSizeDataAccessException`

当检索到的记录数与预期的记录数不同时，会引发此异常，例如，当预期只有一个`Integer`值，但为查询检索到两行时:

```java
@Test(expected = IncorrectResultSizeDataAccessException.class)
public void whenRetrievingMultipleValues_thenIncorrectResultSizeException() {
    JdbcTemplate jdbcTemplate = new JdbcTemplate(restDataSource);

    jdbcTemplate.execute("insert into foo(name) values ('a')");
    jdbcTemplate.execute("insert into foo(name) values ('a')");

    jdbcTemplate.queryForObject("select id from foo where name='a'", Integer.class);
}
```

## 5。`DataSourceLookupFailureException`

当无法获取指定的数据源时，将引发此异常。例如，我们将使用类`JndiDataSourceLookup`来寻找一个不存在的数据源:

```java
@Test(expected = DataSourceLookupFailureException.class)
public void whenLookupNonExistentDataSource_thenDataSourceLookupFailureException() {
    JndiDataSourceLookup dsLookup = new JndiDataSourceLookup();
    dsLookup.setResourceRef(true);
    DataSource dataSource = dsLookup.getDataSource("java:comp/env/jdbc/example_db");
}
```

## 6。`InvalidDataAccessResourceUsageException`

当不正确地访问资源时，例如当用户缺少`SELECT`权限时，就会抛出这个异常。

为了测试这个异常，我们需要撤销用户的`SELECT`权限，然后运行一个选择查询:

```java
@Test(expected = InvalidDataAccessResourceUsageException.class)
public void whenRetrievingDataUserNoSelectRights_thenInvalidResourceUsageException() {
    JdbcTemplate jdbcTemplate = new JdbcTemplate(restDataSource);
    jdbcTemplate.execute("revoke select from tutorialuser");

    try {
        fooService.findAll();
    } finally {
        jdbcTemplate.execute("grant select to tutorialuser");
    }
}
```

请注意，我们正在恢复用户在`finally`块中的权限。

### 6.1。`BadSqlGrammarException`

`InvalidDataAccessResourceUsageException` 的一个非常常见的子类型是`BadSqlGrammarException`，当试图用无效的 SQL 运行查询时抛出:

```java
@Test(expected = BadSqlGrammarException.class)
public void whenIncorrectSql_thenBadSqlGrammarException() {
    JdbcTemplate jdbcTemplate = new JdbcTemplate(restDataSource);
    jdbcTemplate.queryForObject("select * fro foo where id=3", Integer.class);
}
```

当然，请注意`fro –` ，这是查询的无效方面。

## 7。`CannotGetJdbcConnectionException`

当通过`JDBC`的连接尝试失败时，例如当数据库 url 不正确时，抛出该异常。如果我们像下面这样写 url:

```java
jdbc.url=jdbc:mysql:3306://localhost/spring_hibernate4_exceptions?createDatabaseIfNotExist=true
```

然后在试图执行一条语句时会抛出`CannotGetJdbcConnectionException`:

```java
@Test(expected = CannotGetJdbcConnectionException.class)
public void whenJdbcUrlIncorrect_thenCannotGetJdbcConnectionException() {
    JdbcTemplate jdbcTemplate = new JdbcTemplate(restDataSource);
    jdbcTemplate.execute("select * from foo");
}
```

## 8。结论

在这个非常中肯的教程中，我们看了一些最常见的`NonTransientDataAccessException`类的子类型。

所有例子的实现都可以在[GitHub 项目](https://web.archive.org/web/20220628105251/https://github.com/eugenp/tutorials/tree/master/spring-exceptions "NoSuchBeanDefinitionException examples")中找到。当然，所有的例子都使用了内存数据库，所以你不用做任何设置就可以轻松运行它们。