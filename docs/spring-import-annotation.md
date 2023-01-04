# Spring @Import 注释

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-import-annotation>

## 1.概观

在本教程中，我们将学习**如何使用 Spring `@Import`注释，同时阐明它与@** `**ComponentScan**.`有何不同

## 2.配置和 Beans

在理解`@Import`注释之前，我们需要知道什么是 Spring Bean，并对@ `Configuration`注释有一个基本的工作知识。

这两个主题都超出了本教程的范围。尽管如此，我们可以在我们的 Spring Bean 文章和 T2 Spring 文档中了解它们。

假设我们已经准备好了**三个 beans—`Bird`、`Cat`和`Dog`、**——每个 bean 都有自己的配置类。

然后，我们可以用这些`Config`类来**提供我们的上下文:**

```java
@ExtendWith(SpringExtension.class)
@ContextConfiguration(classes = { BirdConfig.class, CatConfig.class, DogConfig.class })
class ConfigUnitTest {

    @Autowired
    ApplicationContext context;

    @Test
    void givenImportedBeans_whenGettingEach_shallFindIt() {
        assertThatBeanExists("dog", Dog.class);
        assertThatBeanExists("cat", Cat.class);
        assertThatBeanExists("bird", Bird.class);
    }

    private void assertThatBeanExists(String beanName, Class<?> beanClass) {
        Assertions.assertTrue(context.containsBean(beanName));
        Assertions.assertNotNull(context.getBean(beanClass));
    }
}
```

## 3.使用`@Import`对配置进行分组

声明所有配置没有问题。但是**想象一下在不同的源**中控制几十个配置类的麻烦。应该有更好的办法。

@ `Import`注释有一个解决方案，它能够对`Configuration`类进行分组:

```java
@Configuration
@Import({ DogConfig.class, CatConfig.class })
class MammalConfiguration {
}
```

现在，我们只需要记住`mammals`:

```java
@ExtendWith(SpringExtension.class)
@ContextConfiguration(classes = { MammalConfiguration.class })
class ConfigUnitTest {

    @Autowired
    ApplicationContext context;

    @Test
    void givenImportedBeans_whenGettingEach_shallFindOnlyTheImportedBeans() {
        assertThatBeanExists("dog", Dog.class);
        assertThatBeanExists("cat", Cat.class);

        Assertions.assertFalse(context.containsBean("bird"));
    }

    private void assertThatBeanExists(String beanName, Class<?> beanClass) {
        Assertions.assertTrue(context.containsBean(beanName));
        Assertions.assertNotNull(context.getBean(beanClass));
    }
}
```

嗯，可能我们很快就会忘记我们的`Bird`，所以让我们再做**一组来包括所有的`animal`配置类**:

```java
@Configuration
@Import({ MammalConfiguration.class, BirdConfig.class })
class AnimalConfiguration {
}
```

最后，没有人掉队，我们只需要记住一节课:

```java
@ExtendWith(SpringExtension.class)
@ContextConfiguration(classes = { AnimalConfiguration.class })
class AnimalConfigUnitTest {
    // same test validating that all beans are available in the context
}
```

## 4.`@Import`对`@ComponentScan`

在继续@ `Import`的例子之前，让我们快速停下来，将其与 [@ `ComponentScan`](/web/20220926182613/https://www.baeldung.com/spring-component-scanning) 进行比较。

### 4.1.类似

这两种注释都可以**接受任何`@Component`或`@Configuration` 类。**

让我们使用@ `Import`添加一个新的@ `Component`:

```java
@Configuration
@Import(Bug.class)
class BugConfig {
}

@Component(value = "bug")
class Bug {
}
```

现在，`Bug` bean 就像任何其他 bean 一样可用。

### 4.2.概念差异

简单地说，我们**可以用两个注释**得到相同的结果。那么，它们之间有什么区别吗？

要回答这个问题，让我们记住 Spring 通常提倡[约定胜于配置](https://web.archive.org/web/20220926182613/https://en.wikipedia.org/wiki/Convention_over_configuration)的方法。

用我们的注解打个比方，、@ `ComponentScan`更像是约定俗成，而@ `Import`则像是配置 。

### 4.3.在实际应用中会发生什么

通常，我们在根包中使用@ `ComponentScan`来启动我们的应用程序**,这样它就可以为我们找到所有的组件。如果我们使用 Spring Boot，那么 **`[@SpringBootApplication](https://web.archive.org/web/20220926182613/https://docs.spring.io/spring-boot/docs/2.1.13.RELEASE/reference/html/using-boot-using-springbootapplication-annotation.html)`已经包含了`@ComponentScan`** ，我们就可以开始了。这显示了惯例的力量。**

现在，让我们假设我们的应用程序增长了很多。现在我们需要处理来自所有不同地方的 beans，比如组件、不同的包结构以及我们自己和第三方构建的模块。

在这种情况下，将所有内容添加到上下文中可能会引发关于使用哪个 bean 的冲突。除此之外，我们可能会得到一个缓慢的启动时间。

另一方面，**我们不想为每个新的组件**写一个@ `Import`,因为这样做会适得其反。

以我们的动物为例。我们确实可以在上下文声明中隐藏导入，但是我们仍然需要记住每个`Config`类的@ `Import`。

### 4.4.一起工作

我们可以力争两全其美。让我们想象一下，我们有一个仅用于我们的`animals`的包。它也可以是一个组件或模块，并保持相同的想法。

然后我们可以有一个 **@ `ComponentScan`只是为了我们的`animal`套餐**:

```java
package com.baeldung.importannotation.animal;

// imports...

@Configuration
@ComponentScan
public class AnimalScanConfiguration {
}
```

和一个 **`@Import`到** **保持对我们将添加**到上下文的控制:

```java
package com.baeldung.importannotation.zoo;

// imports...

@Configuration
@Import(AnimalScanConfiguration.class)
class ZooApplication {
}
```

最后，任何添加到 animal 包中的新 bean 都会被我们的上下文自动找到。我们仍然可以明确控制我们正在使用的配置。

## 5.结论

在这个快速教程中，我们学习了如何使用`@Import`来组织我们的配置。

我们还了解到 **`@Import`与@`ComponentScan`**非常相似，除了 **@ `Import`有一个显式的方法，而@ `ComponentScan`使用了一个隐式的** 。

此外，我们还研究了在实际应用程序中控制我们的配置可能遇到的困难，以及如何通过组合两种注释来处理这些困难。

像往常一样，完整的代码可以在 GitHub 的[上找到。](https://web.archive.org/web/20220926182613/https://github.com/eugenp/tutorials/tree/master/spring-core-4)