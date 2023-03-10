# Spring 数据中的 CrudRepository、JpaRepository 和 PagingAndSortingRepository

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-data-repositories>

## 1。概述

在这篇简短的文章中，我们将关注不同种类的 Spring 数据存储库接口及其功能。我们将谈到:

*   `CrudRepository`
*   `PagingAndSortingRepository`
*   `JpaRepository`

简单地说， [Spring Data](https://web.archive.org/web/20220816182031/https://projects.spring.io/spring-data/) 中的每个存储库都扩展了通用的`Repository`接口，但除此之外，它们都有不同的功能。

## 2。Spring 数据仓库

`Let's start with the [JpaRepository](https://web.archive.org/web/20220816182031/http://static.springsource.org/spring-data/data-jpa/docs/current/api/org/springframework/data/jpa/repository/JpaRepository.html)`–扩展`[PagingAndSortingRepository](https://web.archive.org/web/20220816182031/http://static.springsource.org/spring-data/data-commons/docs/current/api/org/springframework/data/repository/PagingAndSortingRepository.html)`，进而扩展`[CrudRepository](https://web.archive.org/web/20220816182031/http://static.springsource.org/spring-data/data-commons/docs/current/api/org/springframework/data/repository/CrudRepository.html)`。

其中每个都定义了自己的功能:

*   `[CrudRepository](https://web.archive.org/web/20220816182031/http://static.springsource.org/spring-data/data-commons/docs/current/api/org/springframework/data/repository/CrudRepository.html)`提供 CRUD 功能
*   提供对记录进行分页和排序的方法
*   `[JpaRepository](https://web.archive.org/web/20220816182031/http://static.springsource.org/spring-data/data-jpa/docs/current/api/org/springframework/data/jpa/repository/JpaRepository.html)`提供了 JPA 相关的方法，比如批量刷新持久上下文和删除记录

因此，由于这种继承关系， **`JpaRepository`包含了`CrudRepository`和`PagingAndSortingRepository`的完整 API。**

当我们不需要由`JpaRepository`和`PagingAndSortingRepository`提供的全部功能时，我们可以简单地使用`CrudRepository`。

现在让我们看一个简单的例子来更好地理解这些 API。

我们将从一个简单的`Product`实体开始:

```java
@Entity
public class Product {

    @Id
    private long id;
    private String name;

    // getters and setters
}
```

让我们实现一个简单的操作——根据名称找到一个`Product`:

```java
@Repository
public interface ProductRepository extends JpaRepository<Product, Long> {
    Product findByName(String productName);
}
```

仅此而已。Spring 数据存储库将根据我们提供的名称自动生成实现。

这当然是一个非常简单的例子。这里可以更深入的了解 Spring Data JPA [。](/web/20220816182031/https://www.baeldung.com/the-persistence-layer-with-spring-data-jpa)

## 3。`CrudRepository`

现在让我们来看看 [`CrudRepository`](https://web.archive.org/web/20220816182031/http://static.springsource.org/spring-data/data-commons/docs/current/api/org/springframework/data/repository/CrudRepository.html) 界面的代码:

```java
public interface CrudRepository<T, ID extends Serializable>
  extends Repository<T, ID> {

    <S extends T> S save(S entity);

    T findOne(ID primaryKey);

    Iterable<T> findAll();

    Long count();

    void delete(T entity);

    boolean exists(ID primaryKey);
}
```

请注意典型的 CRUD 功能:

*   `save(…) – s`拥有一个`Iterable`的实体。在这里，我们可以传递多个对象来批量保存它们
*   `findOne(…)`–根据传递的主键值获取单个实体
*   `findAll()`–获取数据库中所有可用实体的`Iterable`
*   `count() – r`返回表格中实体总数
*   `delete(…)`–删除基于传递对象的实体
*   exists(…)-根据传递的主键值验证实体是否存在

这个接口看起来非常普通和简单，但实际上，它提供了应用程序中所需的所有基本查询抽象。

## 4。`PagingAndSortingRepository`

现在，让我们看看另一个存储库接口，它扩展了`CrudRepository`:

```java
public interface PagingAndSortingRepository<T, ID extends Serializable> 
  extends CrudRepository<T, ID> {

    Iterable<T> findAll(Sort sort);

    Page<T> findAll(Pageable pageable);
}
```

这个接口提供了一个方法`findAll(Pageable pageable)`，这是实现`Pagination.`的关键

当使用 `Pageable`时，我们创建一个具有某些属性的`Pageable`对象，我们必须至少指定:

1.  页面大小
2.  当前页码
3.  整理

因此，让我们假设我们想要显示按`lastName,`升序排序的结果集的第一页，每一页不超过五条记录。这就是我们如何使用一个`PageRequest`和一个`Sort`定义来实现的:

```java
Sort sort = new Sort(new Sort.Order(Direction.ASC, "lastName"));
Pageable pageable = new PageRequest(0, 5, sort);
```

将 pageable 对象传递给 Spring 数据查询将返回有问题的结果(第一个参数`PageRequest`是从零开始的)。

## 5。`JpaRepository`

最后，我们来看看 [`JpaRepository`](https://web.archive.org/web/20220816182031/http://static.springsource.org/spring-data/data-jpa/docs/current/api/org/springframework/data/jpa/repository/JpaRepository.html) 界面:

```java
public interface JpaRepository<T, ID extends Serializable> extends
  PagingAndSortingRepository<T, ID> {

    List<T> findAll();

    List<T> findAll(Sort sort);

    List<T> save(Iterable<? extends T> entities);

    void flush();

    T saveAndFlush(T entity);

    void deleteInBatch(Iterable<T> entities);
}
```

同样，让我们简单地看一下这些方法:

*   `findAll()`–获取数据库中所有可用实体的`List`
*   `findAll(…)`–获取所有可用实体的`List`，并使用提供的条件对它们进行排序
*   `save(…) – s`拥有一个`Iterable`的实体。在这里，我们可以传递多个对象来批量保存它们
*   `flush() – f`将所有未完成的任务刷新到数据库中
*   `saveAndFlush(…)`–保存实体并立即刷新更改
*   deleteInBatch(…)–删除一个`Iterable`实体。在这里，我们可以传递多个对象来批量删除它们

很明显，上面的接口扩展了`PagingAndSortingRepository` ，这意味着它也包含了`CrudRepository`中的所有方法。

## 6。Spring 数据仓库的缺点

除了这些存储库的所有非常有用的优点之外，直接依赖它们也有一些基本的缺点:

1.  我们将代码耦合到库和它的特定抽象，比如‘page’或‘page able ’;这当然不是这个库独有的——但是我们必须小心不要暴露这些内部实现细节
2.  通过扩展例如`CrudRepository`，我们立刻公开了一套完整的持久化方法。这在大多数情况下可能也没问题，但是我们可能会遇到这样的情况，我们想要对公开的方法进行更细粒度的控制，例如创建一个不包括`CrudRepository`的`save(…)`和`delete(…)`方法的`ReadOnlyRepository`

## 7。结论

本文介绍了 Spring Data JPA 存储库接口的一些简短但重要的差异和特性。

要了解更多信息，请看关于[弹簧持久性](/web/20220816182031/https://www.baeldung.com/persistence-with-spring-series/)的系列文章。