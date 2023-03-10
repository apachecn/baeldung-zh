# 解决“无法配置数据源”错误

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-boot-failed-to-configure-data-source>

## 1.概观

在这个简短的教程中，我们将讨论 Spring Boot 项目中`“Failed to configure a DataSource” error`的成因和解决方案。

我们将使用两种不同的方法来解决这个问题:

1.  定义数据源
2.  禁用数据源的自动配置

## 延伸阅读:

## [在 Spring Boot 以编程方式配置数据源](/web/20220906052622/https://www.baeldung.com/spring-boot-configure-data-source-programmatic)

Learn how to configure a Spring Boot DataSource programmatically, thereby side-stepping Spring Boot's automatic DataSource configuration algorithm.[Read more](/web/20220906052622/https://www.baeldung.com/spring-boot-configure-data-source-programmatic) →

## [为测试配置单独的 Spring 数据源](/web/20220906052622/https://www.baeldung.com/spring-testing-separate-data-source)

A quick, practical tutorial on how to configure a separate data source for testing in a Spring application.[Read more](/web/20220906052622/https://www.baeldung.com/spring-testing-separate-data-source) →

## [Spring Boot 与 H2 数据库](/web/20220906052622/https://www.baeldung.com/spring-boot-h2-database)

Learn how to configure and how to use the H2 database with Spring Boot.[Read more](/web/20220906052622/https://www.baeldung.com/spring-boot-h2-database) →

## 2.问题是

假设我们有一个 Spring Boot 项目，我们已经添加了`[spring-data-starter-jpa](https://web.archive.org/web/20220906052622/https://search.maven.org/classic/#search%7Cga%7C1%7Ca%3A%22spring-boot-starter-data-jpa%22%20g%3A%22org.springframework.boot%22)` [依赖](https://web.archive.org/web/20220906052622/https://search.maven.org/classic/#search%7Cga%7C1%7Ca%3A%22spring-boot-starter-data-jpa%22%20g%3A%22org.springframework.boot%22)和 [MySQL JDBC 驱动](https://web.archive.org/web/20220906052622/https://search.maven.org/classic/#search%7Cga%7C1%7Ca%3A%22mysql-connector-java%22%20g%3A%22mysql%22)到我们的`pom.xml`:

```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <scope>runtime</scope>
</dependency>
```

但是当我们运行该应用程序时，它失败了，并显示以下错误:

```java
Description:

Failed to configure a DataSource: 'url' attribute is not specified and no embedded 
  datasource could be configured.

Reason: Failed to determine a suitable driver class
```

让我们看看为什么会发生这种情况。

## 3.原因

根据设计，Spring Boot 自动配置试图根据添加到类路径中的依赖项自动配置 beans。

由于我们的类路径依赖于 JPA，Spring Boot 试图自动配置 JPA `DataSource`。问题是**我们没有给 Spring 执行自动配置所需的信息。**

例如，我们还没有定义任何 JDBC 连接属性，当使用 MySQL 和 MSSQL 等外部数据库时，我们需要这样做。另一方面，对于内存数据库，比如 H2，我们不会面临这个问题，因为它们可以在没有所有这些信息的情况下创建数据源。

## 4.解决方法

### 4.1.使用属性定义`DataSource`

由于问题是由于缺少数据库连接而出现的，我们可以通过提供数据源属性来解决问题。

首先，让我们用 [**在我们项目的`application.properties`文件**](/web/20220906052622/https://www.baeldung.com/spring-testing-separate-data-source) 中定义数据源属性:

```java
spring.datasource.url=jdbc:mysql://localhost:3306/myDb
spring.datasource.username=user1
spring.datasource.password=pass
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
```

或者我们可以在`application.yml` 中**提供数据源属性:**

```java
spring:
  datasource:
    driverClassName: com.mysql.cj.jdbc.Driver
    url: jdbc:mysql://localhost:3306/myDb
    username: user1
    password: pass
```

### 4.2.以编程方式定义`DataSource`

或者，我们可以通过**使用实用工具构建器类`DataSourceBuilder`以编程方式定义我们的数据源。**

我们需要提供数据库 URL、用户名、密码和 SQL 驱动程序信息来创建我们的数据源:

```java
@Configuration
public class DatasourceConfig {
    @Bean
    public DataSource datasource() {
        return DataSourceBuilder.create()
          .driverClassName("com.mysql.cj.jdbc.Driver")
          .url("jdbc:mysql://localhost:3306/myDb")
          .username("user1")
          .password("pass")
          .build();	
    }
}
```

简而言之，我们可以选择使用上面的任何一个选项来根据我们的需求配置数据源。

### 4.3.排除`DataSourceAutoConfiguration`

在上一节中，我们通过向项目中添加数据源属性解决了这个问题。

但是，如果我们还没有准备好定义我们的数据源，我们如何解决这个问题呢？让我们看看如何**阻止 Spring Boot 自动配置数据源。**

类`DataSourceAutoConfiguration`是使用`spring.datasource.*`属性配置数据源的基类。

现在，有几种方法可以让我们[将它从自动配置](/web/20220906052622/https://www.baeldung.com/spring-boot-exclude-auto-configuration-test)中排除。

首先，我们可以使用我们的`application.properties`文件中的`spring.autoconfigure.exclude` **属性****来禁用自动配置:**

```java
spring.autoconfigure.exclude=org.springframework.boot.autoconfigure.jdbc.DataSourceAutoConfiguration
```

我们可以使用我们的`application.yml`文件做同样的事情:

```java
spring:
  autoconfigure:
    exclude:
    - org.springframework.boot.autoconfigure.jdbc.DataSourceAutoConfiguration
```

或者我们可以**在我们的`@SpringBootApplication`或`@EnableAutoConfiguration`注释**上使用`exclude`属性:

```java
@SpringBootApplication(exclude={DataSourceAutoConfiguration.class})
```

在上面所有的例子中，我们**禁用了`DataSource`的自动配置。**这不会影响任何其他 beans 的自动配置。

因此，总而言之，我们可以使用上述任何一种方法来禁用 Spring Boot 的数据源自动配置。

理想情况下，我们应该提供数据源信息，并且只在测试时使用 exclude 选项。

## 5.结论

在本文中，我们已经看到了导致`“Failed to configure a DataSource”`错误的原因。

首先，我们通过定义数据源解决了这个问题。

接下来，我们讨论了如何在根本不配置数据源的情况下解决这个问题。

和往常一样，本文中使用的完整代码可以在 [GitHub](https://web.archive.org/web/20220906052622/https://github.com/eugenp/tutorials/tree/master/persistence-modules/spring-boot-persistence) 上获得。