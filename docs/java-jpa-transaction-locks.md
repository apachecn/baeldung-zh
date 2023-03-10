# 在 Spring Data JPA 中启用事务锁

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-jpa-transaction-locks>

## 1.概观

在这个快速教程中，我们将讨论在 Spring Data JPA 中为[自定义查询方法](/web/20221005062618/https://www.baeldung.com/spring-data-jpa-query)和预定义的存储库 CRUD 方法启用事务锁。

我们还将了解不同的锁类型和设置事务锁超时。

## 2.锁类型

JPA 定义了两种主要的锁类型，悲观锁和乐观锁。

### 2.1。悲观锁定

**当我们在一个事务中使用[悲观锁定](/web/20221005062618/https://www.baeldung.com/jpa-pessimistic-locking)并访问一个实体时，它会被立即锁定**。事务通过提交或回滚事务来释放锁。

### 2.2。乐观锁定

**在[乐观锁定](/web/20221005062618/https://www.baeldung.com/jpa-optimistic-locking)中，事务不会立即锁定实体。**相反，事务通常用分配给它的版本号保存实体的状态。

当我们试图在不同的事务中更新实体的状态时，事务会在更新期间将保存的版本号与现有的版本号进行比较。

此时，如果版本号不同，就意味着实体不能被修改。如果有一个活动的事务，那么该事务将被回滚，底层 JPA 实现将抛出一个`[OptimisticLockException](https://web.archive.org/web/20221005062618/https://docs.oracle.com/javaee/7/api/javax/persistence/OptimisticLockException.html).`

除了版本号方法，我们可以使用其他方法，比如时间戳、散列值计算或序列化校验和，这取决于哪种方法最适合我们当前的开发环境。

## 3.对查询方法启用事务锁

**要获取实体上的锁，我们可以通过传递所需的锁模式类型** `.` 用`[Lock](https://web.archive.org/web/20221005062618/https://docs.spring.io/spring-data/jpa/docs/current/api/org/springframework/data/jpa/repository/Lock.html)`注释来注释目标查询方法

[锁定模式类型](https://web.archive.org/web/20221005062618/https://docs.oracle.com/javaee/7/api/javax/persistence/LockModeType.html)是锁定实体时要指定的枚举值。然后，指定的锁模式被传播到数据库，以在实体对象上应用相应的锁。

要在 Spring 数据 JPA 存储库的定制查询方法上指定一个锁，我们可以用`@Lock`注释该方法，并指定所需的锁模式类型:

```java
@Lock(LockModeType.OPTIMISTIC_FORCE_INCREMENT)
@Query("SELECT c FROM Customer c WHERE c.orgId = ?1")
public List<Customer> fetchCustomersByOrgId(Long orgId);
```

为了强制锁定预定义的存储库方法，比如`findAll`或`findById(id)`，我们必须在存储库中声明该方法，并用`Lock`注释对该方法进行注释:

```java
@Lock(LockModeType.PESSIMISTIC_READ)
public Optional<Customer> findById(Long customerId);
```

当锁被显式启用并且没有活动事务时，底层 JPA 实现将抛出一个`[TransactionRequiredException](https://web.archive.org/web/20221005062618/https://docs.oracle.com/javaee/7/api/javax/persistence/TransactionRequiredException.html)`。

如果锁不能被授予并且锁定冲突没有导致事务回滚，JPA 抛出一个 [`LockTimeoutException`](https://web.archive.org/web/20221005062618/https://docs.oracle.com/javaee/7/api/javax/persistence/LockTimeoutException.html) 。但是它没有将活动事务标记为回滚。

## 4.设置事务锁定超时

使用悲观锁定时，数据库将尝试立即锁定实体。当不能立即获得锁时，底层 JPA 实现抛出一个`LockTimeoutException`。为了避免这种异常，我们可以指定锁超时值。

在 Spring Data JPA 中，可以通过在查询方法上放置一个 [`QueryHint`](https://web.archive.org/web/20221005062618/https://docs.oracle.com/javaee/7/api/javax/persistence/QueryHint.html) 来使用 [`QueryHints`](https://web.archive.org/web/20221005062618/https://docs.spring.io/spring-data/jpa/docs/current/api/index.html?org/springframework/data/jpa/repository/QueryHints.html) 注释来指定锁超时:

```java
@Lock(LockModeType.PESSIMISTIC_READ)
@QueryHints({@QueryHint(name = "javax.persistence.lock.timeout", value = "3000")})
public Optional<Customer> findById(Long customerId);
```

关于在不同作用域设置锁超时提示的更多细节可以在本文 [ObjectDB 文章](https://web.archive.org/web/20221005062618/https://www.objectdb.com/java/jpa/persistence/lock#Pessimistic_Locking_)中找到。

## 5.结论

在本教程中，我们学习了不同类型的事务锁模式。我们已经学习了如何在 Spring Data JPA 中启用事务锁。我们还讨论了设置锁超时。

在正确的位置应用正确的事务锁有助于在大量并发使用的应用程序中维护数据完整性。

当事务需要严格遵守 ACID 规则时，我们应该使用悲观锁。当我们需要允许多个并发读取，并且在应用程序上下文中最终的一致性是可接受的时，应该应用乐观锁定。

当然，悲观锁定和乐观锁定的示例代码都可以在 Github 上找到[。](https://web.archive.org/web/20221005062618/https://github.com/eugenp/tutorials/tree/master/persistence-modules/hibernate-jpa)