# Spring Boot 应用 ContextRunner 指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-boot-context-runner>

## 1.概观

众所周知，[自动配置是 Spring Boot](/web/20221129013933/https://www.baeldung.com/spring-boot-custom-auto-configuration) 的关键特性之一，但是测试自动配置场景可能会很棘手。

在接下来的章节中，我们将展示 **`ApplicationContextRunner `如何简化自动配置测试。**

## 2.测试自动配置场景

**`ApplicationContextRunner`是一个运行`ApplicationContext`并提供 AssertJ 风格断言**的实用程序类。最好将**用作共享配置的测试类**中的一个字段，然后我们在每个测试中进行定制:

```java
private final ApplicationContextRunner contextRunner 
    = new ApplicationContextRunner(); 
```

让我们通过测试几个案例来展示它的魔力。

### 2.1.测试类别条件

在本节中，我们将通过**测试一些使用`@ConditionalOnClass`和`@ConditionalOnMissingClass`注释**的自动配置类:

```java
@Configuration
@ConditionalOnClass(ConditionalOnClassIntegrationTest.class)
protected static class ConditionalOnClassConfiguration {
    @Bean
    public String created() {
        return "This is created when ConditionalOnClassIntegrationTest "
               + "is present on the classpath";
    }
}

@Configuration
@ConditionalOnMissingClass(
    "com.baeldung.autoconfiguration.ConditionalOnClassIntegrationTest"
)
protected static class ConditionalOnMissingClassConfiguration {
    @Bean
    public String missed() {
        return "This is missed when ConditionalOnClassIntegrationTest "
               + "is present on the classpath";
    }
}
```

我们想测试在给定的预期条件下，自动配置是否正确地实例化或跳过了`created` 和`missed` bean。

*ApplicationContextRunner* 为我们提供了`withUserConfiguration` 方法，我们可以按需提供自动配置，为每个测试定制`ApplicationContext`。

`run`方法将一个`ContextConsumer` 作为将断言应用到上下文的参数。当测试退出时，`ApplicationContext`将自动关闭；

```java
@Test
public void whenDependentClassIsPresent_thenBeanCreated() {
    this.contextRunner.withUserConfiguration(ConditionalOnClassConfiguration.class)
        .run(context -> {
            assertThat(context).hasBean("created");
            assertThat(context.getBean("created"))
              .isEqualTo("This is created when ConditionalOnClassIntegrationTest "
                         + "is present on the classpath");
        });
}

@Test
public void whenDependentClassIsPresent_thenBeanMissing() {
    this.contextRunner.withUserConfiguration(ConditionalOnMissingClassConfiguration.class)
        .run(context -> {
            assertThat(context).doesNotHaveBean("missed");
        });
} 
```

通过前面的例子，我们看到了测试某个类出现在类路径中的场景的简单性。但是当类不在类路径中时，我们如何测试相反的情况呢？

这就是 *FilteredClassLoader* 发挥作用的地方。它用于在运行时过滤类路径上的指定类:

```java
@Test
public void whenDependentClassIsNotPresent_thenBeanMissing() {
    this.contextRunner.withUserConfiguration(ConditionalOnClassConfiguration.class)
        .withClassLoader(new FilteredClassLoader(ConditionalOnClassIntegrationTest.class))
        .run((context) -> {
            assertThat(context).doesNotHaveBean("created");
            assertThat(context).doesNotHaveBean(ConditionalOnClassIntegrationTest.class);
        });
}

@Test
public void whenDependentClassIsNotPresent_thenBeanCreated() {
    this.contextRunner.withUserConfiguration(ConditionalOnMissingClassConfiguration.class)
        .withClassLoader(new FilteredClassLoader(ConditionalOnClassIntegrationTest.class))
        .run((context) -> {
            assertThat(context).hasBean("missed");
            assertThat(context).getBean("missed")
              .isEqualTo("This is missed when ConditionalOnClassIntegrationTest "
                         + "is present on the classpath");
            assertThat(context).doesNotHaveBean(ConditionalOnClassIntegrationTest.class);
        });
}
```

### 2.2.测试 Bean 条件

我们刚刚看了测试`@ConditionalOnClass`和`@ConditionalOnMissingClass`注释，现在**让我们看看当我们使用`@ConditionalOnBean`和`@ConditionalOnMissingBean`注释时会是什么样子。**

作为开始，我们同样需要**一些自动配置类**:

```java
@Configuration
protected static class BasicConfiguration {
    @Bean
    public String created() {
        return "This is always created";
    }
}
@Configuration
@ConditionalOnBean(name = "created")
protected static class ConditionalOnBeanConfiguration {
    @Bean
    public String createOnBean() {
        return "This is created when bean (name=created) is present";
    }
}
@Configuration
@ConditionalOnMissingBean(name = "created")
protected static class ConditionalOnMissingBeanConfiguration {
    @Bean
    public String createOnMissingBean() {
        return "This is created when bean (name=created) is missing";
    }
} 
```

然后，我们像前一节一样调用`withUserConfiguration` 方法，并发送我们的自定义配置类来测试自动配置是否适当地实例化或跳过不同条件下的`createOnBean`或`createOnMissingBean`bean`:`

```java
@Test
public void whenDependentBeanIsPresent_thenConditionalBeanCreated() {
    this.contextRunner.withUserConfiguration(
        BasicConfiguration.class, 
        ConditionalOnBeanConfiguration.class
    )
    // ommitted for brevity
}
@Test
public void whenDependentBeanIsNotPresent_thenConditionalMissingBeanCreated() {
    this.contextRunner.withUserConfiguration(ConditionalOnMissingBeanConfiguration.class)
    // ommitted for brevity
}
```

### 2.3.测试属性条件

在本节中，让我们用**测试使用`@ConditionalOnProperty`注释**的自动配置类。

首先，我们需要这个测试的一个属性:

```java
com.baeldung.service=custom
```

之后，我们编写嵌套的自动配置类来基于前面的属性创建 beans:

```java
@Configuration
@TestPropertySource("classpath:ConditionalOnPropertyTest.properties")
protected static class SimpleServiceConfiguration {
    @Bean
    @ConditionalOnProperty(name = "com.baeldung.service", havingValue = "default")
    @ConditionalOnMissingBean
    public DefaultService defaultService() {
        return new DefaultService();
    }
```

```java
 @Bean
    @ConditionalOnProperty(name = "com.baeldung.service", havingValue = "custom")
    @ConditionalOnMissingBean
    public CustomService customService() {
        return new CustomService();
    }
} 
```

现在，我们调用`withPropertyValues`方法来覆盖每个测试中的属性值:

```java
@Test
public void whenGivenCustomPropertyValue_thenCustomServiceCreated() {
    this.contextRunner.withPropertyValues("com.baeldung.service=custom")
        .withUserConfiguration(SimpleServiceConfiguration.class)
        .run(context -> {
            assertThat(context).hasBean("customService");
            SimpleService simpleService = context.getBean(CustomService.class);
            assertThat(simpleService.serve()).isEqualTo("Custom Service");
            assertThat(context).doesNotHaveBean("defaultService");
        });
}

@Test
public void whenGivenDefaultPropertyValue_thenDefaultServiceCreated() {
    this.contextRunner.withPropertyValues("com.baeldung.service=default")
        .withUserConfiguration(SimpleServiceConfiguration.class)
        .run(context -> {
            assertThat(context).hasBean("defaultService");
            SimpleService simpleService = context.getBean(DefaultService.class);
            assertThat(simpleService.serve()).isEqualTo("Default Service");
            assertThat(context).doesNotHaveBean("customService");
        });
} 
```

## 3.结论

总而言之，本教程只是向**展示了如何使用`ApplicationContextRunner`来运行带有定制的`ApplicationContext`并应用断言**。

我们在这里涵盖了最常用的场景，而不是如何定制`ApplicationContext.`的详尽列表

同时，请记住`ApplicationConetxtRunner`是针对非 web 应用程序的，所以考虑将*WebApplicationContextRunner*用于基于 servlet 的 web 应用程序，将`ReactiveWebApplicationContextRunner`用于反应式 web 应用程序。

本教程的源代码可以在 GitHub 上找到[。](https://web.archive.org/web/20221129013933/https://github.com/eugenp/tutorials/tree/master/spring-boot-modules/spring-boot-autoconfiguration)