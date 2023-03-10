# 禁用 Spring 数据自动配置

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-data-disable-auto-config>

## 1。简介

在这个快速教程中，我们将探索在 Spring Boot 中禁用数据库自动配置的两种不同方法。这在测试时可以派上用场[。](/web/20221211143532/https://www.baeldung.com/spring-boot-exclude-auto-configuration-test)

我们将举例说明 Redis、MongoDB 和 Spring Data JPA。

我们将从基于注释的方法开始，然后我们将研究属性文件方法。

## 2。禁用注释

让我们从 [MongoDB](/web/20221211143532/https://www.baeldung.com/spring-data-mongodb-tutorial) 的例子开始。我们将看看需要排除的类:

```java
@SpringBootApplication(exclude = {
    MongoAutoConfiguration.class, 
    MongoDataAutoConfiguration.class
})
```

类似地，我们将研究禁用 [Redis](/web/20221211143532/https://www.baeldung.com/spring-data-redis-tutorial) 的自动配置:

```java
@SpringBootApplication(exclude = {
    RedisAutoConfiguration.class, 
    RedisRepositoryAutoConfiguration.class
})
```

最后，我们来看看如何禁用 [Spring 数据 JPA](/web/20221211143532/https://www.baeldung.com/the-persistence-layer-with-spring-data-jpa) 的自动配置:

```java
@SpringBootApplication(exclude = {
    DataSourceAutoConfiguration.class, 
    DataSourceTransactionManagerAutoConfiguration.class, 
    HibernateJpaAutoConfiguration.class
})
```

## 3。禁止使用属性文件

我们还可以使用属性文件禁用自动配置。

我们将首先用 MongoDB 来探索它:

```java
spring.autoconfigure.exclude= \
  org.springframework.boot.autoconfigure.mongo.MongoAutoConfiguration, \
  org.springframework.boot.autoconfigure.data.mongo.MongoDataAutoConfiguration
```

现在我们将为 Redis 禁用它:

```java
spring.autoconfigure.exclude= \
  org.springframework.boot.autoconfigure.data.redis.RedisAutoConfiguration, \
  org.springframework.boot.autoconfigure.data.redis.RedisRepositoriesAutoConfiguration
```

最后，我们将对 Spring Data JPA 禁用它:

```java
spring.autoconfigure.exclude= \ 
  org.springframework.boot.autoconfigure.jdbc.DataSourceAutoConfiguration, \
  org.springframework.boot.autoconfigure.orm.jpa.HibernateJpaAutoConfiguration, \
  org.springframework.boot.autoconfigure.jdbc.DataSourceTransactionManagerAutoConfiguration
```

## 4。测试

为了测试，我们将检查自动配置的类的 Spring beans 在我们的[应用程序上下文](/web/20221211143532/https://www.baeldung.com/spring-web-contexts)中是否不存在。

我们将从 MongoDB 的测试开始。我们将验证`MongoTemplate` bean 是否不存在:

```java
@Test(expected = NoSuchBeanDefinitionException.class)
public void givenAutoConfigDisabled_whenStarting_thenNoAutoconfiguredBeansInContext() { 
    context.getBean(MongoTemplate.class); 
}
```

现在我们将检查 JPA。对于 JPA，将缺少`DataSource` bean:

```java
@Test(expected = NoSuchBeanDefinitionException.class)
public void givenAutoConfigDisabled_whenStarting_thenNoAutoconfiguredBeansInContext() {
    context.getBean(DataSource.class);
}
```

最后，对于 Redis，我们将检查应用程序上下文中的`RedisTemplate` bean:

```java
@Test(expected = NoSuchBeanDefinitionException.class)
public void givenAutoConfigDisabled_whenStarting_thenNoAutoconfiguredBeansInContext() {
    context.getBean(RedisTemplate.class);
}
```

## 5。结论

在这篇简短的文章中，我们学习了如何为不同的数据库禁用 Spring Boot 自动配置。

本文中所有示例的源代码都可以在 [GitHub](https://web.archive.org/web/20221211143532/https://github.com/eugenp/tutorials/tree/master/spring-boot-modules/spring-boot-data) 上获得。