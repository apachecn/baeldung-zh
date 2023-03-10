# 使用 JPA 排序

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/jpa-sort>

## 1。概述

这篇文章展示了使用 JPA 排序的各种方法。

## 延伸阅读:

## [春季数据 JPA @Query](/web/20220815044534/https://www.baeldung.com/spring-data-jpa-query)

Learn how to use the @Query annotation in Spring Data JPA to define custom queries using JPQL and native SQL.[Read more](/web/20220815044534/https://www.baeldung.com/spring-data-jpa-query) →

## [JPA 属性转换器](/web/20220815044534/https://www.baeldung.com/jpa-attribute-converters)

Take a look at mapping JDBC types to Java classes in JPA using attribute converters.[Read more](/web/20220815044534/https://www.baeldung.com/jpa-attribute-converters) →

## 2。用 JPA / JQL API 排序

使用 JQL 进行排序是在`Order By`子句的帮助下完成的:

```java
String jql ="Select f from Foo as f order by f.id";
Query query = entityManager.createQuery (jql);
```

基于这个查询，JPA 生成下面的 straighforward **SQL 语句**:

```java
Hibernate: select foo0_.id as id1_4_, foo0_.name as name2_4_ 
    from Foo foo0_ order by foo0_.id
```

注意，JQL 字符串中的 SQL 关键字不区分大小写，但是实体的名称及其属性是区分大小写的。

### 2.1。设置分类顺序

默认情况下**排序顺序是升序**，但是可以在 JQL 字符串中显式设置。正如在纯 SQL 中一样，排序选项是 `asc`和`desc`:

```java
String jql = "Select f from Foo as f order by f.id desc";
Query sortQuery = entityManager.createQuery(jql);
```

**生成的 SQL 查询**将包含订单方向:

```java
Hibernate: select foo0_.id as id1_4_, foo0_.name as name2_4_ 
    from Foo foo0_ order by foo0_.id desc
```

### 2.2。按多个属性排序

为了按多个属性排序，这些属性被添加到 JQL 字符串的 o `rder by`子句中:

```java
String jql ="Select f from Foo as f order by f.name asc, f.id desc";
Query sortQuery = entityManager.createQuery(jql);
```

这两种排序条件都会出现在**生成的 SQL 查询**语句中:

```java
Hibernate: select foo0_.id as id1_4_, foo0_.name as name2_4_ 
    from Foo foo0_ order by foo0_.name asc, foo0_.id desc
```

### 2.3。设置空值的排序优先级

空值的默认优先级是特定于数据库的，但是这可以通过 HQL 查询字符串中的`NULLS FIRST`或`NULLS LAST`子句进行定制。

下面是一个简单的例子——按`Foo`的`name`降序排序，并将`Null`放在最后:

```java
Query sortQuery = entityManager.createQuery
    ("Select f from Foo as f order by f.name desc NULLS LAST");
```

生成的 SQL 查询包括`is null the 1 else 0 end clause`(第 3 行):

```java
Hibernate: select foo0_.id as id1_4_, foo0_.BAR_ID as BAR_ID2_4_, 
    foo0_.bar_Id as bar_Id2_4_, foo0_.name as name3_4_,from Foo foo0_ order 
    by case when foo0_.name is null then 1 else 0 end, foo0_.name desc
```

### 2.4。一对多关系排序

跳过基本的例子，现在让我们来看一个用例，它涉及到在一对多关系–`Bar`中的**排序实体，该关系包含一组`Foo`实体。**

我们希望对`Bar`实体以及它们的`Foo`实体集合进行排序——JPA 对于这项任务来说特别简单:

1.  对集合排序:在`Bar`实体:

    ```java
    @OrderBy("name ASC")
    List <Foo> fooList;
    ```

    中的`Foo`集合之前添加一个`OrderBy`注释
2.  对包含集合的实体进行排序:

    ```java
    String jql = "Select b from Bar as b order by b.id";
    Query barQuery = entityManager.createQuery(jql);
    List<Bar> barList = barQuery.getResultList();
    ```

注意，`@OrderBy`注释是可选的，但是我们在这里使用它，因为我们想要对每个`Bar`的`Foo`集合进行排序。

让我们看看发送到 RDMS 的 SQL 查询:

```java
Hibernate: select bar0_.id as id1_0_, bar0_.name as name2_0_ from Bar bar0_ order by bar0_.id

Hibernate: 
select foolist0_.BAR_ID as BAR_ID2_0_0_, foolist0_.id as id1_4_0_, 
foolist0_.id as id1_4_1_, foolist0_.BAR_ID as BAR_ID2_4_1_, 
foolist0_.bar_Id as bar_Id2_4_1_, foolist0_.name as name3_4_1_ 
from Foo foolist0_ 
where foolist0_.BAR_ID=? order by foolist0_.name asc 
```

第一个查询对父实体 `Bar`进行排序。生成第二个查询是为了对属于`Bar`的子`Foo`实体的集合进行排序。

## 3。使用 JPA 标准查询对象 API 排序

使用 JPA 标准——`orderBy`方法是设置所有排序参数的“一站式”替代方法:可以设置排序所依据的**订单方向**和**属性。以下是该方法的 API:**

*   **`orderBy`** ( `CriteriaBuilder.asc`):按升序排序。
*   **`orderBy`** ( `CriteriaBuilder.desc`):降序排列。

每个 `Order`实例都是通过 C `riteriaBuilder`对象的 `asc`或`desc`方法创建的。

这里有一个简单的例子——按照`name`对`Foos`进行排序:

```java
CriteriaQuery<Foo> criteriaQuery = criteriaBuilder.createQuery(Foo.class);
Root<Foo> from = criteriaQuery.from(Foo.class);
CriteriaQuery<Foo> select = criteriaQuery.select(from);
criteriaQuery.orderBy(criteriaBuilder.asc(from.get("name")));
```

t 方法的参数是区分大小写的，因为它需要匹配属性的名称。

与简单的 JQL 相反，JPA 标准查询对象 API **在查询中强制一个明确的订单方向**。注意在这段代码的最后一行中，`criteriaBuilder`对象通过调用它的`asc` 方法指定了升序排序。

当执行上述代码时，JPA 生成如下所示的 SQL 查询。JPA Criteria 对象生成一个带有显式 `asc`子句的 SQL 语句:

```java
Hibernate: select foo0_.id as id1_4_, foo0_.name as name2_4_
    from Foo foo0_ order by foo0_.name asc
```

### 3.1。按多个属性排序

要按多个属性排序，只需为每个要排序的属性向`orderBy`方法传递一个`Order`实例。

下面是一个简单的例子——分别按照`asc`和`desc`的顺序，按`name`和`id`排序:

```java
CriteriaQuery<Foo> criteriaQuery = criteriaBuilder.createQuery(Foo.class);
Root<Foo> from = criteriaQuery.from(Foo.class); 
CriteriaQuery<Foo> select = criteriaQuery.select(from); 
criteriaQuery.orderBy(criteriaBuilder.asc(from.get("name")),
    criteriaBuilder.desc(from.get("id")));
```

相应的 SQL 查询如下所示:

```java
Hibernate: select foo0_.id as id1_4_, foo0_.name as name2_4_ 
    from Foo foo0_ order by foo0_.name asc, foo0_.id desc
```

## 4。结论

本文探讨了 Java 持久性 API 中的排序方法，包括简单实体和一对多关系中的实体。这些方法将排序工作的负担委托给数据库层。

这个 JPA 排序教程的实现可以在[GitHub 项目](https://web.archive.org/web/20220815044534/https://github.com/eugenp/tutorials/tree/master/persistence-modules/spring-jpa "JPA Sort example project")中找到——这是一个基于 Maven 的项目，所以它应该很容易导入和运行。