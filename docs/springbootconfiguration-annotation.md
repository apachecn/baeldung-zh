# Spring Boot @ spring boot 配置指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/springbootconfiguration-annotation>

## 1.概观

在本教程中，我们将简要讨论 [`@SpringBootConfiguration`](https://web.archive.org/web/20220827110142/https://docs.spring.io/spring-boot/docs/current/api/org/springframework/boot/SpringBootConfiguration.html) 注释。我们还将看看它在 Spring Boot 应用程序中的用法。

## 2.Spring Boot 应用程序配置

**`@SpringBootConfiguration`是一个类级注释**，它是 Spring Boot 框架的一部分。它**表示一个类提供应用配置**。

Spring Boot 喜欢基于 Java 的配置。因此，`@SpringBootConfiguration`注释是应用程序中配置的主要来源。一般来说，定义`main()`方法的类是这个注释的一个很好的候选。

### 2.1.`@SpringBootConfiguration`

大多数 Spring Boot 使用`@SpringBootConfiguration` via [`@SpringBootApplication`](/web/20220827110142/https://www.baeldung.com/spring-boot-annotations) ，一种继承自它的注解。如果一个应用程序使用了`@SpringBootApplication`，那么它已经在使用`@SpringBootConfiguration`。

让我们看看`@SpringBootConfiguration's`在应用程序中的用法。

首先，我们创建一个包含我们的配置的应用程序类:

```java
@SpringBootConfiguration
public class Application {

    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }

    @Bean
    public PersonService personService() {
        return new PersonServiceImpl();
    }
}
```

`@SpringBootConfiguration`注释注释了`Application`类。这向 Spring 容器表明**该类有`@Bean`定义方法**。换句话说，它包含实例化和配置依赖关系的方法。

例如，`Application`类包含了`PersonService` bean 的 bean 定义方法。

此外，容器处理配置类。这反过来为应用程序生成 beans。因此，我们现在可以像使用`@Autowired`或`@Inject`一样使用[依赖注入](/web/20220827110142/https://www.baeldung.com/inversion-control-and-dependency-injection-in-spring)注释。

### 2.2.`@SpringBootConfiguration`对`@Configuration`

`@SpringBootConfiguration`是对 [`@Configuration`](/web/20220827110142/https://www.baeldung.com/spring-bean-annotations) 的替代注释。主要区别在于`@SpringBootConfiguration`允许自动定位配置。这对于单元或集成测试尤其有用。

建议**只有一个`@SpringBootConfiguration`或`@SpringBootApplication`** 供您应用。大多数应用程序将简单地使用`@SpringBootApplication.`

## 3.结论

在本文中，我们快速浏览了一下`@SpringBootConfiguration`注释。此外，我们查看了`@SpringBootConfiguration`在 Spring Boot 应用程序中的用法。我们还复习了 Spring 的`[@Bean](/web/20220827110142/https://www.baeldung.com/spring-core-annotations)`注释`.`

我们这里例子的完整源代码一如既往地在 GitHub 的[上。](https://web.archive.org/web/20220827110142/https://github.com/eugenp/tutorials/tree/master/spring-boot-modules/spring-boot-annotations-2)