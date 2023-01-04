# Spring @Qualifier 注释

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-qualifier-annotation>

## 1.概观

在本教程中，我们将探索 **`@Qualifier`注释对**有什么帮助，它解决了哪些问题，以及如何使用它。

## 延伸阅读:

## [Spring @Primary 注释](/web/20221021155202/https://www.baeldung.com/spring-primary)

Learn how to use Spring's @Primary annotation to give preference to beans when autowiring[Read more](/web/20221021155202/https://www.baeldung.com/spring-primary) →

## [春季布线:@Autowired、@Resource 和@Inject](/web/20221021155202/https://www.baeldung.com/spring-annotations-resource-inject-autowire)

This article will compare and contrast the use of annotations related to dependency injection, namely the @Resource, @Inject, and @Autowired annotations.[Read more](/web/20221021155202/https://www.baeldung.com/spring-annotations-resource-inject-autowire) →

## [@春天查找注释](/web/20221021155202/https://www.baeldung.com/spring-lookup)

Learn how to effectively use the @Lookup annotation in Spring for procedural injection.[Read more](/web/20221021155202/https://www.baeldung.com/spring-lookup) →

我们还将解释它与`@Primary`注释的不同，以及与按名称自动连接的不同。

## 2.自动布线需要消除歧义

[`@Autowired`](/web/20221021155202/https://www.baeldung.com/spring-autowire) 注释是一种很好的方式，可以让在 Spring 中注入依赖项的需求显式化。虽然它很有用，但是在一些用例中，仅仅这个注释不足以让 Spring 理解注入哪个 bean。

默认情况下，Spring 根据类型解析自动连接的条目。

**如果容器中有多个相同类型的 bean 可用，框架将抛出`NoUniqueBeanDefinitionException`** `,`，表明有多个 bean 可用于自动绑定。

让我们想象这样一种情况，在给定的实例中，Spring 有两个可能的候选者作为 bean 合作者注入:

```
@Component("fooFormatter")
public class FooFormatter implements Formatter {

    public String format() {
        return "foo";
    }
}

@Component("barFormatter")
public class BarFormatter implements Formatter {

    public String format() {
        return "bar";
    }
}

@Component
public class FooService {

    @Autowired
    private Formatter formatter;
}
```

如果我们试图将`FooService`加载到我们的上下文中，Spring 框架将抛出一个`NoUniqueBeanDefinitionException`。这是因为**春天不知道给哪颗豆子注射了**。为了避免这个问题，有几种解决方案；`@Qualifier`注解就是其中之一。

## 3.`@Qualifier`注释

通过使用`@Qualifier` 注释，我们可以**消除哪个 bean 需要被注入**的问题。

让我们回顾一下前面的例子，看看我们如何通过包含`@Qualifier`注释来指示我们想要使用哪个 bean 来解决这个问题:

```
public class FooService {

    @Autowired
    @Qualifier("fooFormatter")
    private Formatter formatter;
}
```

通过包含`@Qualifier` 注释，以及我们想要使用的特定实现的名称，在本例`Foo,` 中，当 Spring 找到多个相同类型的 beans 时，我们可以避免歧义。

我们需要考虑要使用的限定符名称是在`@Component`注释中声明的名称。

注意，我们也可以在`Formatter`实现类上使用`@Qualifier`注释，而不是在它们的`@Component`注释中指定名称，以获得相同的效果:

```
@Component
@Qualifier("fooFormatter")
public class FooFormatter implements Formatter {
    //...
}

@Component
@Qualifier("barFormatter")
public class BarFormatter implements Formatter {
    //...
} 
```

## 4.`@Qualifier`对`@Primary`

还有另一个名为 [`@Primary`](/web/20221021155202/https://www.baeldung.com/spring-primary) 的注释，当依赖注入出现歧义时，我们可以使用它来决定注入哪个 bean。

这个注释**定义了当多个相同类型的 beans 出现时的首选项**。除非另有说明，否则将使用与`@Primary`注释相关联的 bean。

让我们看一个例子:

```
@Configuration
public class Config {

    @Bean
    public Employee johnEmployee() {
        return new Employee("John");
    }

    @Bean
    @Primary
    public Employee tonyEmployee() {
        return new Employee("Tony");
    }
}
```

在这个例子中，两个方法返回相同的`Employee`类型。Spring 将注入的 bean 是方法`tonyEmployee`返回的 bean。这是因为它包含了`@Primary` 注解。当我们想要**指定默认情况下应该注入哪一个特定类型的 bean**时，这个注释非常有用。

如果我们在某个注入点需要另一个 bean，我们将需要特别指出它。我们可以通过`@Qualifier`注释做到这一点。例如，我们可以通过使用`@Qualifier` 注释来指定我们想要使用由`johnEmployee`方法返回的 bean。

值得注意的是**如果`@Qualifier` 和`@Primary` 注释都存在，那么`@Qualifier` 注释将优先。**基本上，`@Primary`定义了一个默认，而`@Qualifier` 非常具体。

让我们看看使用`@Primary`注释的另一种方式，这次使用最初的例子:

```
@Component
@Primary
public class FooFormatter implements Formatter {
    //...
}

@Component
public class BarFormatter implements Formatter {
    //...
} 
```

**在这种情况下，`@Primary`注释被放在一个实现类**中，将消除场景的歧义。

## 5.`@Qualifier` vs 按名称自动布线

另一种在自动连接时决定多个 beans 的方法是使用字段的名称来注入。**这是默认设置，以防 Spring** 没有其他提示。让我们看看基于我们最初的例子的一些代码:

```
public class FooService {

    @Autowired
    private Formatter fooFormatter;
}
```

在这种情况下，Spring 将确定要注入的 bean 是`FooFormatter`bean，因为字段名与我们在该 bean 的`@Component`注释中使用的值相匹配。

## 6.结论

在本文中，我们描述了需要明确注入哪些 beans 的场景。特别是，我们检查了`@Qualifier` 注释，并将其与其他类似的确定需要使用哪些 beans 的方法进行了比较。

和往常一样，这篇文章的完整代码可以在 GitHub 的[上找到。](https://web.archive.org/web/20221021155202/https://github.com/eugenp/tutorials/tree/master/spring-di)