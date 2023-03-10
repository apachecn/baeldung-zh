# 显示 Spring Boot 的 Hibernate/JPA SQL 语句

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/sql-logging-spring-boot>

## 1。概述

Spring [JDBC](/web/20220803134023/https://www.baeldung.com/spring-jdbc-jdbctemplate) 和 [JPA](/web/20220803134023/https://www.baeldung.com/the-persistence-layer-with-spring-data-jpa) 提供了对原生 JDBC API 的抽象，允许开发者放弃原生 SQL 查询。然而，出于调试目的，我们经常需要看到那些自动生成的 SQL 查询以及它们的执行顺序。

在这个快速教程中，我们将看看在 Spring Boot 记录这些 SQL 查询的不同方式。

## 延伸阅读:

## [春天的 JDBC](/web/20220803134023/https://www.baeldung.com/spring-jdbc-jdbctemplate)

Introduction to the Spring JDBC abstraction, with example on how to use the JbdcTempalte and NamedParameterJdbcTemplate APIs.[Read more](/web/20220803134023/https://www.baeldung.com/spring-jdbc-jdbctemplate) →

## [Spring Data JPA 简介](/web/20220803134023/https://www.baeldung.com/the-persistence-layer-with-spring-data-jpa)

Introduction to Spring Data JPA with Spring 4 - the Spring config, the DAO, manual and generated queries and transaction management.[Read more](/web/20220803134023/https://www.baeldung.com/the-persistence-layer-with-spring-data-jpa) →

## [冬眠拦截器](/web/20220803134023/https://www.baeldung.com/hibernate-interceptor)

A quick and practical guide to creating Hibernate interceptors.[Read more](/web/20220803134023/https://www.baeldung.com/hibernate-interceptor) →

## 2。记录 JPA 查询

### 2.1。至标准输出

将查询转储到标准输出的最简单方法是将以下内容添加到`application.properties`:

```java
spring.jpa.show-sql=true
```

为了美化或漂亮地打印 SQL，我们可以添加:

```java
spring.jpa.properties.hibernate.format_sql=true
```

虽然这非常简单，**但不推荐**，因为它直接将所有内容卸载到标准输出，而没有对日志框架进行任何优化。

此外，**它不记录预准备语句的参数。**

### 2.2。通过记录器

现在让我们看看如何通过在属性文件中配置记录器来记录 SQL 语句:

```java
logging.level.org.hibernate.SQL=DEBUG
logging.level.org.hibernate.type.descriptor.sql.BasicBinder=TRACE
```

第一行记录 SQL 查询，第二条语句记录准备好的语句参数。

pretty print 属性也可以在这种配置下工作。

通过设置这些属性，**日志将被发送到已配置的 appender。**默认情况下，Spring Boot 使用带有标准输出附加器的`logback`。

## 3。日志记录`JdbcTemplate`查询

要在使用`JdbcTemplate`时配置语句日志记录，我们需要以下属性:

```java
logging.level.org.springframework.jdbc.core.JdbcTemplate=DEBUG
logging.level.org.springframework.jdbc.core.StatementCreatorUtils=TRACE
```

与 JPA 日志记录配置类似，第一行用于记录语句，第二行用于记录准备好的语句的参数。

## 4。它是如何工作的？

**Spring/Hibernate 类，**生成 SQL 语句并设置参数，**已经包含了记录它们的代码。**

但是，这些日志语句的级别分别设置为`DEBUG`和`TRACE`，低于 Spring Boot 的默认级别— `INFO`。

通过添加这些属性，我们只是将这些记录器设置到所需的级别。

## 5。结论

在这篇短文中，我们研究了在 Spring Boot 记录 SQL 查询的方法。

如果我们选择[配置多个 appender](https://web.archive.org/web/20220803134023/https://logback.qos.ch/manual/appenders.html)，我们也可以将 SQL 语句和其他日志语句分离到不同的日志文件中，以保持整洁。