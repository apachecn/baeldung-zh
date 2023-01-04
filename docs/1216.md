# 休眠模式下的 FetchMode

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/hibernate-fetchmode>

## 1.介绍

在这个简短的教程中，我们将看看可以在`@` `org.hibernate.annotations.Fetch`注释中使用的不同的`FetchMode`值。

## 2.树立榜样

例如，我们将使用下面的`Customer`实体，它只有两个属性——一个 id 和一组订单:

```
@Entity
public class Customer {

    @Id
    @GeneratedValue
    private Long id;

    @OneToMany(mappedBy = "customer")
    @Fetch(value = FetchMode.SELECT)
    private Set<Order> orders = new HashSet<>();

    // getters and setters
}
```

此外，我们将创建一个由 id、名称和对`Customer`的引用组成的`Order`实体。

```
@Entity
public class Order {

    @Id
    @GeneratedValue
    private Long id;

    private String name;

    @ManyToOne
    @JoinColumn(name = "customer_id")
    private Customer customer;

    // getters and setters
}
```

在接下来的每一节中，我们将从数据库中获取客户并获得其所有订单:

```
Customer customer = customerRepository.findById(id).get();
Set<Order> orders = customer.getOrders();
```

## 3.`FetchMode.SELECT`

在我们的`Customer` 实体上，我们已经用`@Fetch` 注释对`orders` 属性进行了注释:

```
@OneToMany
@Fetch(FetchMode.SELECT)
private Set<Orders> orders;
```

我们使用`@Fetch`来描述当我们查找`Customer.`时 Hibernate 应该如何检索属性

使用`SELECT`表示属性应该延迟加载。

这意味着对于第一行:

```
Customer customer = customerRepository.findById(id).get();
```

我们不会看到与订单表的连接:

```
Hibernate: 
    select ...from customer
    where customer0_.id=? 
```

下一行是:

```
Set<Order> orders = customer.getOrders();
```

我们将看到对相关订单的后续查询:

```
Hibernate: 
    select ...from order
    where order0_.customer_id=? 
```

`Hibernate FetchMode.SELECT`为每个需要加载的`Order`生成一个单独的查询。

在我们的示例中，给出了一个加载客户的查询和五个加载订单集合的附加查询。

**这就是所谓的`n` + 1 选择问题。**执行一个查询将触发`n`个附加查询。

### 3.1.@ `BatchSize`

`FetchMode.SELECT`有一个使用`@BatchSize` 注释的可选配置注释:

```
@OneToMany
@Fetch(FetchMode.SELECT)
@BatchSize(size=10)
private Set<Orders> orders;
```

`Hibernate`将尝试批量加载由 `size`参数定义的订单集合。

在我们的例子中，我们只有五个订单，所以一个查询就足够了。

我们仍将使用相同的查询:

```
Hibernate:
    select ...from order
    where order0_.customer_id=?
```

**但是只会运行一次。**现在我们只有两个查询:一个加载`Customer`和一个加载订单集合。

## 4.`FetchMode.JOIN`

当`FetchMode.SELECT` 懒散地加载关系时，`FetchMode.JOIN`急切地加载它们，比如通过一个连接:

```
@OneToMany
@Fetch(FetchMode.JOIN)
private Set<Orders> orders;
```

这导致只对`Customer`和它们的`Order`进行一次查询:

```
Hibernate: 
    select ...
    from
        customer customer0_ 
    left outer join
        order order1 
            on customer.id=order.customer_id 
    where
        customer.id=?
```

## 5.`FetchMode.SUBSELECT`

因为`orders`属性是一个集合，我们也可以使用`FetchMode.SUBSELECT`:

```
@OneToMany
@Fetch(FetchMode.SUBSELECT)
private Set<Orders> orders;
```

我们只能对集合使用`SUBSELECT` 。

有了这个设置，我们回到对`Customer:`的一个查询

```
Hibernate: 
    select ...
    from customer customer0_ 
```

和一个针对`Order`的查询，这次使用子选择:

```
Hibernate: 
    select ...
    from
        order order0_ 
    where
        order0_.customer_id in (
            select
                customer0_.id 
            from
                customer customer0_
        )
```

## 6.`FetchMode`对`FetchType`

一般来说，`FetchMode`定义了`Hibernate`将如何获取数据(通过 select、join 或 subselect)。另一方面，`FetchType`定义了 Hibernate 是急切地还是懒洋洋地加载数据。

这两者之间的确切规则如下:

*   如果代码没有设置`FetchMode`，默认为`JOIN`，`FetchType`按照
    T5 的定义工作
*   设置`FetchMode.SELECT`或`FetchMode.SUBSELECT` 后，`FetchType`也按照的定义工作
*   设置了`FetchMode.JOIN` 后，`FetchType`将被忽略，查询将一直处于等待状态

欲了解更多信息，请参考[休眠中的急切/迟缓加载](/web/20220926191606/https://www.baeldung.com/hibernate-lazy-eager-loading)。

## 7.结论

在本教程中，我们已经了解了`FetchMode`的不同值以及它们与`FetchType`的关系。

像往常一样，所有的源代码都可以在 GitHub 上获得[。](https://web.archive.org/web/20220926191606/https://github.com/eugenp/tutorials/tree/master/persistence-modules/hibernate-mapping)