# JPA 中的乐观锁定

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/jpa-optimistic-locking>

## 1.概观

对于企业应用程序，正确管理对数据库的并发访问是至关重要的。这意味着我们应该能够以有效的、最重要的是防错的方式处理多个事务。

此外，我们需要确保并发读取和更新之间的数据保持一致。

为了实现这一点，我们可以使用 Java 持久性 API 提供的乐观锁定机制。这样，同时对相同数据进行的多次更新就不会相互干扰。

## 2.理解乐观锁定

为了使用乐观锁定，**我们需要一个包含带有`@Version`注释的属性的实体。**在使用它时，每个读取数据的事务都保存 version 属性的值。

在事务想要进行更新之前，它会再次检查 version 属性。

如果值在此期间发生了变化，就会抛出一个`OptimisticLockException`。否则，事务提交更新并递增值版本属性。

## 3.悲观锁定与乐观锁定

与乐观锁定相反，JPA 给了我们悲观锁定。这是处理数据并发访问的另一种机制。

我们在以前的一篇文章中讨论过悲观锁定—[JPA 中的悲观锁定](/web/20221123070554/https://www.baeldung.com/jpa-pessimistic-locking)。让我们找出它们之间的区别，以及我们如何从每种类型的锁定中获益。

正如我们之前所说的，**乐观锁定是基于通过检查实体的版本属性来检测实体的变化。**如果任何并发更新发生，`OptmisticLockException`发生。之后，我们可以重试更新数据。

我们可以想象，这种机制适用于读操作比更新或删除操作多得多的应用程序。在实体必须分离一段时间并且不能持有锁的情况下，它也很有用。

相反，悲观锁定机制包括在数据库级别锁定实体。

每个事务都可以获得一个数据锁。只要事务持有锁，它就不能读取、删除或更新锁定的数据。

我们可以假设使用悲观锁可能会导致死锁。但是，它比乐观锁定确保了更高的数据完整性。

## 4.版本属性

版本属性是带有`@Version`注释的属性。**它们是启用乐观锁定所必需的。**

让我们来看一个示例实体类:

```
@Entity
public class Student {

    @Id
    private Long id;

    private String name;

    private String lastName;

    @Version
    private Integer version;

    // getters and setters

}
```

在声明版本属性时，我们应该遵循几个规则:

*   每个实体类只能有一个版本属性。
*   对于映射到多个表的实体，它必须放在主表中。
*   版本属性的类型必须是下列之一:`int`、`Integer`、`long`、`Long`、`short`、`Short`、`java.sql.Timestamp`。

我们应该知道，我们可以通过实体检索版本属性的值，但是我们不应该更新或增加它。只有持久性提供者可以做到这一点，所以数据保持一致。

请注意，持久性提供者能够支持没有版本属性的实体的乐观锁定。然而，在使用乐观锁定时，最好总是包含版本属性。

如果我们试图锁定一个不包含这些属性的实体，并且持久性提供者不支持它，那么就会导致一个`PersistenceException`。

## 5.锁定模式

JPA 为我们提供了两种不同的乐观锁模式(和两个别名):

*   `OPTIMISTIC`获取包含版本属性的所有实体的乐观读锁。
*   `OPTIMISTIC_FORCE_INCREMENT`获得与`OPTIMISTIC`相同的乐观锁，并额外增加版本属性值。
*   `READ`是`OPTIMISTIC`的同义词。
*   `WRITE`是`OPTIMISTIC_FORCE_INCREMENT`的同义词。

我们可以在`LockModeType`类中找到上面列出的所有类型。

### 5.1.`OPTIMISTIC` ( `READ`)

我们已经知道，`OPTIMISTIC`和`READ`锁模式是同义词。然而，JPA 规范建议我们在新的应用中使用`OPTIMISTIC`。

**每当我们请求`OPTIMISTIC`锁模式时，持久性提供者将防止我们的数据被脏读和不可重复的读。**

简而言之，它应该确保任何事务都不会提交另一个事务对数据的任何修改

*   已更新或删除但未提交
*   同时已成功更新或删除

### 5.2.`OPTIMISTIC_INCREMENT` ( `WRITE`)

同上，`OPTIMISTIC_INCREMENT`和`WRITE`是同义词，但前者更可取。

`OPTIMISTIC_INCREMENT`必须满足与`OPTIMISTIC`锁定模式相同的条件。**另外，它增加版本属性的值。**然而，它并没有规定是应该立即执行还是可以推迟到提交或刷新时执行。

值得注意的是，当请求`OPTIMISTIC`锁模式时，持久性提供者被允许提供`OPTIMISTIC_INCREMENT`功能。

## 6.使用乐观锁定

我们应该记住，对于版本化的实体，乐观锁定在默认情况下是可用的。但是有几种方法可以明确地请求它。

### 6.1.发现

要请求乐观锁定，我们可以将适当的`LockModeType`作为参数来传递，以找到`EntityManager`的方法:

```
entityManager.find(Student.class, studentId, LockModeType.OPTIMISTIC);
```

### 6.2.询问

启用锁定的另一种方式是使用`Query`对象的`setLockMode`方法:

```
Query query = entityManager.createQuery("from Student where id = :id");
query.setParameter("id", studentId);
query.setLockMode(LockModeType.OPTIMISTIC_INCREMENT);
query.getResultList()
```

### 6.3.显式锁定

我们可以通过调用 EntityManager 的`lock`方法来设置锁:

```
Student student = entityManager.find(Student.class, id);
entityManager.lock(student, LockModeType.OPTIMISTIC);
```

### 6.4.恢复精神

我们可以像前面的方法一样调用`refresh`方法:

```
Student student = entityManager.find(Student.class, id);
entityManager.refresh(student, LockModeType.READ);
```

### 6.5\. NamedQuery

最后一个选项是使用带有`lockMode`属性的@NamedQuery:

```
@NamedQuery(name="optimisticLock",
  query="SELECT s FROM Student s WHERE s.id LIKE :id",
  lockMode = WRITE)
```

## 7.`OptimisticLockException`

**当持久性提供者发现实体上的乐观锁定冲突时，它抛出`OptimisticLockException`。**我们应该知道，由于异常，活动事务总是被标记为回滚。

知道我们如何应对`OptimisticLockException`是件好事。方便的是，这个异常包含对冲突实体的引用。**然而，持久性提供者并不强制在每种情况下都提供它。**不保证对象将可用。

不过，有一种处理上述异常的推荐方法。我们应该通过重新加载或刷新来再次检索实体，最好是在新的事务中。之后，我们可以尝试再更新一次。

## 8.结论

在本文中，我们熟悉了一个可以帮助我们编排并发事务的工具。

乐观锁定使用包含在实体中的版本属性来控制对它们的并发修改。

因此，它确保任何更新或删除不会被覆盖或悄悄地丢失。与悲观锁定相反，它不在数据库级别锁定实体，因此，它不容易受到数据库死锁的影响。

我们已经知道，默认情况下，版本化实体启用乐观锁定。但是，有几种方法可以通过使用各种锁模式类型来显式请求它。

我们还应该记住，每次实体更新发生冲突时，我们都应该期待一个`OptimisticLockException`。

最后，这篇文章的源代码可以在 GitHub 的[上找到。](https://web.archive.org/web/20221123070554/https://github.com/eugenp/tutorials/tree/master/persistence-modules/hibernate-jpa)