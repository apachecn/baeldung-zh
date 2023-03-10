# Spring 数据 JPA 派生的删除方法

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-data-jpa-deleteby>

## 1.介绍

Spring Data JPA 允许我们定义从数据库中读取、更新或删除记录的[派生方法](/web/20221128041423/https://www.baeldung.com/the-persistence-layer-with-spring-data-jpa)。这非常有帮助，因为它减少了数据访问层的样板代码。

在本教程中，**我们将通过实际的代码例子来关注定义和使用 Spring 数据派生的删除方法**。

## 2.派生的`deleteBy`方法

首先，让我们建立我们的例子。我们将定义一个`Fruit`实体来保存水果店中商品的名称和颜色:

```java
@Entity
public class Fruit {
    @Id
    private long id;
    private String name;
    private String color;
    // standard getters and setters
}
```

接下来，我们将通过扩展`JpaRepository`接口并向该类添加我们的派生方法来添加我们的存储库以操作`Fruit`实体。

**派生方法可以定义为动词+实体中定义的属性。**一些允许的动词是`findBy, deleteBy,` 和`removeBy`。

让我们来推导一个方法，通过`name`删除`Fruit`:

```java
@Repository
public interface FruitRepository extends JpaRepository<Fruit, Long> {
    Long deleteByName(String name);
}
```

在这个例子中，`deleteByName`方法返回已删除记录的计数。

类似地，我们也可以导出一个形式的`delete`方法:

```java
List<Fruit> deleteByColor(String color);
```

这里，`deleteByColor` 方法删除所有给定颜色的水果，并返回一个被删除记录的列表。

**让我们测试派生的删除方法。**首先，我们将通过定义`test-fruit-data.sql:`中的数据，在`Fruit`表中插入一些记录

```java
insert into fruit(id,name,color) values (1,'apple','red');
insert into fruit(id,name,color) values (2,'custard apple','green');
insert into fruit(id,name,color) values (3,'mango','yellow');
insert into fruit(id,name,color) values (4,'guava','green');
```

然后，我们将删除所有“绿色”水果:

```java
@Transactional
@Test
@Sql(scripts = { "/test-fruit-data.sql" })
public void givenFruits_WhenDeletedByColor_ThenDeletedFruitsShouldReturn() {
     List<Fruit> fruits = fruitRepository.deleteByColor("green");

     assertEquals("number of fruits are not matching", 2, fruits.size());
     fruits.forEach(fruit -> assertEquals("It's not a green fruit", "green", fruit.getColor()));
} 
```

**另外，注意我们需要使用`@Transactional`注释来删除方法。**

接下来，让我们为第二个`deleteBy`方法添加一个类似的测试用例:

```java
@Transactional
@Test
@Sql(scripts = { "/test-fruit-data.sql" })
public void givenFruits_WhenDeletedByName_ThenDeletedFruitCountShouldReturn() {

    Long deletedFruitCount = fruitRepository.deleteByName("apple");

    assertEquals("deleted fruit count is not matching", 1, deletedFruitCount.intValue());
}
```

## 3.派生的`removeBy`方法

**我们也可以使用`removeBy`动词来派生删除方法:**

```java
Long removeByName(String name);
List<Fruit> removeByColor(String color);
```

**注意，这两种方法的行为没有区别。**

最终的`interface`将会是这样的:

```java
@Repository
public interface FruitRepository extends JpaRepository<Fruit, Long> {

    Long deleteByName(String name);

    List<Fruit> deleteByColor(String color);

    Long removeByName(String name);

    List<Fruit> removeByColor(String color);
}
```

让我们为`removeBy`方法添加类似的单元测试:

```java
@Transactional
@Test
@Sql(scripts = { "/test-fruit-data.sql" })
public void givenFruits_WhenRemovedByColor_ThenDeletedFruitsShouldReturn() {
    List<Fruit> fruits = fruitRepository.removeByColor("green");

    assertEquals("number of fruits are not matching", 2, fruits.size());
}
```

```java
@Transactional
@Test
@Sql(scripts = { "/test-fruit-data.sql" })
public void givenFruits_WhenRemovedByName_ThenDeletedFruitCountShouldReturn() {
    Long deletedFruitCount = fruitRepository.removeByName("apple");

    assertEquals("deleted fruit count is not matching", 1, deletedFruitCount.intValue());
}
```

## 4.派生的删除方法 vs `@Query`

我们可能会遇到这样的场景:派生方法的名称太大，或者涉及到不相关实体之间的 SQL 连接。

在这种情况下，我们也可以使用`@Query`和`[@Modifying](/web/20221128041423/https://www.baeldung.com/spring-data-jpa-modifying-annotation)`注释来实现删除操作。

让我们使用一个定制查询来看看我们派生的删除方法的等价代码:

```java
@Modifying
@Query("delete from Fruit f where f.name=:name or f.color=:color")
List<int> deleteFruits(@Param("name") String name, @Param("color") String color);
```

尽管这两种解决方案看起来相似，并且确实达到了相同的结果，但是它们采用的方法略有不同。**`@Query`方法创建一个针对数据库的 JPQL 查询。相比之下，`deleteBy`方法执行一个读查询，然后逐个删除每个条目。**

另外，`deleteBy`方法可以返回已删除记录的列表，而定制查询将返回已删除记录的数量。

## 5.结论

在本文中，我们主要关注派生的 Spring 数据派生的删除方法。本文中使用的完整源代码可以在 GitHub 上找到[。](https://web.archive.org/web/20221128041423/https://github.com/eugenp/tutorials/tree/master/persistence-modules/spring-data-jpa-crud)