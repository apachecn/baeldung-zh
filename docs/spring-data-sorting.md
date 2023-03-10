# 使用 Spring 数据对查询结果进行排序

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-data-sorting>

## 1.介绍

在本教程中，我们将**学习如何用 [Spring 数据](/web/20220707143830/https://www.baeldung.com/spring-data)对查询结果**进行排序。

首先，我们将看看我们想要查询和排序的数据的模式。然后我们将讨论如何用 Spring 数据实现这一点。

我们开始吧！

## 2.测试数据

下面我们有一些样本数据。尽管我们在这里将它表示为一个表，但是我们可以使用 Spring Data 支持的任何一个数据库来持久化它。

我们要回答的问题是，“谁占据了航线上的哪个座位？”为了更加方便用户，我们将按座位号排序。

| 西方人名的第一个字 | 姓 | 座位号 |
| 吉尔(女子名ˌ等于 Juliana) | 史密斯（姓氏） | Fifty |
| 前夕 | 杰克逊 | Ninety-four |
| 图像读取器设备（figure-reader electronic device 的缩写） | 布洛格斯 | Twenty-two |
| 里基 | 博比 | Thirty-six |
| 你呢 | Kolisi | eighty-five |

## 3.领域

为了创建一个 [Spring 数据仓库](https://web.archive.org/web/20220707143830/https://docs.spring.io/spring-data/data-commons/docs/current/reference/html/#repositories.core-concepts)，我们需要提供一个域类，以及一个 id 类型。

在这里，我们将我们的乘客建模为 JPA 实体，但是我们也可以像对 MongoDB 文档或任何其他模型抽象一样简单地对其建模:

```java
@Entity
class Passenger {

    @Id
    @GeneratedValue
    @Column(nullable = false)
    private Long id;

    @Basic(optional = false)
    @Column(nullable = false)
    private String firstName;

    @Basic(optional = false)
    @Column(nullable = false)
    private String lastName;

    @Basic(optional = false)
    @Column(nullable = false)
    private int seatNumber;

    // constructor, getters etc.
}
```

## 4.使用 Spring 数据排序

对于 Spring 数据的排序，我们有几种不同的选择。

### 4.1。使用`OrderBy`方法关键字进行排序

一种选择是使用 Spring Data 的方法派生，从而根据方法名和签名生成查询。

**在这里，我们需要做的就是在我们的方法名**中包含关键字`OrderBy`，以及我们想要排序的属性名和方向(Asc 或 Desc)。

我们可以使用这个约定创建一个查询，按照座位号以升序返回我们的乘客:

```java
interface PassengerRepository extends JpaRepository<Passenger, Long> {

    List<Passenger> findByOrderBySeatNumberAsc();
}
```

我们还可以将这个关键字与所有标准的 Spring 数据方法名称结合起来。

让我们看一个根据姓氏`and`查找乘客，根据座位号排序的方法示例:

```java
List<Passenger> findByLastNameOrderBySeatNumberAsc(String lastName);
```

### 4.2.使用`Sort`参数进行分类

**我们的第二个选项是包含一个`[Sort](https://web.archive.org/web/20220707143830/https://docs.spring.io/spring-data/commons/docs/current/api/org/springframework/data/domain/Sort.html)`参数**，指定我们想要排序的属性名和方向:

```java
List<Passenger> passengers = repository.findAll(Sort.by(Sort.Direction.ASC, "seatNumber"));
```

在本例中，我们使用了`findAll()`方法，并在调用它时添加了`Sort`选项。

我们还可以将此参数添加到新的方法定义中:

```java
List<Passenger> findByLastName(String lastName, Sort sort);
```

最后，如果我们正在分页，我们可以在一个 [`Pageable`](https://web.archive.org/web/20220707143830/https://docs.spring.io/spring-data/commons/docs/current/api/org/springframework/data/domain/Pageable.html) 对象中指定我们的排序:

```java
Page<Passenger> page = repository.findAll(PageRequest.of(0, 1, Sort.by(Sort.Direction.ASC, "seatNumber")));
```

## 5.结论

我们有两个简单的选择来对 Spring 数据进行排序:使用`OrderBy`关键字进行方法派生，或者使用`[Sort](https://web.archive.org/web/20220707143830/https://docs.spring.io/spring-data/commons/docs/current/api/org/springframework/data/domain/Sort.html)`对象作为方法参数。

和往常一样，本文中使用的代码可以从 GitHub 上的[处获得。](https://web.archive.org/web/20220707143830/https://github.com/eugenp/tutorials/tree/master/persistence-modules/spring-data-jpa-query)