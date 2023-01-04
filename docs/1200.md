# 春季数据中的 Exists 查询

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-data-exists-query>

## 1.介绍

在许多以数据为中心的应用程序中，可能会出现需要检查特定对象是否已经存在的情况。

在本教程中，我们将讨论使用 Spring 数据和 JPA 实现这一点的几种方法。

## 2.样本实体

为了给我们的例子做准备，我们将创建一个实体`Car`，它有两个属性`model `和`power`:

```
@Entity
public class Car {

    @Id
    @GeneratedValue
    private int id;

    private Integer power;
    private String model;

    // getters, setters, ...
}
```

## 3.按 ID 搜索

`JpaRepository` 接口公开了`existsById`方法，该方法检查数据库中是否存在具有给定`id`的实体:

```
int searchId = 2; // ID of the Car
boolean exists = repository.existsById(searchId)
```

让我们假设`searchId` 是我们在测试设置期间创建的`Car`的`id`。为了测试的可重复性，我们不应该使用硬编码的数字(例如“2”)，因为`Car`的`id`属性很可能是自动生成的，并且会随着时间而改变。**`existsById`查询是检查对象是否存在的最简单但最不灵活的方式**。

## 4.使用派生查询方法

我们还可以使用 Spring 的派生查询方法特性来制定我们的查询。在我们的例子中，我们想要检查具有给定模型名称的`Car`是否存在；因此，我们设计了以下查询方法:

```
boolean existsCarByModel(String model);
```

需要注意的是，方法的命名不是任意的，它必须遵循一定的规则。然后，Spring 将为存储库生成代理，这样它就可以从方法的名称中获得 SQL 查询。现代的 ide，如 IntelliJ IDEA，将提供语法补全。

当查询变得更复杂时，比如合并排序、限制结果，以及包括几个查询条件时，这些方法名会变得很长，甚至到了难以理解的程度。派生查询方法也可能看起来有点神奇，因为它们隐含的和“约定”的性质。

然而，当干净整洁的代码很重要时，以及当开发人员想要依赖一个经过良好测试的框架时，它们会派上用场。

## 5.通过`Example`搜索

`Example` 是一种非常强大的检查存在的方式，因为它使用`Example` *匹配器*来动态构建查询。所以每当我们需要动态性时，这是一个很好的方法。在我们的 [Spring 数据查询](/web/20220926185529/https://www.baeldung.com/spring-data-query-by-example)文章中可以找到关于 Spring 的`ExampleMatcher`以及如何使用它们的全面解释。

### 5.1.火柴人

假设我们想以不区分大小写的方式搜索模型名称。我们将从创建我们的`ExampleMatcher`开始:

```
ExampleMatcher modelMatcher = ExampleMatcher.matching()
  .withIgnorePaths("id") 
  .withMatcher("model", ignoreCase());
```

注意，我们必须显式忽略`id`路径，因为`id `是主键，默认情况下会自动选择这些路径。

### 5.2.探测器

接下来，我们需要定义一个所谓的“探针”，它是我们想要查找的类的一个实例。它设置了所有与搜索相关的属性。然后，我们将它连接到我们的`nameMatcher,`并执行查询:

```
Car probe = new Car();
probe.setModel("bmw");
Example<Car> example = Example.of(probe, modelMatcher);
boolean exists = repository.exists(example);
```

伴随着巨大的灵活性而来的是巨大的复杂性，尽管`ExampleMatcher` API 可能非常强大，但是使用它会产生相当多的额外代码。我们建议**在动态查询中使用它，或者如果没有其他方法适合我们的需求**。

## 6.使用 Exists 语义编写自定义 JPQL 查询

我们将研究的最后一种方法使用 JPQL (Java 持久性查询语言)来实现带有 exists `–`语义的定制查询:

```
@Query("select case when count(c)> 0 then true else false end from Car c where lower(c.model) like lower(:model)")
boolean existsCarLikeCustomQuery(@Param("model") String model);
```

想法是**基于`model`属性执行一个不区分大小写的`count`查询，评估返回值，并将结果映射到一个 Java** `**boolean**.`再次强调，大多数 ide 对 JPQL 语句有很好的支持。

[定制 JPQL 查询](/web/20220926185529/https://www.baeldung.com/spring-data-jpa-query)可以被视为派生方法的替代方案，当我们习惯使用类似 SQL 的语句并且不介意额外的`@Query`注释时，它通常是一个不错的选择。

## 7.结论

在本文中，我们学习了如何使用 Spring 数据和 JPA 检查数据库中是否存在某个对象。何时使用每种方法没有硬性规定，因为这很大程度上取决于手头的用例以及个人偏好。

不过，作为一个好的经验法则，如果有选择，出于健壮性、性能和代码清晰性的原因，开发人员应该总是倾向于更直接的方法。此外，一旦决定了是使用派生查询还是自定义 JPQL 查询，最好尽可能长时间地坚持这一选择，以确保一致的编码风格。

完整的源代码示例可以在 [GitHub](https://web.archive.org/web/20220926185529/https://github.com/eugenp/tutorials/tree/master/persistence-modules/spring-data-jpa-query) 上找到。