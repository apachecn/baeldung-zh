# @EnableConfigurationProperties 指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-enable-config-properties>

## 1.介绍

在这个快速教程中，**我们将展示如何在 [`@ConfigurationProperties`](/web/20220827110142/https://www.baeldung.com/configuration-properties-in-spring-boot) 注释类**中使用`@EnableConfigurationProperties`注释。

## 2.`@EnableConfigurationProperties`注释的目的

**`@EnableConfigurationProperties`标注严格连接到`@ConfiguratonProperties.`**

它在我们的应用程序中支持`@ConfigurationProperties`带注释的类。然而，值得指出的是， [Spring Boot 文档](https://web.archive.org/web/20220827110142/https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#boot-features-external-config-typesafe-configuration-properties)说，**每个项目自动包含`@EnableConfigurationProperties.`** 因此，`@ConfiguratonProperties`支持在每个 Spring Boot 应用程序中被隐式打开。

为了在我们的项目中使用一个配置类，我们需要将它注册为一个常规的 Spring bean。

首先，我们可以用`@Component.`来注释这样一个类，或者，我们可以使用一个`@Bean`工厂方法。

然而，在某些情况下，**我们可能更喜欢保留一个`@ConfigurationProperties`类作为简单的 POJO** 。这就是`@EnableConfigurationProperties`派上用场的时候了。我们可以在这个注释上直接指定所有的配置 beans。

**这是一种快速注册`@ConfigurationProperties`带注释的 beans 的便捷方式。**

## 3.使用`@EnableConfigurationProperties`

现在，让我们看看如何在实践中使用`@EnableConfigurationProperties`。

首先，我们需要定义我们的示例配置类:

```java
@ConfigurationProperties(prefix = "additional")
public class AdditionalProperties {

    private String unit;
    private int max;

    // standard getters and setters
}
```

**注意，我们仅用`@ConfigurationProperties.`** 注释了`AdditionalProperties`，它仍然是一个简单的 POJO！

最后，让我们使用@ `EnableConfigurationProperties`注册我们的配置 bean:

```java
@Configuration
@EnableConfigurationProperties(AdditionalProperties.class)
public class AdditionalConfiguration {

    @Autowired
    private AdditionalProperties additionalProperties;

    // make use of the bound properties
}
```

仅此而已！**我们现在可以像使用任何其他春豆一样使用 `AdditionalProperties`。**

## 4.结论

在这个快速教程中，我们向**展示了一种在 Spring 中快速注册@ `ConfigurationProperties`注释类的便捷方法。**

和往常一样，本文中使用的所有例子都可以在 GitHub 上找到。