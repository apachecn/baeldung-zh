# Spring 中的@Component vs @Repository 和@Service

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-component-repository-service>

## 1。简介

在这个快速教程中，我们将了解 Spring 框架中的*@组件、@存储库、*和*@服务*注释之间的区别。

## 延伸阅读:

## [Spring @ Autowired 指南](/web/20220827110142/https://www.baeldung.com/spring-autowire)

A guide to the most commonest usage of Springs @Autowired annotation and qualifiers[Read more](/web/20220827110142/https://www.baeldung.com/spring-autowire) →

## [Spring @ Qualifier 注释](/web/20220827110142/https://www.baeldung.com/spring-qualifier-annotation)

@Autowired alone isn't sometimes enough to disambiguate dependencies. You can wire more exactly using the @Qualifier annotation. @Primary can also help.[Read more](/web/20220827110142/https://www.baeldung.com/spring-qualifier-annotation) →

## 2.弹簧注释

在大多数典型的应用程序中，我们有不同的层，如数据访问、表示层、服务层、业务层等。

此外，在每一层中，我们有不同的 beans。为了自动检测这些 beans， **Spring 使用类路径扫描注释**。

然后它在`ApplicationContext`中注册每个 bean。

以下是其中一些注释的简要概述:

*   `@Component`是任何 Spring 管理的组件的通用原型。
*   在服务层注释类。
*   在持久层注释类，持久层将充当数据库仓库。

我们已经有了一篇关于这些注释的[扩展文章](/web/20220827110142/https://www.baeldung.com/spring-bean-annotations)，所以我们在这里将重点放在它们之间的差异上。

## 3。有什么不同？

这些定型之间的主要区别在于它们用于不同的分类。当我们注释一个自动检测的类时，我们应该使用各自的原型。

现在让我们更详细地看一下。

### 3.1。`@Component`

**我们可以在应用程序中使用@Component 将 beans 标记为 Spring 的托管组件**。Spring 只会用`@Component,` 拾取和注册豆子，一般不会去找`@Service`和`@Repository`。

它们被注册在`ApplicationContext`中，因为它们被标注了`@Component`:

```java
@Component
public @interface Service {
} 
```

```java
@Component
public @interface Repository {
} 
```

`@Service`和`@Repository`是`@Component`的特例。它们在技术上是相同的，但我们使用它们的目的不同。

### 3.2。`@Repository`

**`@Repository`的工作是捕捉特定于持久性的异常，并将它们作为 Spring 的统一未检查异常**之一重新抛出。

为此，Spring 提供了`PersistenceExceptionTranslationPostProcessor`，我们需要将它添加到我们的应用程序上下文中(如果我们使用 Spring Boot，就已经包含了):

```java
<bean class=
  "org.springframework.dao.annotation.PersistenceExceptionTranslationPostProcessor"/>
```

这个 bean post 处理器向任何用`@Repository.`注释的 bean 添加一个 advisor

### 3.3。`@Service`

**我们用@Service 标记 beans，以表明它们拥有业务逻辑**。除了在服务层使用之外，该注释没有任何其他特殊用途。

## 4。结论

**在本文中，我们了解了*@组件、@存储库、*和*@服务*注释**之间的区别。我们分别检查了每个注释，以了解它们的使用范围。

总之，根据图层约定选择注记总是一个好主意。