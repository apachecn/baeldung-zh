# Spring TestContext 框架中的编程事务

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-test-programmatic-transactions>

## 1。简介

Spring 在整个[应用代码](/web/20221128051929/https://www.baeldung.com/transaction-configuration-with-jpa-and-spring)以及[集成测试](/web/20221128051929/https://www.baeldung.com/spring-jpa-test-in-memory-database)中对声明式事务管理提供了出色的支持。

然而，我们可能偶尔需要对事务边界进行细粒度的控制。

在本文中，我们将看到**如何在事务测试**中以编程方式与 Spring 设置的自动事务进行交互。

## 2。先决条件

让我们假设在我们的 Spring 应用程序中有一些集成测试。

具体来说，我们正在考虑与数据库交互的测试，例如，检查我们的持久层行为是否正确。

让我们考虑一个标准的测试类——注释为事务性的:

```
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(classes = { HibernateConf.class })
@Transactional
public class HibernateBootstrapIntegrationTest { ... }
```

在这样的测试中，**每一个测试方法都被包装在一个事务中，当方法退出**时，事务被回滚。

当然，也可以只注释特定的方法。我们将在本文中讨论的所有内容也适用于这种情况。

## 3。`TestTransaction`班

我们将在本文的剩余部分讨论一个类:`org.springframework.test.context.transaction.TestTransaction`。

这是一个带有一些静态方法的实用程序类，我们可以使用这些方法在测试中与事务进行交互。

每个方法都与测试方法执行期间唯一的当前事务进行交互。

### 3.1。检查当前交易的状态

我们在测试中经常做的一件事是检查事物是否处于它们应该处于的状态。

因此，我们可能希望检查当前是否有活动的事务:

```
assertTrue(TestTransaction.isActive());
```

或者，我们可能有兴趣检查当前事务是否被标记为回滚:

```
assertTrue(TestTransaction.isFlaggedForRollback());
```

如果是，那么 Spring 将在它结束之前回滚它，或者自动或者以编程方式。否则，它会在关闭它之前提交它。

### 3.2。将事务标记为提交或回滚

我们可以通过编程方式更改策略，在关闭事务之前提交或回滚事务:

```
TestTransaction.flagForCommit();
TestTransaction.flagForRollback();
```

通常，测试中的事务在开始时会被标记为回滚。然而，如果方法有一个`@Commit` 注释，它们开始被标记为提交:

```
@Test
@Commit
public void testFlagForCommit() {
    assertFalse(TestTransaction.isFlaggedForRollback());
}
```

注意，这些方法只是标记事务，正如它们的名字所暗示的那样。也就是说，**事务不会立即提交或回滚，而只是在它即将结束时提交或回滚。**

### 3.3。开始和结束交易

要提交或回滚事务，我们要么让方法退出，要么显式结束它:

```
TestTransaction.end();
```

如果以后我们想再次与数据库交互，我们必须启动一个新的事务:

```
TestTransaction.start();
```

请注意，新事务将按照方法的默认设置标记为回滚(或提交)。换句话说，先前对`flagFor…` 的调用对新事务没有任何影响。

## 4。一些实施细节

没什么神奇的。现在我们来看一下它的实现，以了解更多关于 Spring 测试中的事务。

我们可以看到，它的几个方法只是访问当前事务，并封装了它的一些功能。

### 4.1。`TestTransaction`从哪里获取当前交易？

让我们直接看代码:

```
TransactionContext transactionContext
  = TransactionContextHolder.getCurrentTransactionContext();
```

`TransactionContextHolder` 只是一个静态的包裹着一个`ThreadLocal`拿着一个`TransactionContext`。

### 4.2。谁设置线程本地上下文？

如果我们看看谁调用了`setCurrentTransactionContext` 方法，我们会发现只有一个调用者:`TransactionalTestExecutionListener.beforeTestMethod`。

`TransactionalTestExecutionListener`是 Springs 在注释了`@Transactional`的测试上自动配置的监听器。

请注意，`TransactionContext` 并不包含对任何实际事务的引用；相反，它仅仅是`PlatformTransactionManager`的一个门面。

是的，这段代码是高度分层和抽象的。这通常是 Spring 框架的核心部分。

有趣的是，在这种复杂性之下，Spring 没有任何魔法——只有许多必要的簿记、管道、异常处理等等。

## 5。结论

在这个快速教程中，我们看到了如何在基于 Spring 的测试中以编程方式与事务进行交互。

所有这些例子的实现都可以在 GitHub 项目中找到——这是一个 Maven 项目，所以它应该很容易导入和运行。