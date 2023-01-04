# Hibernate 中的紧急/延迟加载

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/hibernate-lazy-eager-loading>

## 1。概述

使用 ORM 时，数据获取/加载可以分为两种类型:急切型和懒惰型。

在这个快速教程中，我们将指出不同之处，并展示如何在 Hibernate 中使用它们。

## 2。Maven 依赖关系

为了使用 Hibernate，让我们首先定义我们的`pom.xml`中的主要依赖项:

```
<dependency>
    <groupId>org.hibernate</groupId>
    <artifactId>hibernate-core</artifactId>   
    <version>5.2.2.Final</version>
</dependency>
```

Hibernate 的最新版本可以在[这里](https://web.archive.org/web/20221027145013/https://mvnrepository.com/artifact/org.hibernate/hibernate-core)找到。

## 3。急切和懒惰的加载

这里我们要讨论的第一件事是什么是延迟加载和急切加载:

*   **急切加载**是一种现场进行数据初始化的设计模式。
*   **惰性加载**是一种设计模式，我们用它来尽可能推迟对象的初始化。

让我们看看这是如何工作的。

首先，我们来看看`UserLazy`类:

```
@Entity
@Table(name = "USER")
public class UserLazy implements Serializable {

    @Id
    @GeneratedValue
    @Column(name = "USER_ID")
    private Long userId;

    @OneToMany(fetch = FetchType.LAZY, mappedBy = "user")
    private Set<OrderDetail> orderDetail = new HashSet();

    // standard setters and getters
    // also override equals and hashcode

}
```

接下来，我们将看到`OrderDetail`类:

```
@Entity
@Table (name = "USER_ORDER")
public class OrderDetail implements Serializable {

    @Id
    @GeneratedValue
    @Column(name="ORDER_ID")
    private Long orderId;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name="USER_ID")
    private UserLazy user;

    // standard setters and getters
    // also override equals and hashcode

}
```

一个`User`可以有多个`OrderDetails`。**在急切加载策略中，如果我们加载`User`数据，它也会加载与之相关的所有订单，并将其存储在内存中。**

但是当我们启用延迟加载时，如果我们拉起一个`UserLazy`，`OrderDetail`数据不会被初始化并加载到内存中，直到我们对它进行显式调用。

在下一节中，我们将看到如何在 Hibernate 中实现这个例子。

## 4。装载配置

让我们看看如何在 Hibernate 中配置抓取策略。

我们可以通过使用这个注释参数来启用延迟加载:

```
fetch = FetchType.LAZY
```

对于快速获取，我们使用这个参数:

```
fetch = FetchType.EAGER
```

为了设置急切加载，我们使用了`UserLazy`的孪生类`UserEager`。

在下一节中，我们将看看这两种抓取类型之间的区别。

## 5。差异

正如我们所提到的，这两种读取之间的主要区别在于数据载入内存的时间。

让我们来看看:

```
List<UserLazy> users = sessionLazy.createQuery("From UserLazy").list();
UserLazy userLazyLoaded = users.get(3);
return (userLazyLoaded.getOrderDetail());
```

使用惰性初始化方法，`orderDetailSet`只有在我们使用 getter 或其他方法显式调用它时才会被初始化:

```
UserLazy userLazyLoaded = users.get(3);
```

但是在`UserEager`中用一个急切的方法，它会在第一行中立即初始化:

```
List<UserEager> user = sessionEager.createQuery("From UserEager").list();
```

对于延迟加载，我们使用一个代理对象并启动一个单独的 SQL 查询来加载`orderDetailSet`。

在 Hibernate 中，禁用代理或延迟加载的想法被认为是一种不好的做法。这会导致获取和存储大量数据，而不管是否需要。

我们可以使用以下方法来测试功能:

```
Hibernate.isInitialized(orderDetailSet);
```

现在让我们看一下在这两种情况下生成的查询:

```
<property name="show_sql">true</property>
```

上面`fetching.hbm.xml`中的设置显示了生成的 SQL 查询。如果我们查看控制台输出，我们可以看到生成的查询。

对于延迟加载，下面是为加载`User`数据而生成的查询:

```
select user0_.USER_ID as USER_ID1_0_,  ... from USER user0_
```

然而，在急切加载中，我们看到了用`USER_ORDER`进行的连接:

```
select orderdetai0_.USER_ID as USER_ID4_0_0_, orderdetai0_.ORDER_ID as ORDER_ID1_1_0_, orderdetai0_ ...
  from USER_ORDER orderdetai0_ where orderdetai0_.USER_ID=?
```

上面的查询是为所有的`Users`生成的，这会导致比其他方法多得多的内存使用。

## 6。优缺点

### 6.1。惰性加载

优势:

*   初始加载时间比另一种方法短得多
*   比另一种方法消耗更少的内存

缺点:

*   延迟初始化可能会在不必要的时候影响性能。
*   在某些情况下，我们需要特别小心地处理延迟初始化的对象，否则我们可能会以异常结束。

### 6.2。急切加载

优势:

*   没有延迟初始化相关的性能影响

缺点:

*   初始装载时间长
*   加载太多不必要的数据可能会影响性能

## 7。休眠中的延迟加载

通过提供类的代理实现，Hibernate 在实体和关联上应用了惰性加载方法。

Hibernate 通过用一个从实体的类派生的代理来替代实体来拦截对实体的调用。在我们的例子中，在控制权交给`User`类实现之前，缺失的请求信息将从数据库中加载。

我们还应该注意，当关联被表示为一个集合类时(在上面的例子中，它被表示为`Set<OrderDetail> orderDetailSet`)，一个包装器被创建并替换为一个原始集合。

要了解更多关于代理设计模式的信息，请参考[这里的](https://web.archive.org/web/20221027145013/https://docs.oracle.com/javase/8/docs/technotes/guides/reflection/proxy.html)。

## 8。结论

在本文中，我们展示了 Hibernate 中使用的两种主要抓取类型的例子。

要想获得更高级的专业知识，可以查看 Hibernate 的官方网站。

要获得本文中讨论的代码，请查看这个[存储库](https://web.archive.org/web/20221027145013/https://github.com/eugenp/tutorials/tree/master/persistence-modules/spring-data-jpa-query-2)。