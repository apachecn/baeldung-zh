# JPA CascadeType。移除 vs orphan 移除

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/jpa-cascade-remove-vs-orphanremoval>

## 1.概观

在本教程中，我们将讨论在使用 [JPA](/web/20220625232931/https://www.baeldung.com/the-persistence-layer-with-spring-and-jpa) 时从数据库中删除实体的两个选项之间的区别。

首先，我们将从`CascadeType.REMOVE`开始，这是当父实体的删除发生时删除一个或多个子实体的**方法。然后我们来看看在 JPA 2.0 中引入的`orphanRemoval`属性。这为我们提供了一种从数据库**中**删除孤立实体的方法。**

在整个教程中，我们将使用一个简单的在线商店领域来演示我们的例子。

## 2.领域模型

如前所述，本文使用了一个简单的在线商店领域。其中 `OrderRequest` 有一个 `ShipmentInfo` 和一个 `LineItem` 的列表。

鉴于此，我们来考虑:

*   对于 `ShipmentInfo,` 的删除当发生 `OrderRequest` 的删除时，我们将使用 `CascadeType.REMOVE`
*   为了从 `OrderRequest` 中删除一个 `LineItem` ，我们将使用 `orphanRemoval`

首先，让我们创建一个`ShipmentInfo`  实体:

```java
@Entity
public class ShipmentInfo {

    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    private Long id;

    private String name;

    // constructors
}
```

接下来，让我们创建一个 `LineItem` 实体:

```java
@Entity
public class LineItem {

    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    private Long id;

    private String name;

    @ManyToOne
    private OrderRequest orderRequest;

    // constructors, equals, hashCode
}
```

最后，让我们通过创建一个 *OrderRequest* 实体将所有这些放在一起:

```java
@Entity
public class OrderRequest {

    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    private Long id;

    @OneToOne(cascade = { CascadeType.REMOVE, CascadeType.PERSIST })
    private ShipmentInfo shipmentInfo;

    @OneToMany(orphanRemoval = true, cascade = CascadeType.PERSIST, mappedBy = "orderRequest")
    private List<LineItem> lineItems;

    // constructors

    public void removeLineItem(LineItem lineItem) {
        lineItems.remove(lineItem);
    }
}
```

值得强调一下`removeLineItem`方法，它将*行项目*从*订单请求*中分离出来。

## 3.`CascadeType.REMOVE`

如前所述，用`CascadeType.REMOVE`标记一个引用字段是一种方法，每当父实体被删除时，**删除一个** **子实体或多个子实体** **。**

在我们的例子中，一个`OrderRequest`有一个`ShipmentInfo`，它有一个`CascadeType.REMOVE`。

当删除一个`OrderRequest`时，为了验证从数据库中删除 `ShipmentInfo` ，让我们创建一个简单的集成测试:

```java
@Test
public void whenOrderRequestIsDeleted_thenDeleteShipmentInfo() {
    createOrderRequestWithShipmentInfo();

    OrderRequest orderRequest = entityManager.find(OrderRequest.class, 1L);

    entityManager.getTransaction().begin();
    entityManager.remove(orderRequest);
    entityManager.getTransaction().commit();

    Assert.assertEquals(0, findAllOrderRequest().size());
    Assert.assertEquals(0, findAllShipmentInfo().size());
}

private void createOrderRequestWithShipmentInfo() {
    ShipmentInfo shipmentInfo = new ShipmentInfo("name");
    OrderRequest orderRequest = new OrderRequest(shipmentInfo);

    entityManager.getTransaction().begin();
    entityManager.persist(orderRequest);
    entityManager.getTransaction().commit();

    Assert.assertEquals(1, findAllOrderRequest().size());
    Assert.assertEquals(1, findAllShipmentInfo().size());
}
```

从断言中，我们可以看到删除`OrderRequest`导致相关的`ShipmentInfo`也被成功删除。

## 4.`orphanRemoval`

如前所述，其用法是**从数据库**中删除 **孤立实体** **。** 不再附属于其父实体的实体被定义为孤儿。

在我们的例子中，一个`OrderRequest`有一个`LineItem` 对象的集合，其中 我们使用 `@OneToMany` 注释来标识关系`.` 这也是我们将`orphanRemoval` 属性设置为`true`的地方。为了从`OrderRequest`中分离出`LineItem`，我们可以使用我们之前创建的`removeLineItem`方法。

一切就绪后，一旦我们使用了`removeLineItem`方法并保存了`OrderRequest`，孤立的`LineItem` 就会从数据库中删除。

为了验证从数据库中删除孤立的 `LineItem` ，让我们创建另一个集成测试:

```java
@Test
public void whenLineItemIsRemovedFromOrderRequest_thenDeleteOrphanedLineItem() {
    createOrderRequestWithLineItems();

    OrderRequest orderRequest = entityManager.find(OrderRequest.class, 1L);
    LineItem lineItem = entityManager.find(LineItem.class, 2L);
    orderRequest.removeLineItem(lineItem);

    entityManager.getTransaction().begin();
    entityManager.merge(orderRequest);
    entityManager.getTransaction().commit();

    Assert.assertEquals(1, findAllOrderRequest().size());
    Assert.assertEquals(2, findAllLineItem().size());
}

private void createOrderRequestWithLineItems() {
    List<LineItem> lineItems = new ArrayList<>();
    lineItems.add(new LineItem("line item 1"));
    lineItems.add(new LineItem("line item 2"));
    lineItems.add(new LineItem("line item 3"));

    OrderRequest orderRequest = new OrderRequest(lineItems);

    entityManager.getTransaction().begin();
    entityManager.persist(orderRequest);
    entityManager.getTransaction().commit();

    Assert.assertEquals(1, findAllOrderRequest().size());
    Assert.assertEquals(3, findAllLineItem().size());
}
```

同样，从断言中可以看出，我们已经成功地从数据库中删除了孤立的`LineItem`。

另外，值得一提的是，`removeLineItem `方法修改了`LineItem`的列表，而不是给它重新赋值。做后者会导致一个`PersistenceException`。

为了验证所陈述的行为，让我们创建一个最终的集成测试:

```java
@Test(expected = PersistenceException.class)
public void whenLineItemsIsReassigned_thenThrowAnException() {
    createOrderRequestWithLineItems();

    OrderRequest orderRequest = entityManager.find(OrderRequest.class, 1L);
    orderRequest.setLineItems(new ArrayList<>());

    entityManager.getTransaction().begin();
    entityManager.merge(orderRequest);
    entityManager.getTransaction().commit();
}
```

## 5.结论

在本文中，我们使用一个简单的在线商店域探索了`CascadeType.REMOVE`和`orphanRemoval`之间的区别。此外，为了验证实体是否从数据库中正确删除，我们创建了几个集成测试。

和往常一样，GitHub 上的[提供了这篇文章的完整源代码。](https://web.archive.org/web/20220625232931/https://github.com/eugenp/tutorials/tree/master/persistence-modules/java-jpa-3)