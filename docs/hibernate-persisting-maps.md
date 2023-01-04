# 用 Hibernate 保存地图

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/hibernate-persisting-maps>

## 1.介绍

在 Hibernate 中，我们可以通过让我们的一个字段成为`List`来表示 Java beans 中的[一对多关系](/web/20220908191632/https://www.baeldung.com/hibernate-one-to-many)。

在这个快速教程中，我们将探索用`Map`代替的各种方法。

## 2.`Map` s 不同于`List` s

使用一个`Map` 来表示一对多关系不同于一个`List`,因为我们有一个键。

这个键将我们的实体关系变成了一个`ternary`关联，其中每个键都指向一个简单的值或者一个可嵌入的对象或者一个实体。因此，要使用`Map`，我们总是需要**一个连接表来存储引用父实体的外键——键和值。**

但是这个连接表与其他连接表有些不同，因为主键不一定是父表和目标表的外键。取而代之的是，我们将主键作为父项的外键和作为我们`Map.`的键的列的组合

`Map`中的键值对可能有两种类型:[值类型和](https://web.archive.org/web/20220908191632/https://javabydeveloper.com/hibernate-entity-types-vs-value-types/)实体类型。在接下来的几节中，我们将看看在 Hibernate 中表示这些关联的方法。

## 3.使用`@MapKeyColumn`

假设我们有一个`Order`实体，我们想要跟踪订单中所有商品的名称和价格。因此，**我们想引入一个`Map<String, Double>` 到`Order`，它将把商品的名称映射到它的价格:**

```java
@Entity
@Table(name = "orders")
public class Order {
    @Id
    @GeneratedValue
    @Column(name = "id")
    private int id;

    @ElementCollection
    @CollectionTable(name = "order_item_mapping", 
      joinColumns = {@JoinColumn(name = "order_id", referencedColumnName = "id")})
    @MapKeyColumn(name = "item_name")
    @Column(name = "price")
    private Map<String, Double> itemPriceMap;

    // standard getters and setters
}
```

**我们需要指示 Hibernate 从哪里获取键和值。**对于键，**我们用@ `MapKey`** `**Column**,` 表示`Map`的键是我们的连接表`order_item_mapping`的`item_name`列。类似地，`@Column `指定`Map's `值对应于连接表的`price `列。

另外，`itemPriceMap `对象是值类型映射，因此我们必须使用`@ElementCollection `注释。

除了基本值类型对象， [@ `Embeddable`](https://web.archive.org/web/20220908191632/https://docs.jboss.org/hibernate/jpa/2.1/api/javax/persistence/Embeddable.html) 对象也可以以类似的方式用作`Map`的值。

## 4.使用`@MapKey`

众所周知，需求会随着时间而变化——所以，现在，假设我们需要存储更多的`Item `属性以及`itemName `和`itemPrice`:

```java
@Entity
@Table(name = "item")
public class Item {
    @Id
    @GeneratedValue
    @Column(name = "id")
    private int id;

    @Column(name = "name")
    private String itemName;

    @Column(name = "price")
    private double itemPrice;

    @Column(name = "item_type")
    @Enumerated(EnumType.STRING)
    private ItemType itemType;

    @Temporal(TemporalType.TIMESTAMP)
    @Column(name = "created_on")
    private Date createdOn;

    // standard getters and setters
}
```

相应地，让我们将`Order `实体类中的`Map<String, Double>`改为`Map<String, Item> `:

```java
@Entity
@Table(name = "orders")
public class Order {
    @Id
    @GeneratedValue
    @Column(name = "id")
    private int id;

    @OneToMany(cascade = CascadeType.ALL)
    @JoinTable(name = "order_item_mapping", 
      joinColumns = {@JoinColumn(name = "order_id", referencedColumnName = "id")},
      inverseJoinColumns = {@JoinColumn(name = "item_id", referencedColumnName = "id")})
    @MapKey(name = "itemName")
    private Map<String, Item> itemMap;

}
```

注意，这次我们将使用`@MapKey` 注释，这样 Hibernate 将使用`Item#` `itemName`作为映射键列，而不是在连接表中引入一个额外的列。因此，在本例中，**连接表** `**order_item_mapping** ` **没有键列**——而是引用了`I` `tem`的名称。

这与`@MapKeyColumn. `相反，当**使用`@MapKeyColumn, `时，映射键驻留在连接表**中。这就是为什么**我们不能同时使用两个注释来定义实体映射的原因。**

另外，`itemMap `是一个实体类型的地图，因此我们必须使用 [`@OneToMany`](/web/20220908191632/https://www.baeldung.com/hibernate-one-to-many) 或`[@ManyToMany](/web/20220908191632/https://www.baeldung.com/hibernate-many-to-many)`来标注关系。

## 5.使用`@MapKeyEnumerated`和`@MapKeyTemporal`

每当我们指定一个 enum 作为`Map`键时，我们使用`@MapKeyEnumerated`。类似地，对于时态值，使用`@MapKeyTemporal`。其行为分别与标准的 [`@Enumerated`](https://web.archive.org/web/20220908191632/https://docs.jboss.org/hibernate/jpa/2.1/api/javax/persistence/Enumerated.html) 和 [`@Temporal`](https://web.archive.org/web/20220908191632/https://docs.jboss.org/hibernate/jpa/2.1/api/javax/persistence/Temporal.html) 注释十分相似。

默认情况下，这些与`@MapKeyColumn`类似，因为**将在连接表中创建一个键列。**如果我们想要重用已经存储在持久化实体中的值，我们应该另外用`@MapKey`标记该字段。

## 6.使用`@MapKeyJoinColumn`

接下来，假设我们还需要跟踪每件商品的卖家。我们可以这样做的一个方法是添加一个`Seller` 实体，并将它绑定到我们的`Item` 实体:

```java
@Entity
@Table(name = "seller")
public class Seller {

    @Id
    @GeneratedValue
    @Column(name = "id")
    private int id;

    @Column(name = "name")
    private String sellerName;

    // standard getters and setters

}
```

```java
@Entity
@Table(name = "item")
public class Item {
    @Id
    @GeneratedValue
    @Column(name = "id")
    private int id;

    @Column(name = "name")
    private String itemName;

    @Column(name = "price")
    private double itemPrice;

    @Column(name = "item_type")
    @Enumerated(EnumType.STRING)
    private ItemType itemType;

    @Temporal(TemporalType.TIMESTAMP)
    @Column(name = "created_on")
    private Date createdOn;

    @ManyToOne(cascade = CascadeType.ALL)
    @JoinColumn(name = "seller_id")
    private Seller seller;

    // standard getters and setters
}
```

在这种情况下，让我们假设我们的用例是按照`Seller. `将所有`Order`的`Item`分组，因此，让我们将`Map<String, Item>` 改为`Map<Seller, Item>`:

```java
@Entity
@Table(name = "orders")
public class Order {
    @Id
    @GeneratedValue
    @Column(name = "id")
    private int id;

    @OneToMany(cascade = CascadeType.ALL)
    @JoinTable(name = "order_item_mapping", 
      joinColumns = {@JoinColumn(name = "order_id", referencedColumnName = "id")},
      inverseJoinColumns = {@JoinColumn(name = "item_id", referencedColumnName = "id")})
    @MapKeyJoinColumn(name = "seller_id")
    private Map<Seller, Item> sellerItemMap;

    // standard getters and setters

}
```

我们需要添加`@MapKeyJoinColumn` 来实现这一点，因为该注释允许 Hibernate 将`seller_id `列(映射键)与`item_id `列一起保存在连接表`order_item_mapping `中。因此，在从数据库中读取数据时，我们可以轻松地执行一个`GROUP BY`操作。

## 7.结论

在本文中，我们了解了几种根据所需映射在 Hibernate 中持久化`Map`的方法。

一如既往，本教程的源代码可以在 Github 上找到[。](https://web.archive.org/web/20220908191632/https://github.com/eugenp/tutorials/tree/master/persistence-modules/hibernate-mapping)