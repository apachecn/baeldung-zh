# 休眠分页

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/hibernate-pagination>

## 1。概述

这篇文章是对 Hibernate 中分页的快速介绍。我们将看看标准的 HQL 以及`ScrollableResults` API，最后，看看 Hibernate 的分页标准。

## 延伸阅读:

## [用 Spring 引导 Hibernate 5](/web/20221018034507/https://www.baeldung.com/hibernate-5-spring)

A quick and practical guide to integrating Hibernate 5 with Spring.[Read more](/web/20221018034507/https://www.baeldung.com/hibernate-5-spring) →

## [Hibernate 继承映射](/web/20221018034507/https://www.baeldung.com/hibernate-inheritance)

A practical guide to understanding different inheritance mapping strategies with JPA / Hibernate.[Read more](/web/20221018034507/https://www.baeldung.com/hibernate-inheritance) →

## [显示来自 Spring Boot 的 Hibernate/JPA SQL 语句](/web/20221018034507/https://www.baeldung.com/sql-logging-spring-boot)

Learn how you can configure logging of the generated SQL statements in your Spring Boot application.[Read more](/web/20221018034507/https://www.baeldung.com/sql-logging-spring-boot) →

## 2。用 HQL 和`setFirstResult,` `setMaxResults` API 分页

在 Hibernate 中进行分页最简单也是最常见的方式是使用 HQL 的**:**

```java
Session session = sessionFactory.openSession();
Query query = sess.createQuery("From Foo");
query.setFirstResult(0);
query.setMaxResults(10);
List<Foo> fooList = fooList = query.list();
```

这个例子使用了一个基本的`Foo`实体，非常类似于 JQL 实现的 JPA 唯一的区别是查询语言。

如果我们为 Hibernate 打开**日志，我们将看到下面的 SQL 正在运行:**

```java
Hibernate: 
    select
        foo0_.id as id1_1_,
        foo0_.name as name2_1_ 
    from
        Foo foo0_ limit ?
```

### 2.1。总计数和最后一页

在不知道实体总数的情况下，分页解决方案是不完整的:

```java
String countQ = "Select count (f.id) from Foo f";
Query countQuery = session.createQuery(countQ);
Long countResults = (Long) countQuery.uniqueResult();
```

最后，根据总数和给定的页面大小，我们可以计算出最后一页的大小:

```java
int pageSize = 10;
int lastPageNumber = (int) (Math.ceil(countResults / pageSize));
```

此时，我们可以看一下**分页**的完整示例，其中我们计算最后一页，然后检索它:

```java
@Test
public void givenEntitiesExist_whenRetrievingLastPage_thenCorrectSize() {
    int pageSize = 10;
    String countQ = "Select count (f.id) from Foo f";
    Query countQuery = session.createQuery(countQ);
    Long countResults = (Long) countQuery.uniqueResult();
    int lastPageNumber = (int) (Math.ceil(countResults / pageSize));

    Query selectQuery = session.createQuery("From Foo");
    selectQuery.setFirstResult((lastPageNumber - 1) * pageSize);
    selectQuery.setMaxResults(pageSize);
    List<Foo> lastPage = selectQuery.list();

    assertThat(lastPage, hasSize(lessThan(pageSize + 1)));
}
```

## 3。使用 HQL 和 ScrollableResults API 使用 Hibernate 分页

使用`ScrollableResul` ts 实现分页有可能**减少数据库调用**。这种方法在程序滚动时对结果集进行流式处理，因此不需要重复查询来填充每个页面:

```java
String hql = "FROM Foo f order by f.name";
Query query = session.createQuery(hql);
int pageSize = 10;

ScrollableResults resultScroll = query.scroll(ScrollMode.FORWARD_ONLY);
resultScroll.first();
resultScroll.scroll(0);
List<Foo> fooPage = Lists.newArrayList();
int i = 0;
while (pageSize > i++) {
    fooPage.add((Foo) resultScroll.get(0));
    if (!resultScroll.next())
        break;
}
```

这种方法不仅省时(只有一次数据库调用)，而且允许用户访问结果集的总计数**，而不需要额外的查询**:

```java
resultScroll.last();
int totalResults = resultScroll.getRowNumber() + 1;
```

另一方面，请记住，虽然滚动非常有效，但是大窗口可能会占用相当多的内存。

## 4。使用标准 API 使用 Hibernate 分页

最后，让我们看看**一个更灵活的解决方案**——使用标准:

```java
Criteria criteria = session.createCriteria(Foo.class);
criteria.setFirstResult(0);
criteria.setMaxResults(pageSize);
List<Foo> firstPage = criteria.list();
```

Hibernate Criteria query API 也使得**获得总计数**变得非常简单——通过使用一个`Projection`对象:

```java
Criteria criteriaCount = session.createCriteria(Foo.class);
criteriaCount.setProjection(Projections.rowCount());
Long count = (Long) criteriaCount.uniqueResult();
```

正如你所看到的，使用这个 API 会产生比普通 HQL 更少的冗长代码，但是**这个 API 是完全类型安全的，并且更加灵活**。

## 5。结论

本文简要介绍了在 Hibernate 中进行分页的各种方法。

这个 Spring JPA 教程的实现可以在 GitHub 项目中找到——这是一个基于 Eclipse 的项目，所以它应该很容易导入和运行。