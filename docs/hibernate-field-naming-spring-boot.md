# 用 Spring Boot 命名休眠字段

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/hibernate-field-naming-spring-boot>

## 1。简介

在这个简短的教程中，我们将看到如何在一个 [Spring Boot](/web/20221206183643/https://www.baeldung.com/spring-boot) 应用程序中使用 [Hibernate 命名策略](/web/20221206183643/https://www.baeldung.com/hibernate-naming-strategy)。

## 2。Maven 依赖关系

如果我们从一个基于 Maven 的 Spring Boot 应用程序开始，并且我们乐于接受 Spring 数据，那么我们只需要添加 Spring 数据 JPA 依赖关系:

```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
```

另外，我们需要一个数据库，所以我们将使用[内存数据库 H2](https://web.archive.org/web/20221206183643/https://search.maven.org/search?q=g:com.h2database%20AND%20a:h2) :

```java
<dependency>
    <groupId>com.h2database</groupId>
    <artifactId>h2</artifactId>
    <scope>runtime</scope>
</dependency>
```

Spring 数据 JPA 依赖为我们带来了 Hibernate 依赖。

## 3。Spring Boot 命名策略

**Hibernate 使用一个`physical strategy `和一个`implicit strategy. `** 映射字段名，我们之前在这个[教程](/web/20221206183643/https://www.baeldung.com/hibernate-naming-strategy)中讨论过如何使用命名策略。

和 Spring Boot 为这两者提供了默认值:

*   `spring.jpa.hibernate.naming.physical-strategy`默认为`org.springframework.boot.orm.jpa.hibernate.SpringPhysicalNamingStrategy`，并且
*   `spring.jpa.hibernate.naming.implicit-strategy `默认为`org.springframework.boot.orm.jpa.hibernate.SpringImplicitNamingStrategy`

我们可以覆盖这些值，但默认情况下，这些值将:

*   用下划线替换点
*   把骆驼箱换成蛇箱，还有
*   小写表名

例如，一个`AddressBook `实体将被创建为`address_book `表。

## 4。行动中的战略

如果我们创建一个`Account`实体:

```java
@Entity
public class Account {
    @Id 
    private Long id;
    private String defaultEmail;
}
```

然后在我们的属性文件中打开一些 SQL 调试:

```java
hibernate.show_sql: true
```

启动时，我们会在日志中看到下面的`create`语句:

```java
Hibernate: create table account (id bigint not null, default_email varchar(255))
```

正如我们所看到的，字段使用了 snake case 和小写，遵循了 Spring 约定。

**记住，在这种情况下我们不需要指定`physical-strategy`或`implicit-strategy `属性，因为我们接受默认值。**

## 5。定制策略

因此，假设我们想要使用 JPA 1.0 中的策略。

我们只需要在我们的属性文件中指明它:

```java
spring:
  jpa:
    hibernate:
      naming:
        physical-strategy: org.hibernate.boot.model.naming.PhysicalNamingStrategyStandardImpl
        implicit-strategy: org.hibernate.boot.model.naming.ImplicitNamingStrategyLegacyJpaImpl
```

或者，将它们暴露为:

```java
@Bean
public PhysicalNamingStrategy physical() {
    return new PhysicalNamingStrategyStandardImpl();
}

@Bean
public ImplicitNamingStrategy implicit() {
    return new ImplicitNamingStrategyLegacyJpaImpl();
}
```

不管怎样，如果我们用这些变化运行我们的例子，我们将看到一个稍微不同的 DDL 语句:

```java
Hibernate: create table Account (id bigint not null, defaultEmail varchar(255), primary key (id))
```

正如我们所看到的，这次这些策略遵循了 JPA 1.0 的命名约定。

**现在，如果我们想使用 JPA 2.0 命名规则，我们需要将`ImplicitNamingStrategyJpaCompliantImpl`设置为隐式策略。**

## 6。结论

在本教程中，我们看到了 Spring Boot 如何将命名策略应用于 Hibernate 的查询生成器，我们还看到了如何定制这些策略。

一如既往，请务必在 GitHub 上查看所有这些样本[。](https://web.archive.org/web/20221206183643/https://github.com/eugenp/tutorials/tree/master/persistence-modules/spring-boot-persistence)