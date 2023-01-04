# Spring 数据 JPA 和命名实体图

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-data-jpa-named-entity-graphs>

## 1。概述

**简单来说，[实体图](/web/20221205120254/https://www.baeldung.com/jpa-entity-graph)是 JPA 2.1 中描述查询的另一种方式。**我们可以使用它们来制定性能更好的查询。

在本教程中，我们将通过一个简单的例子来学习如何用 [Spring 数据 JPA](/web/20221205120254/https://www.baeldung.com/the-persistence-layer-with-spring-data-jpa) 实现实体图。

## 2。实体

首先，让我们创建一个名为`Item`的模型，它具有多个特征:

```
@Entity
public class Item {

    @Id
    private Long id;
    private String name;

    @OneToMany(mappedBy = "item")
    private List<Characteristic> characteristics = new ArrayList<>();

    // getters and setters
}
```

现在让我们定义 C `haracteristic`实体:

```
@Entity
public class Characteristic {

    @Id
    private Long id;
    private String type;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn
    private Item item;

    //Getters and Setters
}
```

正如我们在代码中看到的,`Item` 实体中的`characteristics`字段和`Characteristic`实体中的`item`字段都是使用`fetch`参数延迟加载的。因此，**我们的目标是在运行时急切地加载它们。**

## 3。实体图形

在 Spring Data JPA 中，我们可以使用`@NamedEntityGraph` 和`@EntityGraph` 注释的**组合来定义一个实体图。或者，我们也可以只使用`@EntityGraph`注释的 **`attributePaths`参数来定义特定的实体图。****

让我们看看如何能做到这一点。

### 3.1。`@NamedEntityGraph`同

首先，我们可以在我们的`Item` 实体上直接使用 JPA 的`@NamedEntityGraph`注释:

```
@Entity
@NamedEntityGraph(name = "Item.characteristics",
    attributeNodes = @NamedAttributeNode("characteristics")
)
public class Item {
	//...
}
```

然后，我们可以将`@EntityGraph`注释附加到我们的一个存储库方法上:

```
public interface ItemRepository extends JpaRepository<Item, Long> {

    @EntityGraph(value = "Item.characteristics")
    Item findByName(String name);
}
```

如代码所示，我们已经将之前在`Item`实体上创建的实体图名称**、**传递给了`@EntityGraph`注释。当我们调用方法时，这是 Spring 数据将使用的查询。

**`@EntityGraph`标注的类型参数默认值为`EntityGraphType.FETCH`** 。当我们使用它时，Spring 数据模块将在指定的属性节点上应用`FetchType.EAGER`策略。对于其他人，将应用`FetchType.LAZY`策略。

因此在我们的例子中，`characteristics`属性将被急切地加载，即使`@OneToMany`注释的默认获取策略是惰性的。

这里的一个问题是**如果定义的`fetch`策略是`EAGER, `，那么我们不能将其行为更改为** `**LAZY**.`这是设计好的，因为后续操作可能需要在执行过程中的稍后时刻急切获取数据。

### 3.2。没有`@NamedEntityGraph`

**或者，我们也可以用`attributePaths.`定义一个特别的实体图**

让我们添加一个特别的实体图到我们的`CharacteristicsRepository` 中，它急切地加载它的`Item`父节点:

```
public interface CharacteristicsRepository 
  extends JpaRepository<Characteristic, Long> {

    @EntityGraph(attributePaths = {"item"})
    Characteristic findByType(String type);    
}
```

这将急切地加载`Characteristic`、**实体的`item`属性，即使我们的实体为该属性声明了一个延迟加载策略。**

这很方便，因为我们可以内联定义实体图，而不是引用现有的命名实体图。

## 4。测试用例

现在我们已经定义了我们的实体图，让我们创建一个测试用例来验证它:

```
@DataJpaTest
@RunWith(SpringRunner.class)
@Sql(scripts = "/entitygraph-data.sql")
public class EntityGraphIntegrationTest {

    @Autowired
    private ItemRepository itemRepo;

    @Autowired
    private CharacteristicsRepository characteristicsRepo;

    @Test
    public void givenEntityGraph_whenCalled_shouldRetrunDefinedFields() {
        Item item = itemRepo.findByName("Table");
        assertThat(item.getId()).isEqualTo(1L);
    }

    @Test
    public void givenAdhocEntityGraph_whenCalled_shouldRetrunDefinedFields() {
        Characteristic characteristic = characteristicsRepo.findByType("Rigid");
        assertThat(characteristic.getId()).isEqualTo(1L);
    }
}
```

第一个测试将使用通过`@NamedEntityGraph`注释定义的实体图。

让我们看看 Hibernate 生成的 SQL:

```
select 
    item0_.id as id1_10_0_,
    characteri1_.id as id1_4_1_,
    item0_.name as name2_10_0_,
    characteri1_.item_id as item_id3_4_1_,
    characteri1_.type as type2_4_1_,
    characteri1_.item_id as item_id3_4_0__,
    characteri1_.id as id1_4_0__
from 
    item item0_ 
left outer join 
    characteristic characteri1_ 
on 
    item0_.id=characteri1_.item_id 
where 
    item0_.name=?
```

为了比较，让我们从存储库中删除`@EntityGraph`注释并检查查询:

```
select 
    item0_.id as id1_10_,
    item0_.name as name2_10_ 
from 
    item item0_ 
where 
    item0_.name=?
```

从这些查询中，我们可以清楚地观察到，没有`@EntityGraph`注释**生成的查询没有加载`Characteristic`实体的任何属性。**因此，它只加载`Item`实体。

最后，让我们比较一下第二个测试的 Hibernate 查询和`@EntityGraph`注释:

```
select 
    characteri0_.id as id1_4_0_,
    item1_.id as id1_10_1_,
    characteri0_.item_id as item_id3_4_0_,
    characteri0_.type as type2_4_0_,
    item1_.name as name2_10_1_ 
from 
    characteristic characteri0_ 
left outer join 
    item item1_ 
on 
    characteri0_.item_id=item1_.id 
where 
    characteri0_.type=?
```

没有`@EntityGraph`注释的查询:

```
select 
    characteri0_.id as id1_4_,
    characteri0_.item_id as item_id3_4_,
    characteri0_.type as type2_4_ 
from 
    characteristic characteri0_ 
where 
    characteri0_.type=?
```

## 5。结论

在本教程中，我们学习了如何在 Spring 数据中使用 JPA 实体图。有了 Spring 数据，**我们可以创建链接到不同实体图的多个存储库方法**。

这篇文章的例子可以在 GitHub 的[上找到。](https://web.archive.org/web/20221205120254/https://github.com/eugenp/tutorials/tree/master/persistence-modules/spring-data-jpa-query)