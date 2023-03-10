# 用 JPA 返回自动生成的 Id

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/jpa-get-auto-generated-id>

## 1.介绍

在本教程中，我们将讨论如何用 JPA 处理自动生成的 id。在我们看一个实际的例子之前，有两个关键的概念我们必须理解，即生命周期和 id 生成策略。

## 2.实体生命周期和 Id 生成

每个实体在其生命周期中都有四种可能的状态。这些状态是 **`new`、`managed`、`detached`和`removed`** 。我们的焦点将集中在`new`和`managed`州。**创建对象时，实体处于`new`状态**。因此，`EntityManager` 不知道这个对象。调用`EntityManager`上的`persist`方法，对象从`new`状态转换到`managed`状态。此方法需要一个活动的事务。

JPA 为 id 生成定义了四种策略。我们可以将这四种策略分为两类:

*   id 是预先分配的，在`commit`之前`EntityManager` 可以使用
*   id 在交易后分配`commit`

关于每个 id 生成策略的更多细节，请参考我们的文章，[JPA 何时设置主键](/web/20220926194434/https://www.baeldung.com/jpa-strategies-when-set-primary-key)。

## 3.问题陈述

返回一个对象的 id 可能会成为一个麻烦的任务。我们需要理解前面提到的原则来避免问题。**根据 JPA 配置，服务可能返回 id 等于零(或 null)的对象**。重点将放在服务类实现上，以及不同的修改如何为我们提供解决方案。

我们将使用 JPA 规范创建一个 Maven 模块，并将 Hibernate 作为它的实现。为了简单起见，我们将使用 H2 内存数据库。

让我们从创建一个域实体并将其映射到一个数据库表开始。对于这个例子，我们将创建一个具有一些基本属性的`User`实体:

```java
@Entity
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private long id;
    private String username;
    private String password;

    //...
}
```

在 domain 类之后，我们将创建`a UserService` 类。这个简单的服务将有一个对`EntityManager`的引用和一个将`User`对象保存到数据库的方法:

```java
public class UserService {
    EntityManager entityManager;

    public UserService(EntityManager entityManager) {
        this.entityManager = entityManager;
    }

    @Transactional
    public long saveUser(User user){
        entityManager.persist(user);
        return user.getId();
    }
}
```

这种设置是我们之前提到过的常见缺陷。我们可以通过测试证明`saveUser`方法的返回值是零:

```java
@Test
public void whenNewUserIsPersisted_thenEntityHasNoId() {
    User user = new User();
    user.setUsername("test");
    user.setPassword(UUID.randomUUID().toString());

    long index = service.saveUser(user);

    Assert.assertEquals(0L, index);
}
```

在接下来的部分中，我们将回过头来理解为什么会发生这种情况，以及我们如何解决它。

## 4.人工交易控制

在对象创建之后，我们的`User`实体处于`new`状态。在`saveUser`方法中的`persist`方法调用之后，实体状态变为`managed`。我们从 recap 部分记得，在事务`commit` 之后**托管对象获得一个 id。由于 `saveUser`方法仍在运行，由 [`@Transactional`](/web/20220926194434/https://www.baeldung.com/transaction-configuration-with-jpa-and-spring) 注释创建的事务还没有提交。当`saveUser`完成执行时，我们的托管实体获得一个 id。**

一个可能的解决方案是**手动调用`EntityManager`上的`flush`方法**。另一方面，我们可以**手动控制事务**并保证我们的方法正确返回 id。我们可以用`EntityManager`做到这一点:

```java
@Test
public void whenTransactionIsControlled_thenEntityHasId() {
    User user = new User();
    user.setUsername("test");
    user.setPassword(UUID.randomUUID().toString());

    entityManager.getTransaction().begin();
    long index = service.saveUser(user);
    entityManager.getTransaction().commit();

    Assert.assertEquals(2L, index);
}
```

## 5.使用 Id 生成策略

到目前为止，我们使用的是第二类，其中 id 分配发生在事务`commit`之后。**预分配策略可以在交易`commit,`之前为我们提供 id，因为它们在内存**中保存了少量 id。这个选项并不总是能够实现，因为并不是所有的数据库引擎都支持所有的生成策略。将战略改为`GenerationType.SEQUENCE` 可以解决我们的问题。该策略使用数据库序列，而不是像`GenerationType.IDENTITY.`中那样使用自动递增的列

为了改变策略，我们编辑我们的域实体类:

```java
@Entity
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.SEQUENCE)
    private long id;

    //...
}
```

## 6.结论

在本文中，我们介绍了 JPA 中的 id 生成技术。首先，我们简要回顾了 id 生成的最重要的关键方面。然后我们讨论了 JPA 中使用的常见配置，以及它们的优缺点。本文引用的所有代码都可以在 GitHub 上找到[。](https://web.archive.org/web/20220926194434/https://github.com/eugenp/tutorials/tree/master/persistence-modules/java-jpa-3)