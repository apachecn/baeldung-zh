# 使用 Spring Boot 配置光连接池

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-boot-hikari>

## 1。概述

[光](https://web.archive.org/web/20221102081808/https://github.com/brettwooldridge/HikariCP)是一个 JDBC `DataSource`实现，它提供了一个连接池机制。

与其他实现相比，它可能是轻量级的，并且性能更好。关于光的介绍，请看[这篇文章](/web/20221102081808/https://www.baeldung.com/hikaricp)。

这个快速教程展示了我们如何配置 Spring Boot 2 或 Spring Boot 1 应用程序来使用光`DataSource`。

## 延伸阅读:

## [JPA 中的一对一关系](/web/20221102081808/https://www.baeldung.com/jpa-one-to-one)

Learn three different ways to maintain a one-to-one relationship with JPA.[Read more](/web/20221102081808/https://www.baeldung.com/jpa-one-to-one) →

## [Spring Boot 与冬眠](/web/20221102081808/https://www.baeldung.com/spring-boot-hibernate)

A quick, practical intro to integrating Spring Boot and Hibernate/JPA.[Read more](/web/20221102081808/https://www.baeldung.com/spring-boot-hibernate) →

## 2。用 Spring Boot 2.x 配置光

在 Spring Boot 2 中，光是默认的数据源实现。

但是，要使用最新版本，我们需要在 pom.xml 中显式添加光依赖项:

```java
<dependency>
    <groupId>com.zaxxer</groupId>
    <artifactId>HikariCP</artifactId>
    <version>4.0.3</version>
</dependency>
```

这是 Spring Boot 1.x 的变化之处:

*   对光的依赖现在自动包含在`spring-boot-starter-data-jpa` 和`spring-boot-starter-jdbc`中。
*   自动确定`DataSource`实现的发现算法现在更喜欢光而不是 TomcatJDBC(参见[参考手册](https://web.archive.org/web/20221102081808/https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/))。

因此，如果我们想在基于 Spring Boot 2.x 的应用程序中使用光，除非我们想使用它的最新版本，否则我们无事可做。

## 3。调谐光配置参数

光相对于其他`DataSource`实现的优势之一是它提供了大量的配置参数。

我们可以通过使用前缀`spring.datasource.hikari`并附加光参数的名称来指定这些参数的值:

```java
spring.datasource.hikari.connectionTimeout=30000 
spring.datasource.hikari.idleTimeout=600000 
spring.datasource.hikari.maxLifetime=1800000 
...
```

在[光 GitHub 网站](https://web.archive.org/web/20221102081808/https://github.com/brettwooldridge/HikariCP#configuration-knobs-baby)和 [Spring 文档](https://web.archive.org/web/20221102081808/https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#spring.datasource.hikari)中可以找到所有光参数的列表和详细的解释。

## 4。用 Spring Boot 1.x 配置光

Spring Boot 1.x 默认使用 [Tomcat JDBC 连接池](https://web.archive.org/web/20221102081808/https://tomcat.apache.org/tomcat-8.5-doc/jdbc-pool.html)。

一旦我们将`spring-boot-starter-data-jpa`包含到我们的`pom.xml`中，我们将过渡性地包含对 Tomcat JDBC 实现的依赖。在运行期间，Spring Boot 将创建一个雄猫`DataSource`供我们使用。

要配置 Spring Boot 使用光连接池，我们有两个选择。

### 4.1。Maven 依赖关系

首先，我们需要在我们的`pom.xml`中包括对光的依赖:

```java
<dependency>
    <groupId>com.zaxxer</groupId>
    <artifactId>HikariCP</artifactId>
    <version>4.0.3</version>
</dependency>
```

最新版本可以在 [Maven Central](https://web.archive.org/web/20221102081808/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22com.zaxxer%22%20AND%20a%3A%22HikariCP%22) 上找到。

### 4.2。显式配置

告诉 Spring Boot 使用光的最安全的方法是显式配置数据源实现。

为此，我们只需**将属性`spring.datasource.type `设置为我们想要使用的**的`DataSource`实现的完全限定名:

```java
@RunWith(SpringRunner.class)
@SpringBootTest(
    properties = "spring.datasource.type=com.zaxxer.hikari.HikariDataSource"
)
public class HikariIntegrationTest {

    @Autowired
    private DataSource dataSource;

    @Test
    public void hikariConnectionPoolIsConfigured() {
        assertEquals("com.zaxxer.hikari.HikariDataSource", dataSource.getClass().getName());
    }
}
```

### 4.3。移除 Tomcat JDBC 依赖关系

第二个选择是让 Spring Boot 找到光的实现本身。

**如果 Spring Boot 在类路径中找不到 Tomcat `DataSource`，它将自动寻找下一个光`DataSource`。**发现算法在[参考手册](https://web.archive.org/web/20221102081808/https://docs.spring.io/spring-boot/docs/1.5.15.RELEASE/reference/htmlsingle/#boot-features-connect-to-production-database)中有描述。

要从类路径中删除 Tomcat 连接池，我们可以在我们的`pom.xml`中排除它:

```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
    <exclusions>
        <exclusion>
            <groupId>org.apache.tomcat</groupId>
            <artifactId>tomcat-jdbc</artifactId>
         </exclusion>
     </exclusions>
</dependency>
```

现在，上一节中的测试也可以在不设置`spring.datasource.type` 属性的情况下工作。

## 5。结论

在本文中，我们在 Spring Boot 2.x 应用程序中配置了光`DataSource`实现。我们学会了如何利用 Spring Boot 的自动配置。

我们还了解了在使用 Spring Boot 1.x 时配置光所需的更改

Spring Boot 1.x 示例的代码在[这里](https://web.archive.org/web/20221102081808/https://github.com/eugenp/tutorials/tree/master/spring-4)可用，Spring Boot 2.x 示例的代码在[这里](https://web.archive.org/web/20221102081808/https://github.com/eugenp/tutorials/tree/master/spring-5)可用。