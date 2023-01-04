# Spring 数据 JPA 查询示例

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-data-query-by-example>

## 1.介绍

在本教程中，我们将**学习如何使用[Spring Data](/web/20221129002814/https://www.baeldung.com/spring-data)[Query by Example API](https://web.archive.org/web/20221129002814/https://docs.spring.io/spring-data/jpa/docs/current/reference/html/#query-by-example)**查询数据。

首先，我们将定义我们想要查询的数据的模式。接下来，我们将检查 Spring 数据中的一些相关类。然后，我们将浏览几个例子。

我们开始吧！

## 2.测试数据

我们的测试数据是乘客姓名以及他们所占座位的列表。

| 西方人名的第一个字 | 姓 | 座位号 |
| 吉尔(女子名ˌ等于 Juliana) | 史密斯（姓氏） | Fifty |
| 前夕 | 杰克逊 | Ninety-four |
| 图像读取器设备（figure-reader electronic device 的缩写） | 布洛格斯 | Twenty-two |
| 里基 | 博比 | Thirty-six |
| 你呢 | Kolisi | eighty-five |

## 3.领域

让我们创建我们需要的 [Spring 数据仓库](/web/20221129002814/https://www.baeldung.com/spring-data-repositories)，并提供我们的域类和 id 类型。

首先，我们将`Passenger`建模为一个 JPA 实体:

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

不使用 JPA，我们可以把它建模成另一种抽象。

## 4.按示例 API 查询

首先，我们来看看`JpaRepository`界面。正如我们所见，它扩展了`[QueryByExampleExecutor](https://web.archive.org/web/20221129002814/https://docs.spring.io/spring-data/commons/docs/current/api/org/springframework/data/repository/query/QueryByExampleExecutor.html)` 接口以支持示例查询:

```java
public interface JpaRepository<T, ID>
  extends PagingAndSortingRepository<T, ID>, QueryByExampleExecutor<T> {}
```

这个接口引入了更多我们从 Spring 数据中熟悉的`find()`方法的变体。但是，每个方法也接受一个 [`Example`](https://web.archive.org/web/20221129002814/https://docs.spring.io/spring-data/commons/docs/current/api/org/springframework/data/domain/Example.html) 的实例:

```java
public interface QueryByExampleExecutor<T> {
    <S extends T> Optional<S> findOne(Example<S> var1);
    <S extends T> Iterable<S> findAll(Example<S> var1);
    <S extends T> Iterable<S> findAll(Example<S> var1, Sort var2);
    <S extends T> Page<S> findAll(Example<S> var1, Pageable var2);
    <S extends T> long count(Example<S> var1);
    <S extends T> boolean exists(Example<S> var1);
}
```

其次，`Example`接口公开了访问`probe`和 [`ExampleMatcher`](https://web.archive.org/web/20221129002814/https://docs.spring.io/spring-data/commons/docs/current/api/org/springframework/data/domain/ExampleMatcher.html) 的方法。

认识到`probe`是我们的`Entity`的实例是很重要的:

```java
public interface Example<T> {

    static <T> org.springframework.data.domain.Example<T> of(T probe) {
        return new TypedExample(probe, ExampleMatcher.matching());
    }

    static <T> org.springframework.data.domain.Example<T> of(T probe, ExampleMatcher matcher) {
        return new TypedExample(probe, matcher);
    }

    T getProbe();

    ExampleMatcher getMatcher();

    default Class<T> getProbeType() {
        return ProxyUtils.getUserClass(this.getProbe().getClass());
    }
}
```

总之，我们的`probe`和`ExampleMatcher`一起指定了我们的查询。

## 5.限制

像所有事情一样，按示例查询 API 也有一些限制。例如:

*   不支持嵌套和分组语句，例如: `(`名 `= ?0 and` 姓 `= ?1) or` `seatNumber` `= ?2`
*   字符串匹配仅包括精确、不区分大小写、开始、结束、包含和正则表达式
*   除了`String`之外的所有类型都是精确匹配的

现在我们对 API 及其局限性有了更多的了解，让我们深入一些例子。

## 6.例子

### 6.1.区分大小写的匹配

让我们从一个简单的例子开始，讨论默认行为:

```java
@Test
public void givenPassengers_whenFindByExample_thenExpectedReturned() {
    Example<Passenger> example = Example.of(Passenger.from("Fred", "Bloggs", null));

    Optional<Passenger> actual = repository.findOne(example);

    assertTrue(actual.isPresent());
    assertEquals(Passenger.from("Fred", "Bloggs", 22), actual.get());
}
```

特别是，静态的`Example.of()`方法使用`ExampleMatcher.matching()`构建了一个`Example`。

换句话说，**将对`Passenger`的所有非空属性**进行精确匹配。因此，`String`属性的匹配是区分大小写的。

然而，如果我们所能做的只是对所有非空属性进行精确匹配，那就没什么用了。

这就是`ExampleMatcher`的用武之地。通过构建我们自己的`ExampleMatcher`，我们可以定制行为来满足我们的需求。

### 6.2.不区分大小写的匹配

记住这一点，让我们看看另一个例子，这次使用`withIgnoreCase()`来实现不区分大小写的匹配:

```java
@Test
public void givenPassengers_whenFindByExampleCaseInsensitiveMatcher_thenExpectedReturned() {
    ExampleMatcher caseInsensitiveExampleMatcher = ExampleMatcher.matchingAll().withIgnoreCase();
    Example<Passenger> example = Example.of(Passenger.from("fred", "bloggs", null),
      caseInsensitiveExampleMatcher);

    Optional<Passenger> actual = repository.findOne(example);

    assertTrue(actual.isPresent());
    assertEquals(Passenger.from("Fred", "Bloggs", 22), actual.get());
}
```

在这个例子中，请注意我们首先调用了`ExampleMatcher.matchingAll() –`，它的行为与我们在前一个例子中使用的`ExampleMatcher.matching()`相同。

### 6.3.自定义匹配

我们还可以**基于每个属性**调整匹配器的行为，并使用`ExampleMatcher.matchingAny()`匹配任何属性:

```java
@Test
public void givenPassengers_whenFindByExampleCustomMatcher_thenExpectedReturned() {
    Passenger jill = Passenger.from("Jill", "Smith", 50);
    Passenger eve = Passenger.from("Eve", "Jackson", 95);
    Passenger fred = Passenger.from("Fred", "Bloggs", 22);
    Passenger siya = Passenger.from("Siya", "Kolisi", 85);
    Passenger ricki = Passenger.from("Ricki", "Bobbie", 36);

    ExampleMatcher customExampleMatcher = ExampleMatcher.matchingAny()
      .withMatcher("firstName", ExampleMatcher.GenericPropertyMatchers.contains().ignoreCase())
      .withMatcher("lastName", ExampleMatcher.GenericPropertyMatchers.contains().ignoreCase());

    Example<Passenger> example = Example.of(Passenger.from("e", "s", null), customExampleMatcher);

    List<Passenger> passengers = repository.findAll(example);

    assertThat(passengers, contains(jill, eve, fred, siya));
    assertThat(passengers, not(contains(ricki)));
}
```

### 6.4.忽略属性

另一方面，我们也可能只想对属性的子集进行**查询。**

我们通过使用`ExampleMatcher.ignorePaths(String… paths)`忽略一些属性来实现这一点:

```java
@Test
public void givenPassengers_whenFindByIgnoringMatcher_thenExpectedReturned() {
    Passenger jill = Passenger.from("Jill", "Smith", 50); 
    Passenger eve = Passenger.from("Eve", "Jackson", 95); 
    Passenger fred = Passenger.from("Fred", "Bloggs", 22);
    Passenger siya = Passenger.from("Siya", "Kolisi", 85);
    Passenger ricki = Passenger.from("Ricki", "Bobbie", 36);

    ExampleMatcher ignoringExampleMatcher = ExampleMatcher.matchingAny()
      .withMatcher("lastName", ExampleMatcher.GenericPropertyMatchers.startsWith().ignoreCase())
      .withIgnorePaths("firstName", "seatNumber");

    Example<Passenger> example = Example.of(Passenger.from(null, "b", null), ignoringExampleMatcher);

    List<Passenger> passengers = repository.findAll(example);

    assertThat(passengers, contains(fred, ricki));
    assertThat(passengers, not(contains(jill));
    assertThat(passengers, not(contains(eve)); 
    assertThat(passengers, not(contains(siya)); 
}
```

## 7.结论

在本文中，我们演示了如何使用示例查询 API。

我们已经演示了如何使用 [`Example`](https://web.archive.org/web/20221129002814/https://docs.spring.io/spring-data/commons/docs/current/api/org/springframework/data/domain/Example.html) 和 [`ExampleMatcher`](https://web.archive.org/web/20221129002814/https://docs.spring.io/spring-data/commons/docs/current/api/org/springframework/data/domain/ExampleMatcher.html) 以及`[QueryByExampleExecutor](https://web.archive.org/web/20221129002814/https://docs.spring.io/spring-data/commons/docs/current/api/org/springframework/data/repository/query/QueryByExampleExecutor.html)`接口，通过一个示例数据实例来查询一个表。

总之，你可以在 GitHub 上找到代码[。](https://web.archive.org/web/20221129002814/https://github.com/eugenp/tutorials/tree/master/persistence-modules/spring-data-jpa-query)