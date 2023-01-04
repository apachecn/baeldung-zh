# JPA 分页

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/jpa-pagination>

## 1。概述

本文说明了如何在 Java 持久性 API 中实现**分页。**

它解释了如何使用基本 JQL 和更安全的基于类型标准的 API 进行分页，讨论了每个实现的优点和已知问题。

## 延伸阅读:

## [用弹簧座和角度表分页](/web/20221027152928/https://www.baeldung.com/pagination-with-a-spring-rest-api-and-an-angularjs-table)

An extensive look at how to implement a simple API with pagination with Spring and how to consume it with AngularJS and UI Grid.[Read more](/web/20221027152928/https://www.baeldung.com/pagination-with-a-spring-rest-api-and-an-angularjs-table) →

## [Spring JPA–多个数据库](/web/20221027152928/https://www.baeldung.com/spring-data-jpa-multiple-databases)

How to set up Spring Data JPA to work with multiple, separate databases.[Read more](/web/20221027152928/https://www.baeldung.com/spring-data-jpa-multiple-databases) →

## [春季数据 JPA @Query](/web/20221027152928/https://www.baeldung.com/spring-data-jpa-query)

Learn how to use the @Query annotation in Spring Data JPA to define custom queries using JPQL and native SQL.[Read more](/web/20221027152928/https://www.baeldung.com/spring-data-jpa-query) →

## 2。用 JQL 和`setFirstResult()`、 `setMaxResults()` API 分页

实现分页最简单的方法是使用**Java 查询语言**——创建一个查询，并通过`setMaxResults` 和`s`e` tFirstResult` 对其进行配置:

```java
Query query = entityManager.createQuery("From Foo");
int pageNumber = 1;
int pageSize = 10;
query.setFirstResult((pageNumber-1) * pageSize); 
query.setMaxResults(pageSize);
List <Foo> fooList = query.getResultList();
```

API 很简单:

*   `**setFirstResult(int**)`:设置结果集中开始分页的偏移位置
*   `**setMaxResults(int)**`:设置页面中应包含的实体的最大数量

### 2.1。总计数和最后一页

对于一个更完整的分页解决方案，我们还需要获得总结果计数:

```java
Query queryTotal = entityManager.createQuery
    ("Select count(f.id) from Foo f");
long countResult = (long)queryTotal.getSingleResult();
```

计算**最后一页**也很有用:

```java
int pageSize = 10;
int pageNumber = (int) ((countResult / pageSize) + 1);
```

注意，这种获取结果集总计数的方法需要额外的查询(针对计数)。

## 3。使用实体的 Id 与 JQL 分页

另一种简单的分页策略是**首先检索完整的 id**，然后基于这些 id**检索完整的实体**。这允许更好地控制实体提取，但也意味着它需要加载整个表来检索 id:

```java
Query queryForIds = entityManager.createQuery(
  "Select f.id from Foo f order by f.lastName");
List<Integer> fooIds = queryForIds.getResultList();
Query query = entityManager.createQuery(
  "Select f from Foo e where f.id in :ids");
query.setParameter("ids", fooIds.subList(0,10));
List<Foo> fooList = query.getResultList();
```

最后，还要注意，它需要两个不同的查询来检索完整的结果。

## 4。使用标准 API 使用 JPA 分页

接下来，让我们看看如何**利用 JPA 标准 API** 实现分页:

```java
int pageSize = 10;
CriteriaBuilder criteriaBuilder = entityManager
  .getCriteriaBuilder();
CriteriaQuery<Foo> criteriaQuery = criteriaBuilder
  .createQuery(Foo.class);
Root<Foo> from = criteriaQuery.from(Foo.class);
CriteriaQuery<Foo> select = criteriaQuery.select(from);
TypedQuery<Foo> typedQuery = entityManager.createQuery(select);
typedQuery.setFirstResult(0);
typedQuery.setMaxResults(pageSize);
List<Foo> fooList = typedQuery.getResultList();
```

当目标是创建动态的、故障安全的查询时，这很有用。与“硬编码”、“基于字符串”的 JQL 或 HQL 查询相比，`JPA Criteria`减少了运行时故障，因为编译器会动态检查查询错误。

使用 JPA 标准**获得实体总数**非常简单:

```java
CriteriaQuery<Long> countQuery = criteriaBuilder
  .createQuery(Long.class);
countQuery.select(criteriaBuilder.count(
  countQuery.from(Foo.class)));
Long count = entityManager.createQuery(countQuery)
  .getSingleResult();
```

最终结果是**一个完整的分页解决方案**，使用 JPA 标准 API:

```java
int pageNumber = 1;
int pageSize = 10;
CriteriaBuilder criteriaBuilder = entityManager.getCriteriaBuilder();

CriteriaQuery<Long> countQuery = criteriaBuilder
  .createQuery(Long.class);
countQuery.select(criteriaBuilder
  .count(countQuery.from(Foo.class)));
Long count = entityManager.createQuery(countQuery)
  .getSingleResult();

CriteriaQuery<Foo> criteriaQuery = criteriaBuilder
  .createQuery(Foo.class);
Root<Foo> from = criteriaQuery.from(Foo.class);
CriteriaQuery<Foo> select = criteriaQuery.select(from);

TypedQuery<Foo> typedQuery = entityManager.createQuery(select);
while (pageNumber < count.intValue()) {
    typedQuery.setFirstResult(pageNumber - 1);
    typedQuery.setMaxResults(pageSize);
    System.out.println("Current page: " + typedQuery.getResultList());
    pageNumber += pageSize;
}
```

## 5。结论

本文探讨了 JPA 中可用的基本分页选项。

有些有缺点——主要与查询性能有关，但这些缺点通常会被改进的控制和整体灵活性所抵消。

这个 Spring JPA 教程的实现可以在[GitHub 项目](https://web.archive.org/web/20221027152928/https://github.com/eugenp/tutorials/tree/master/persistence-modules/spring-jpa "Spring JPA Tutorial - example project")中找到——这是一个基于 Maven 的项目，所以它应该很容易导入和运行。