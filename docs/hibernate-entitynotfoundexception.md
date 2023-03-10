# Hibernate 中的 EntityNotFoundException

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/hibernate-entitynotfoundexception>

## 1.介绍

在本教程中，我们将讨论来自`javax.persistence` 包的`EntityNotFoundException` 。我们将讨论这种异常可能发生的情况，之后，我们将为这些情况编写测试。

## 2.`EntityNotFoundException`是什么时候抛出的？

针对此异常的 Oracle 文档定义了持久性提供者可以抛出 [`EntityNotFoundException`](https://web.archive.org/web/20220625072049/https://docs.oracle.com/javaee/7/api/javax/persistence/EntityNotFoundException.html) 的三种情况:

*   不存在的实体上
*   `EntityManager.refresh` 数据库中不存在的对象
*   对数据库中不存在的实体进行悲观锁定

除了这三个用例，还有一个更模糊的用例。**当使用** **`@ManyToOne` 关系和懒惰加载**时，也会出现这种异常。

当我们使用`@ManyToOne` 注释时，那么被引用的实体必须存在。这通常通过使用外键来确保数据库的完整性。如果在我们的关系模型中不使用外键或者我们的数据库不一致，我们可以在获取实体时看到`EntityNotFoundException` 。我们将在下一节用一个例子来说明这一点。

## 3.`EntityNotFoundException`在实践中

首先，让我们来看一个更简单的用例。在上一节中，我们提到了 [`getReference`方法](/web/20220625072049/https://www.baeldung.com/jpa-entity-manager-get-reference)。我们使用这个方法来获取一个特定实体的代理。该代理仅初始化了主键字段。当我们在这个代理实体上调用一个 getter 时，持久性提供者初始化其余的字段。如果数据库中不存在该实体，那么我们得到`EntityNotFoundException`:

```java
@Test(expected = EntityNotFoundException.class)
public void givenNonExistingUserId_whenGetReferenceIsUsed_thenExceptionIsThrown() {
    User user = entityManager.getReference(User.class, 1L);
    user.getName();
} 
```

`User`实体是初等的。它只有两个字段，没有关系。我们在测试的第一行创建了一个主键值为 1L 的代理实体。之后，我们在代理实体上调用 getter。持久性提供者试图通过主键获取实体，因为记录不存在，所以抛出了一个`EntityNotFoundException`。

对于下一个例子，我们将使用不同的域实体。我们将创建`Item`和`Category` 实体，它们之间具有双向关系:

```java
@Entity
public class Item implements Serializable {
    @Id
    @Column(unique = true, nullable = false)
    private long id;
    private String name;
    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "category_id", foreignKey = @ForeignKey(ConstraintMode.NO_CONSTRAINT))
    private Category category;
    // getters and setters
}
@Entity
public class Category implements Serializable {
    @Id
    @Column(unique = true, nullable = false)
    private long id;
    private String name;
    @OneToMany
    @JoinColumn(name = "category_id", foreignKey = @ForeignKey(ConstraintMode.NO_CONSTRAINT))
    private List<Item> items = new ArrayList<>();
    // getters and setters
}
```

请注意，我们对`@ManyToOne`注释使用了延迟获取。同样，我们使用`@ForeignKey` 从数据库中删除约束:

```java
@Test(expected = EntityNotFoundException.class)
public void givenItem_whenManyToOneEntityIsMissing_thenExceptionIsThrown() {
    entityManager.createNativeQuery("Insert into Item (category_id, name, id) values (1, 'test', 1)").executeUpdate();
    entityManager.flush();
    Item item = entityManager.find(Item.class, 1L);
    item.getCategory().getName();
}
```

在这个测试中，我们通过 id 获取项目实体。方法`find`将返回一个没有完全初始化`Category`的`Item`对象，因为我们使用了`FetchType.LAZY` (只设置了`id`，类似于前面的例子)。当我们在`Category`上调用 getter 时，对象持久化提供者将试图从数据库中获取对象，我们将得到一个异常，因为记录不存在。

`@ManyToOne`关系假设被引用的实体存在。外键和数据库完整性确保了这些实体的存在。如果不是这种情况，有一种方法可以忽略缺失的实体。

**结合`@NotFound(action = NotFoundAction.IGNORE)` 和`@ManyToOne` 注释将阻止持久性提供者抛出`EntityNotFoundException`、**，但是我们必须手工处理丢失的实体以避免`NullPointerException`。

## 4.结论

在本文中，我们讨论了在哪些情况下会发生`EntityNotFoundException`以及我们如何处理它。首先，我们回顾了官方文档并涵盖了常见的用例。之后，我们将介绍更复杂的情况以及如何解决这个问题。

像往常一样，我们可以在 GitHub 上找到这篇文章[中的代码。](https://web.archive.org/web/20220625072049/https://github.com/eugenp/tutorials/tree/master/persistence-modules/hibernate-exceptions)