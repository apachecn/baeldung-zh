# JPA 查询的类型

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/jpa-queries>

## 1。概述

在本教程中，我们将讨论不同类型的 [JPA](/web/20221205231836/https://www.baeldung.com/jpa-hibernate-difference) 查询。此外，我们将重点比较它们之间的差异，并阐述各自的优缺点。

## 2.设置

首先，让我们定义将在本文所有示例中使用的`UserEntity`类:

```java
@Table(name = "users")
@Entity
public class UserEntity {

    @Id
    private Long id;
    private String name;
    //Standard constructor, getters and setters.

}
```

**JPA 查询有三种基本类型:**

*   `Query`，用 Java 持久性查询语言(JPQL)语法编写
*   `NativeQuery`，用纯 SQL 语法编写
*   `Criteria API Query`，通过不同的方法以编程方式构建

让我们来探索它们。

## 3.`Query`

**A `Query`在语法上类似于 SQL，一般用于执行 CRUD 操作:**

```java
public UserEntity getUserByIdWithPlainQuery(Long id) {
    Query jpqlQuery = getEntityManager().createQuery("SELECT u FROM UserEntity u WHERE u.id=:id");
    jpqlQuery.setParameter("id", id);
    return (UserEntity) jpqlQuery.getSingleResult();
}
```

这个`Query`从`users`表中检索匹配的记录，并将其映射到`UserEntity`对象。

还有两个附加的`Query`子类型:

*   `TypedQuery`
*   `NamedQuery`

让我们看看他们的行动。

### 3.1。`TypedQuery`

我们需要注意前面例子中的`return`语句。JPA 无法推断出`Query`的结果类型是什么，因此，我们必须进行强制转换。

但是， **JPA 提供了一个特殊的`Query`子类型，称为`TypedQuery.`** ，如果我们事先知道我们的`Query`结果类型，这总是首选。此外，它使我们的代码更可靠，更容易测试。

让我们来看一个`TypedQuery`替代方案，与我们的第一个例子相比:

```java
public UserEntity getUserByIdWithTypedQuery(Long id) {
    TypedQuery<UserEntity> typedQuery
      = getEntityManager().createQuery("SELECT u FROM UserEntity u WHERE u.id=:id", UserEntity.class);
    typedQuery.setParameter("id", id);
    return typedQuery.getSingleResult();
}
```

这样，**我们可以免费得到更强的类型，**避免以后可能的类型转换异常。

### 3.2。`NamedQuery`

虽然我们可以在特定的方法上动态定义一个`Query`,但是它们最终会发展成一个难以维护的代码库。如果我们能把一般用法的查询放在一个集中的、易于阅读的地方会怎么样？

JPA 也用另一个叫做`[NamedQuery](/web/20221205231836/https://www.baeldung.com/hibernate-named-query)`的`Query`子类型来覆盖我们。

我们可以在`orm.xml`或属性文件中定义`NamedQueries`。

**此外，我们可以在`Entity`类本身上定义`NamedQuery`，提供一种集中、快速和简单的方式来读取和查找`Entity`的相关查询。**

所有的`NamedQueries`必须有一个唯一的名称。

让我们看看如何将一个`NamedQuery`添加到我们的`UserEntity`类中:

```java
@Table(name = "users")
@Entity
@NamedQuery(name = "UserEntity.findByUserId", query = "SELECT u FROM UserEntity u WHERE u.id=:userId")
public class UserEntity {

    @Id
    private Long id;
    private String name;
    //Standard constructor, getters and setters.

}
```

**如果我们使用的是版本 8 之前的 Java，那么`@NamedQuery`注释必须组合在`@NamedQueries`注释中。从 Java 8 开始，我们可以简单地在我们的`Entity`类中重复`@NamedQuery`注释。**

使用`NamedQuery`非常简单:

```java
public UserEntity getUserByIdWithNamedQuery(Long id) {
    Query namedQuery = getEntityManager().createNamedQuery("UserEntity.findByUserId");
    namedQuery.setParameter("userId", id);
    return (UserEntity) namedQuery.getSingleResult();
}
```

## 4.`NativeQuery`

**A `NativeQuery`就是一个简单的 SQL 查询。这些允许我们释放数据库的全部能力，因为我们可以使用 JPQL 限制语法中没有的专有特性。**

这是有代价的。我们用`NativeQuery`失去了应用程序的数据库可移植性，因为我们的 JPA 提供者不能再从数据库实现或供应商那里提取具体的细节。

让我们看看如何使用一个`NativeQuery`来产生与我们之前的例子相同的结果:

```java
public UserEntity getUserByIdWithNativeQuery(Long id) {
    Query nativeQuery
      = getEntityManager().createNativeQuery("SELECT * FROM users WHERE id=:userId", UserEntity.class);
    nativeQuery.setParameter("userId", id);
    return (UserEntity) nativeQuery.getSingleResult();
}
```

我们必须始终考虑 a `NativeQuery`是否是唯一的选择。**大多数时候，一个好的 JPQL `Query`可以满足我们的需求，最重要的是，维护实际数据库实现的抽象层次。**

使用`NativeQuery`并不一定意味着将我们局限于某个特定的数据库供应商。毕竟，如果我们的查询不使用专有的 SQL 命令，而只使用标准的 SQL 语法，切换提供者应该不是问题。

## 5.`Query, NamedQuery`和`NativeQuery`

到目前为止，我们已经学习了`Query`、`NamedQuery`和`NativeQuery`。

现在，让我们快速重温一下，总结一下它们的优缺点。

### 5.1.`Query`

我们可以使用`entityManager.createQuery(queryString)`创建一个查询。

接下来，我们来探讨一下`Query`的利弊:

优点:

*   当我们使用`EntityManager`创建查询时，我们可以构建动态查询字符串
*   查询是用 JPQL 编写的，所以它们是可移植的

缺点:

*   对于动态查询，根据[查询计划缓存](/web/20221205231836/https://www.baeldung.com/hibernate-query-plan-cache)，它可能会被多次编译成一条原生 SQL 语句
*   这些查询可能分散到不同的 Java 类中，并与 Java 代码混合在一起。因此，如果一个项目包含许多查询，维护起来可能会很困难

### 5.2.`NamedQuery`

一旦定义了一个`NamedQuery`，我们可以使用`EntityManager`来引用它:

```java
entityManager.createNamedQuery(queryName);
```

现在，我们来看看`NamedQueries`的优缺点:

优点:

*   `NamedQueries`在持久性单元加载时被编译和验证。也就是说，它们只编译一次
*   我们可以将`NamedQueries`集中化以使它们更容易维护——例如，在`orm.xml`中，在属性文件中，或者在`@Entity`类中

缺点:

*   `NamedQueries`总是静态的
*   `NamedQueries`可以在 Spring Data JPA 存储库中引用。但是不支持[动态排序](/web/20221205231836/https://www.baeldung.com/spring-data-sorting#sorting-with-spring-data)

### 5.3.`NativeQuery`

我们可以使用`EntityManager`创建一个`NativeQuery`:

```java
entityManager.createNativeQuery(sqlStmt);
```

根据结果映射，我们还可以将第二个参数传递给方法，比如一个`Entity`类，正如我们在前面的例子中看到的。

`NativeQueries`也有利弊。让我们快速看一下:

优点:

*   随着我们的查询变得复杂，有时 JPA 生成的 SQL 语句并不是最佳的。在这种情况下，我们可以使用`NativeQueries`来提高查询效率
*   `NativeQueries`允许我们使用特定于数据库供应商的特性。有时候，这些特性可以提高我们的查询性能

缺点:

*   特定于供应商的特性可以带来便利和更好的性能，但是我们为此付出的代价是失去了从一个数据库到另一个数据库的可移植性

## 6. **`Criteria` API 查询**

**[`Criteria` API 查询](/web/20221205231836/https://www.baeldung.com/hibernate-criteria-queries)是以编程方式构建的类型安全查询——在语法上有点类似于 JPQL 查询:**

```java
public UserEntity getUserByIdWithCriteriaQuery(Long id) {
    CriteriaBuilder criteriaBuilder = getEntityManager().getCriteriaBuilder();
    CriteriaQuery<UserEntity> criteriaQuery = criteriaBuilder.createQuery(UserEntity.class);
    Root<UserEntity> userRoot = criteriaQuery.from(UserEntity.class);
    UserEntity queryResult = getEntityManager().createQuery(criteriaQuery.select(userRoot)
      .where(criteriaBuilder.equal(userRoot.get("id"), id)))
      .getSingleResult();
    return queryResult;
}
```

直接使用`Criteria` API 查询可能令人望而生畏，但是当我们需要添加动态查询元素或者与 [JPA `Metamodel`结合使用时，它们可能是一个很好的选择。](/web/20221205231836/https://www.baeldung.com/hibernate-criteria-queries-metamodel)

## 7。结论

在这篇简短的文章中，我们学习了什么是 JPA 查询，以及它们的用法。

JPA 查询是从我们的数据访问层抽象出我们的业务逻辑的好方法，因为我们可以依赖 JPQL 语法，并让我们选择的 JPA 提供者处理`Query`翻译。

本文中的所有代码都可以在 GitHub 上的[处获得。](https://web.archive.org/web/20221205231836/https://github.com/eugenp/tutorials/tree/master/persistence-modules/java-jpa-2)