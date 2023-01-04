# JPA 标准查询

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/hibernate-criteria-queries>

## 1。概述

在本教程中，我们将讨论一个非常有用的 JPA 特性——条件查询。

它使我们能够在不使用原始 SQL 的情况下编写查询，并为我们提供了对查询的一些面向对象的控制，这是 Hibernate 的主要特性之一。Criteria API 允许我们以编程方式构建一个 criteria query 对象，在这里我们可以应用不同种类的过滤规则和逻辑条件。

从 Hibernate 5.2 开始，Hibernate Criteria API 被弃用，新的开发集中在 JPA Criteria API 上。我们将探索如何使用 Hibernate 和 JPA 来构建条件查询。

## 延伸阅读:

## [春季数据 JPA @Query](/web/20220926194733/https://www.baeldung.com/spring-data-jpa-query)

Learn how to use the @Query annotation in Spring Data JPA to define custom queries using JPQL and native SQL.[Read more](/web/20220926194733/https://www.baeldung.com/spring-data-jpa-query) →

## [Spring Data JPA 简介](/web/20220926194733/https://www.baeldung.com/the-persistence-layer-with-spring-data-jpa)

Introduction to Spring Data JPA with Spring 4 - the Spring config, the DAO, manual and generated queries and transaction management.[Read more](/web/20220926194733/https://www.baeldung.com/the-persistence-layer-with-spring-data-jpa) →

## [使用 JPA 进行查询的指南](/web/20220926194733/https://www.baeldung.com/querydsl-with-jpa-tutorial)

A quick guide to using Querydsl with the Java Persistence API.[Read more](/web/20220926194733/https://www.baeldung.com/querydsl-with-jpa-tutorial) →

## 2。Maven 依赖关系

为了说明 API，我们将使用参考 JPA 实现 Hibernate。

要使用 Hibernate，我们要确保将它的最新版本添加到我们的`pom.xml` 文件中:

```java
<dependency>
    <groupId>org.hibernate</groupId>
    <artifactId>hibernate-core</artifactId>   
    <version>5.3.2.Final</version>
</dependency>
```

我们可以在这里找到 Hibernate [的最新版本。](https://web.archive.org/web/20220926194733/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.hibernate%22%20AND%20a%3A%22hibernate-core%22)

## 3。使用标准的简单示例

让我们先看看如何使用条件查询来检索数据。我们将看看如何从数据库中获取特定类的所有实例。

我们有一个代表数据库中元组`“ITEM”`的`Item`类:

```java
public class Item implements Serializable {

    private Integer itemId;
    private String itemName;
    private String itemDescription;
    private Integer itemPrice;

   // standard setters and getters
}
```

让我们来看一个简单的标准查询，它将从数据库中检索所有的`“ITEM”`行:

```java
Session session = HibernateUtil.getHibernateSession();
CriteriaBuilder cb = session.getCriteriaBuilder();
CriteriaQuery<Item> cr = cb.createQuery(Item.class);
Root<Item> root = cr.from(Item.class);
cr.select(root);

Query<Item> query = session.createQuery(cr);
List<Item> results = query.getResultList();
```

上面的查询是如何获取所有项目的简单演示。让我们一步步来看:

1.  从`SessionFactory`对象创建一个`Session`的实例
2.  通过调用`getCriteriaBuilder()`方法创建 C `riteriaBuilder`的实例
3.  通过调用`CriteriaBuilder` `createQuery()`方法创建一个`CriteriaQuery`的实例
4.  通过调用`Session` `createQuery()`方法创建一个`Query`的实例
5.  调用`query`对象的`getResultList()`方法，它给我们结果

既然我们已经讨论了基础知识，让我们继续讨论条件查询的一些特性。

### 3.1。使用`Expressions`

**`CriteriaBuilder`通过使用`CriteriaQuery where()`方法，提供`CriteriaBuilder`创建的`Expressions`，可以根据特定条件**限制查询结果。

我们来看一些常用的`Expressions`的例子。

为了获得价格超过 1000 的项目:

```java
cr.select(root).where(cb.gt(root.get("itemPrice"), 1000));
```

接下来，获取`itemPrice`小于 1000 的项目:

```java
cr.select(root).where(cb.lt(root.get("itemPrice"), 1000));
```

有`itemName`的项目包含`Chair`:

```java
cr.select(root).where(cb.like(root.get("itemName"), "%chair%"));
```

记录的`itemPrice`在 100 和 200 之间:

```java
cr.select(root).where(cb.between(root.get("itemPrice"), 100, 200));
```

`Skate Board`、 `Paint`、`Glue`中有`itemName`的项目:

```java
cr.select(root).where(root.get("itemName").in("Skate Board", "Paint", "Glue"));
```

检查给定属性是否为空:

```java
cr.select(root).where(cb.isNull(root.get("itemDescription")));
```

检查给定属性是否不为空:

```java
cr.select(root).where(cb.isNotNull(root.get("itemDescription")));
```

我们还可以使用方法`isEmpty()`和`isNotEmpty()` 来测试一个类中的`List`是否为空。

此外，我们可以结合两个或更多的上述比较。 **T** **he Criteria API 允许我们轻松地链接表达式**:

```java
Predicate[] predicates = new Predicate[2];
predicates[0] = cb.isNull(root.get("itemDescription"));
predicates[1] = cb.like(root.get("itemName"), "chair%");
cr.select(root).where(predicates);
```

使用逻辑运算添加两个表达式:

```java
Predicate greaterThanPrice = cb.gt(root.get("itemPrice"), 1000);
Predicate chairItems = cb.like(root.get("itemName"), "Chair%");
```

用`Logical OR`连接的具有上述定义条件的项目:

```java
cr.select(root).where(cb.or(greaterThanPrice, chairItems));
```

用`Logical AND`连接得到符合上述定义条件的项目:

```java
cr.select(root).where(cb.and(greaterThanPrice, chairItems));
```

### 3.2。分类

现在我们知道了`Criteria`的基本用法，让我们看看`Criteria`的排序功能。

在下面的示例中，我们先按名称的升序，然后按价格的降序对列表进行排序:

```java
cr.orderBy(
  cb.asc(root.get("itemName")), 
  cb.desc(root.get("itemPrice")));
```

在下一节，我们将看看如何做聚合函数。

### 3.3。投影、聚集和分组功能

现在让我们看看不同的聚合函数。

获取行数:

```java
CriteriaQuery<Long> cr = cb.createQuery(Long.class);
Root<Item> root = cr.from(Item.class);
cr.select(cb.count(root));
Query<Long> query = session.createQuery(cr);
List<Long> itemProjected = query.getResultList();
```

下面是一个聚合函数的例子——`Average`的`Aggregate`函数:

```java
CriteriaQuery<Double> cr = cb.createQuery(Double.class);
Root<Item> root = cr.from(Item.class);
cr.select(cb.avg(root.get("itemPrice")));
Query<Double> query = session.createQuery(cr);
List avgItemPriceList = query.getResultList();
```

其他有用的聚集方法有`sum()`、`max()`、`min()`、、`count()`等。

### 3.4。`CriteriaUpdate`

**从 JPA 2.1 开始，支持使用`Criteria` API 执行数据库更新。**

`CriteriaUpdate`有一个`set()`方法，可用于为数据库记录提供新值:

```java
CriteriaUpdate<Item> criteriaUpdate = cb.createCriteriaUpdate(Item.class);
Root<Item> root = criteriaUpdate.from(Item.class);
criteriaUpdate.set("itemPrice", newPrice);
criteriaUpdate.where(cb.equal(root.get("itemPrice"), oldPrice));

Transaction transaction = session.beginTransaction();
session.createQuery(criteriaUpdate).executeUpdate();
transaction.commit();
```

在上面的代码片段中，我们从`CriteriaBuilder`创建了一个`CriteriaUpdate<Item>` 的实例，并使用它的`set()`方法为`itemPrice`提供新值。为了更新多个属性，我们只需要多次调用`set()`方法。

### 3.5。`CriteriaDelete`

`CriteriaDelete`使用`Criteria` API 启用删除操作。

我们只需要创建一个`CriteriaDelete`的实例，并使用`where()`方法来应用限制:

```java
CriteriaDelete<Item> criteriaDelete = cb.createCriteriaDelete(Item.class);
Root<Item> root = criteriaDelete.from(Item.class);
criteriaDelete.where(cb.greaterThan(root.get("itemPrice"), targetPrice));

Transaction transaction = session.beginTransaction();
session.createQuery(criteriaDelete).executeUpdate();
transaction.commit();
```

## 4。对 HQL 的优势

在前面的章节中，我们介绍了如何使用条件查询。

显然，与 HQL 相比，标准查询最主要也是最强有力的优势是漂亮、干净、面向对象的 API。

与简单的 HQL 相比，我们可以简单地编写更加灵活、动态的查询。该逻辑可以用 IDE 重构，并具有 Java 语言本身的所有类型安全优势。

当然，也有一些缺点，尤其是在更复杂的连接方面。

因此，我们通常必须使用最好的工具来完成工作——在大多数情况下，这可能是标准 API，但在某些情况下，我们必须使用更低的标准。

## 5。结论

在本文中，我们重点介绍了 Hibernate 和 JPA 中标准查询的基础知识，以及 API 的一些高级特性。

这里讨论的代码可以在 [GitHub 库](https://web.archive.org/web/20220926194733/https://github.com/eugenp/tutorials/tree/master/persistence-modules/hibernate-queries)中找到。