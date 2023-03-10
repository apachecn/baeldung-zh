# @ dynamic update with Spring Data JPA

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-data-jpa-dynamicupdate>

## 1。概述

当我们在 Hibernate 中使用 Spring Data JPA 时，我们也可以使用 Hibernate 的附加特性。`@DynamicUpdate`就是这样一个特征。

`@DynamicUpdate`是可以应用于 JPA 实体的类级注释。**它确保 Hibernate 只使用它为更新实体**而生成的 SQL 语句中修改过的列。

在本文中，我们将借助一个 **[Spring Data JPA](/web/20221206061107/https://www.baeldung.com/the-persistence-layer-with-spring-data-jpa)** 的例子来看看`@DynamicUpdate`注释。

## 2。JPA `@Entity`

当应用程序启动时，Hibernate 为所有实体的 CRUD 操作生成 SQL 语句。这些 SQL 语句只生成一次，并缓存在内存中，以提高性能。

生成的 SQL update 语句包括一个实体的所有列。如果我们更新一个实体，被修改的列的值被传递给 SQL update 语句。对于没有更新的列，Hibernate 使用它们现有的值进行更新。

我们试着用一个例子来理解这一点。首先，让我们考虑一个名为`Account`的 JPA 实体:

```java
@Entity
public class Account {

    @Id
    private int id;

    @Column
    private String name;

    @Column
    private String type;

    @Column
    private boolean active;

    // Getters and Setters
}
```

接下来，让我们为`Account`实体编写一个 JPA 存储库:

```java
@Repository
public interface AccountRepository extends JpaRepository<Account, Integer> {
}
```

现在，我们将使用`AccountRepository`来更新一个`Account`对象的`name`字段:

```java
Account account = accountRepository.findOne(ACCOUNT_ID);
account.setName("Test Account");
accountRepository.save(account);
```

执行这个更新后，我们可以验证生成的 SQL 语句。生成的 SQL 语句将包括`Account`的所有列:

```java
update Account set active=?, name=?, type=? where id=?
```

## 3.JPA `@Entity`带`@DynamicUpdate`

我们已经看到，尽管我们只修改了`name`字段，Hibernate 已经在 SQL 语句中包含了所有的列。

现在，让我们将`@DynamicUpdate`注释添加到`Account`实体中:

```java
@Entity
@DynamicUpdate
public class Account {
    // Existing data and methods
}
```

接下来，让我们运行在上一节中使用的相同的更新代码。我们可以看到 Hibernate 生成的 SQL，在这种情况下，只包含了`name`列:

```java
update Account set name=? where id=?
```

那么，**当我们在实体**上使用`@DynamicUpdate`时会发生什么呢？

实际上，当我们在实体上使用`@DynamicUpdate` 时，Hibernate 不使用缓存的 SQL 语句进行更新。相反，它会在我们每次更新实体时生成一条 SQL 语句。这个**生成的 SQL 只包含更改过的列**。

为了找出改变的列，Hibernate 需要跟踪当前实体的状态。因此，当我们更改实体的任何字段时，它会比较实体的当前状态和修改后的状态。

这意味着 **@ `DynamicUpdate`有一个与之相关的性能开销**。因此，我们应该只在真正需要的时候使用它。

当然，在一些情况下我们应该使用这种注释——例如，如果一个实体表示一个有大量列的表，而这些列中只有少数需要经常更新。同样，当我们使用无版本乐观锁定时，我们需要使用`@DynamicUpdate`。

## 4.结论

在本教程中，我们研究了 Hibernate 的`@DynamicUpdate`注释。我们已经使用了一个 Spring 数据 JPA 的例子来看看`@DynamicUpdate`的运行。此外，我们已经讨论了什么时候应该使用这个特性，什么时候不应该使用。

和往常一样，本教程中使用的完整代码示例可以在 Github 上的[处获得。](https://web.archive.org/web/20221206061107/https://github.com/eugenp/tutorials/tree/master/persistence-modules/spring-hibernate-5)