# 检测 Spring 事务是否是活动的

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-transaction-active>

## 1.概观

检测事务对于审计目的或者处理没有实现良好事务约定的复杂代码库是有用的。

在这个简短的教程中，我们将介绍几种在代码中检测 [Spring 事务](/web/20220524023143/https://www.baeldung.com/transaction-configuration-with-jpa-and-spring)的方法。

## 2.交易配置

为了让事务在 Spring 中工作，必须启用事务管理。如果我们使用一个具有 spring-data-*或 spring-tx 依赖关系的 Spring Boot 项目，Spring 将默认启用事务管理。否则，我们必须启用事务并显式提供事务管理器。

首先，我们需要向我们的`@Configuration`类添加 [`@EnableTransactionManagement`](/web/20220524023143/https://www.baeldung.com/spring-enable-annotations#enabletransactionmanagement) 注释。这为我们的项目启用了 Spring 的注释驱动的事务管理。

接下来，我们必须提供一个 [`PlatformTransactionManager`](/web/20220524023143/https://www.baeldung.com/spring-programmatic-transaction-management#platform-transaction-manager) 或一个`ReactiveTransactionManager` bean。这颗豆子需要一个`DataSource`。我们可以选择使用一些公共库，比如 H2 或 MySQL 的库。我们的实现与本教程无关。

一旦我们启用了事务，我们就可以使用 [`@Transactional`](/web/20220524023143/https://www.baeldung.com/transaction-configuration-with-jpa-and-spring#the-transactional-annotation) 注释来生成事务。

## 3.使用`TransactionSynchronizationManager`

Spring 已经提供了一个名为`TransactionSychronizationManager`的类。幸运的是，这个类有一个静态方法，它允许我们知道我们是否在一个事务中，叫做`isActualTransactionActive()`。

为了测试这一点，让我们用`@Transactional`注释一个测试方法。我们可以断言`isActualTransactionActive()`返回`true`:

```java
@Test
@Transactional
public void givenTransactional_whenCheckingForActiveTransaction_thenReceiveTrue() {
    assertTrue(TransactionSynchronizationManager.isActualTransactionActive());
}
```

类似地，当我们移除`@Transactional`注释时，测试应该断言`false`被返回:

```java
@Test
public void givenNoTransactional_whenCheckingForActiveTransaction_thenReceiveFalse() {
    assertFalse(TransactionSynchronizationManager.isActualTransactionActive());
}
```

## 4.使用 Spring 事务日志记录

也许我们不需要以编程方式检测事务。如果我们想在应用程序的日志中查看事务发生的时间，我们可以在属性文件中启用 Spring 的事务日志:

```java
logging.level.org.springframework.transaction.interceptor = TRACE
```

一旦我们启用了该日志记录级别，事务日志将开始出现:

```java
2020-10-02 14:45:07,162 TRACE - Getting transaction for [com.Class.method]
2020-10-02 14:45:07,273 TRACE - Completing transaction for [com.Class.method]
```

没有任何上下文，这些日志不会提供非常有用的信息。我们可以简单地添加一些我们自己的日志记录，我们应该很容易就能在 Spring 管理的代码中看到事务发生的位置。

## 5.结论

在本文中，我们看到了如何检查 Spring 事务是否是活动的。我们学习了如何使用`TransactionSynchronizationManager.isActualTransactionActive()`方法以编程方式检测事务。我们还发现了如何启用 Spring 的内部事务日志，以防我们希望在日志中看到事务。

和往常一样，代码示例可以在 GitHub 上找到[。](https://web.archive.org/web/20220524023143/https://github.com/eugenp/tutorials/tree/master/persistence-modules/spring-persistence-simple)