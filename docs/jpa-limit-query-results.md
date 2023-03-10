# 用 JPA 和 Spring 数据 JPA 限制查询结果

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/jpa-limit-query-results>

## 1.介绍

在本教程中，我们将**学习用 [JPA](https://web.archive.org/web/20221012184424/https://en.wikipedia.org/wiki/Java_Persistence_API) 和 [Spring Data JPA](https://web.archive.org/web/20221012184424/https://spring.io/projects/spring-data-jpa) 限制查询结果**。

首先，我们将看看我们想要查询的表，以及我们想要重现的 SQL 查询。

然后我们将深入探讨如何使用 JPA 和 Spring Data JPA 实现这一点。

我们开始吧！

## 2.测试数据

下面是我们将在本文中查询的表格。

我们要回答的问题是，“第一个被占用的座位是什么，谁在占用它？”

| 西方人名的第一个字 | 姓 | 座位号 |
| 吉尔(女子名ˌ等于 Juliana) | 史密斯（姓氏） | Fifty |
| 前夕 | 杰克逊 | Ninety-four |
| 图像读取器设备（figure-reader electronic device 的缩写） | 布洛格斯 | Twenty-two |
| 里基 | 博比 | Thirty-six |
| 你呢 | Kolisi | eighty-five |

## 3.SQL

使用 SQL，我们可能会编写一个如下所示的查询:

```java
SELECT firstName, lastName, seatNumber FROM passengers ORDER BY seatNumber LIMIT 1;
```

## 4.JPA 设置

对于 JPA，我们首先需要一个实体来映射我们的表:

```java
@Entity
class Passenger {

    @Id
    @GeneratedValue
    @Column(nullable = false)
    private Long id;

    @Basic(optional = false)
    @Column(nullable = false)
    private String fistName;

    @Basic(optional = false)
    @Column(nullable = false)
    private String lastName;

    @Basic(optional = false)
    @Column(nullable = false)
    private int seatNumber;

    // constructor, getters etc.
}
```

接下来，我们需要一个封装查询代码的方法，在这里实现为`PassengerRepositoryImpl#findOrderedBySeatNumberLimitedTo(int limit)`:

```java
@Repository
class PassengerRepositoryImpl {

    @PersistenceContext
    private EntityManager entityManager;

    @Override
    public List<Passenger> findOrderedBySeatNumberLimitedTo(int limit) {
        return entityManager.createQuery("SELECT p FROM Passenger p ORDER BY p.seatNumber",
          Passenger.class).setMaxResults(limit).getResultList();
    }
}
```

在我们的存储库方法中，我们使用 [`EntityManager`](https://web.archive.org/web/20221012184424/https://docs.oracle.com/javaee/7/api/javax/persistence/EntityManager.html) 来创建一个`[Query](https://web.archive.org/web/20221012184424/https://docs.oracle.com/javaee/7/api/javax/persistence/Query.html)`，我们称之为`[setMaxResults()](https://web.archive.org/web/20221012184424/https://docs.oracle.com/javaee/7/api/javax/persistence/Query.html#setMaxResults-int-)`方法。

这个对 [`Query#setMaxResults`](https://web.archive.org/web/20221012184424/https://docs.oracle.com/javaee/7/api/javax/persistence/Query.html#setMaxResults-int-) 的调用最终会将 limit 语句附加到生成的 SQL 语句中:

```java
select
  passenger0_.id as id1_15_,
  passenger0_.fist_name as fist_nam2_15_,
  passenger0_.last_name as last_nam3_15_,
  passenger0_.seat_number as seat_num4_15_
from passenger passenger0_ order by passenger0_.seat_number limit ?
```

## 5.含春季数据 JPA

我们也可以使用 Spring 数据 JPA 来生成我们的 SQL。

### 5.1.`first`或`top`

一种方法是使用带有关键字`first`或`top.`的方法名派生

我们可以选择指定一个数字作为返回的最大结果大小。如果我们省略它，Spring Data JPA 假设结果大小为 1。

因为我们想知道哪个座位是第一个被占用的，谁在占用它，所以我们可以通过以下两种方式省略号码:

```java
Passenger findFirstByOrderBySeatNumberAsc();
Passenger findTopByOrderBySeatNumberAsc();
```

如果我们限制到一个实例结果，如上所述，那么我们也可以使用`Optional`包装结果:

```java
Optional<Passenger> findFirstByOrderBySeatNumberAsc();
Optional<Passenger> findTopByOrderBySeatNumberAsc();
```

### 5.2.`Pageable`

或者，我们可以使用一个 [`Pageable`](https://web.archive.org/web/20221012184424/https://docs.spring.io/spring-data/commons/docs/current/api/org/springframework/data/domain/Pageable.html) 对象:

```java
Page<Passenger> page = repository.findAll(
  PageRequest.of(0, 1, Sort.by(Sort.Direction.ASC, "seatNumber")));
```

如果我们看一下`[SimpleJpaRepository](https://web.archive.org/web/20221012184424/https://docs.spring.io/spring-data/jpa/docs/current/api/org/springframework/data/jpa/repository/support/SimpleJpaRepository.html), `的默认实现`[JpaRepository,](https://web.archive.org/web/20221012184424/https://docs.spring.io/spring-data/jpa/docs/current/api/org/springframework/data/jpa/repository/JpaRepository.html)`，我们可以看到它也调用了`[Query#setMaxResults](https://web.archive.org/web/20221012184424/https://docs.oracle.com/javaee/7/api/javax/persistence/Query.html#setMaxResults-int-)`:

```java
protected <S extends T > Page < S > readPage(TypedQuery < S > query, 
  Class < S > domainClass, Pageable pageable,
  @Nullable Specification < S > spec) {
    if (pageable.isPaged()) {
        query.setFirstResult((int) pageable.getOffset());
        query.setMaxResults(pageable.getPageSize());
    }

    return PageableExecutionUtils.getPage(query.getResultList(), pageable, () -> {
        return executeCountQuery(this.getCountQuery(spec, domainClass));
    });
}
```

### 5.3.比较

这两种选择都会产生我们想要的 SQL，其中`first `和`top`倾向于约定，`Pageable`倾向于配置:

```java
select
  passenger0_.id as id1_15_,
  passenger0_.fist_name as fist_nam2_15_,
  passenger0_.last_name as last_nam3_15_,
  passenger0_.seat_number as seat_num4_15_ 
from passenger passenger0_ order by passenger0_.seat_number asc limit ?
```

## 6.结论

在 [JPA](https://web.archive.org/web/20221012184424/https://en.wikipedia.org/wiki/Java_Persistence_API) 中限制查询结果与 SQL 略有不同；我们没有将 limit 关键字直接包含到我们的 [JPQL](https://web.archive.org/web/20221012184424/https://en.wikipedia.org/wiki/Java_Persistence_Query_Language) 中。

相反，我们只是对 [`Query#maxResults`](https://web.archive.org/web/20221012184424/https://docs.spring.io/spring-data/jpa/docs/current/api/org/springframework/data/jpa/repository/support/SimpleJpaRepository.html) 进行单个方法调用，或者在我们的 [Spring Data JPA](https://web.archive.org/web/20221012184424/https://docs.spring.io/spring-data/jpa/docs/current/reference/html/#repositories.limit-query-result) 方法名中包含关键字`first`或者`top`。

和往常一样，代码可以在 GitHub 的[上获得。](https://web.archive.org/web/20221012184424/https://github.com/eugenp/tutorials/tree/master/persistence-modules/spring-data-jpa-query)