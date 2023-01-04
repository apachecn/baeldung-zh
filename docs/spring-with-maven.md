# 和 Maven 一起的春天

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-with-maven>

**目录**

1.  [概述](#overview)
2.  【Maven 的基本 Spring 依赖关系
3.  [春天与 Maven 的坚持](#persistence)
4.  [Spring MVC with Maven](#mvc)
5.  [春天安全与 Maven](#security)
6.  【Maven 的春季测试
7.  [使用里程碑](#milestones)
8.  [使用快照](#snapshots)
9.  [结论](#conclusion)

## 1。概述

本教程演示了如何通过 Maven 设置**Spring 依赖关系。最新的春季发布可以在 Maven Central 上找到[。](https://web.archive.org/web/20221122114820/https://search.maven.org/classic/#search|ga|1|g%3A%22org.springframework%22 "Spring on Maven Central")**

## 2。Maven 的基本 Spring 依赖关系

Spring 被设计成高度模块化——使用 Spring 的一部分不应该也不需要另一部分。例如，基本的 Spring 上下文可以没有持久性或 MVC Spring 库。

让我们从一个基本的 **Maven 设置**开始，它将只使用**和`spring-context`依赖关系**:

```java
<properties>
    <org.springframework.version>5.2.8.RELEASE</org.springframework.version>
</properties>
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-context</artifactId>
    <version>${org.springframework.version}</version>
    <scope>runtime</scope>
</dependency>
```

这个依赖项——`**spring-context**`——定义了实际的 Spring Injection 容器，有少量的依赖项:`spring-core`、`spring-expression`、`spring-aop`和`spring-beans`。这些通过启用对一些**核心 Spring 技术**的支持来扩充容器:核心 Spring 实用程序、 [Spring 表达式语言](https://web.archive.org/web/20221122114820/https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#expressions "SpEL in the Reference") (SpEL)、[面向方面编程](https://web.archive.org/web/20221122114820/https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#aop "Spring AOP in the Reference")支持和 [JavaBeans 机制](https://web.archive.org/web/20221122114820/https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-definition "Spring Beans in the Reference")。

注意，我们在 **`runtime`范围**中定义依赖——这将确保在任何特定于 Spring 的 API 上都没有编译时依赖。对于更高级的用例，`runtime`范围可能会从一些选定的 Spring 依赖项中移除，但是对于更简单的项目，**不需要针对 Spring** 进行编译来充分利用框架。

还要注意，JDK 8 是 Spring 5.2 所需的最低 Java 版本。它还支持 JDK 11 作为当前的 LTS 分支，支持 JDK 13 作为最新的 OpenJDK 版本。

## 3。Maven 的 Spring Persistence】

现在让我们来看看**持久化的 Spring 依赖关系**——主要是`spring-orm`:

```java
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-orm</artifactId>
    <version>${org.springframework.version}</version>
</dependency>
```

这附带了 Hibernate 和 JPA 支持——比如`HibernateTemplate`和`JpaTemplate`——以及一些额外的、与持久性相关的依赖项:`spring-jdbc`和`spring-tx`。

JDBC 数据访问库定义了 [Spring JDBC 支持](https://web.archive.org/web/20221122114820/https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/data-access.html#jdbc "Spring JDBC in the Reference")以及`JdbcTemplate`，`spring-tx`代表了极其灵活的[事务管理抽象](https://web.archive.org/web/20221122114820/https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/data-access.html#transaction "Spring Transactions in the Reference")。

## 4。带 Maven 的 spring MVC

为了使用 Spring Web 和 Servlet 支持，除了上面的核心依赖项之外，还需要在`pom`中包含两个依赖项:

```java
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-web</artifactId>
    <version>${org.springframework.version}</version>
</dependency>
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-webmvc</artifactId>
    <version>${org.springframework.version}</version>
</dependency>
```

`spring-web`依赖包含 Servlet 和 Portlet 环境的通用 web 特定实用程序，而 **`spring-webmvc`为 Servlet 环境启用 MVC 支持**。

由于`spring-webmvc`有`spring-web`作为依赖项，使用`spring-webmvc`时不需要显式定义`spring-web`。

从 Spring 5.0 开始，为了支持反应式堆栈 web 框架，我们可以添加对 [Spring WebFlux](https://web.archive.org/web/20221122114820/https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web-reactive.html#spring-webflux) 的依赖:

```java
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-webflux</artifactId>
    <version>${org.springframework.version}</version>
</dependency> 
```

## 5。Maven 的 Spring Security】

**安全 Maven 依赖性**在 [Spring Security with Maven](/web/20221122114820/https://www.baeldung.com/spring-security-with-maven "Spring Security with Maven") 文章中有深入讨论。

## 6。Maven 的春季测试

Spring 测试框架可以通过以下依赖项包含在项目中:

```java
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-test</artifactId>
    <version>${spring.version}</version>
    <scope>test</scope>
</dependency>
```

使用 Spring 5，我们也可以执行[并发测试执行](/web/20221122114820/https://www.baeldung.com/spring-5-concurrent-tests)。

## 7。使用里程碑

Spring 的发布版本托管在 Maven Central 上。然而，如果一个项目需要使用里程碑版本，那么需要向 pom 添加一个定制的 Spring 存储库:

```java
<repositories>
    <repository>
        <id>repository.springframework.maven.milestone</id>
        <name>Spring Framework Maven Milestone Repository</name>
        <url>http://repo.spring.io/milestone/</url>
    </repository>
</repositories>
```

一旦定义了这个存储库，项目就可以定义依赖项，例如:

```java
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-core</artifactId>
    <version>5.3.0-M1</version>
</dependency>
```

## 8。使用快照

与里程碑类似，快照托管在自定义存储库中:

```java
<repositories>
    <repository>
        <id>repository.springframework.maven.snapshot</id>
        <name>Spring Framework Maven Snapshot Repository</name>
        <url>http://repo.spring.io/snapshot/</url>
    </repository>
</repositories>
```

一旦在 pom.xml 中启用了快照存储库，就可以引用以下依赖项:

```java
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-core</artifactId>
    <version>4.0.3.BUILD-SNAPSHOT</version>
</dependency>
```

以及–对于 5.x:

```java
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-core</artifactId>
    <version>5.3.0-SNAPSHOT</version>
</dependency>
```

## 9。结论

本文讨论了在 Maven 中使用 **Spring 的实际细节。这里介绍的 Maven 依赖项当然是一些主要的依赖项，还有几个可能值得一提，但还没有达到标准。然而，这应该是在项目中使用 Spring 的一个好的起点。**