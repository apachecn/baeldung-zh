# 使用 Spring Boot 配置设置 MySQL JDBC 时区

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/mysql-jdbc-timezone-spring-boot>

## 1.概观

有时，当我们在 MySQL 中存储日期时，我们意识到来自数据库的日期与我们的系统或 JVM 不同。

其他时候，我们只需要用另一个时区运行我们的应用程序。

在本教程中，我们将看到使用 Spring Boot 配置改变 MySQL 时区的不同方法**。**

## 2.时区作为 URL 参数

我们可以指定时区的一种方法是在连接 URL 字符串中作为参数。

为了选择我们的时区，我们必须添加`connectionTimeZone`属性来指定时区:

```java
spring:
  datasource:
    url: jdbc:mysql://localhost:3306/test?connectionTimeZone=UTC
    username: root
    password:
```

当然，我们也可以用 Java 配置来配置数据源。

我们可以在 MySQL 官方文档中找到关于这个属性和其他属性的更多信息。

## 3.Spring Boot 房产

或者，我们可以在 Spring Boot 配置中指定`time_zone` 属性，而不是通过`connectionTimeZone` URL 参数来指示时区:

```java
spring.jpa.properties.hibernate.jdbc.time_zone=UTC
```

或者和 YAML 一起:

```java
spring:
  jpa:
    properties:
      hibernate:
        jdbc:
          time_zone: UTC
```

## 4.JVM 默认时区

当然，我们可以更新 Java 的默认时区。

为了选择我们的时区，我们必须在 URL 中添加属性`forceConnectionTimeZoneToSession=true` 。然后我们只需要添加一个简单的方法:

```java
@PostConstruct
void started() {
  TimeZone.setDefault(TimeZone.getTimeZone("UTC"));
}
```

但是，这种解决方案可能会产生其他问题，因为它是应用程序范围的。也许应用程序的其他部分需要另一个时区。例如，我们可能需要连接到不同的数据库，出于某种原因，它们需要将日期存储在不同的时区。

## 5.结论

在本教程中，我们看到了在 Spring 中配置 MySQL JDBC 时区的几种不同方法。我们用一个 URL 参数、一个属性并通过改变 JVM 的默认时区来实现。

和往常一样，GitHub 上的[是完整的例子集。](https://web.archive.org/web/20220926192333/https://github.com/eugenp/tutorials/tree/master/persistence-modules/spring-boot-mysql)