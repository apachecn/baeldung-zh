# 使用多元宇宙的 Java 软件事务内存

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-multiverse-stm>

## 1。概述

在本文中，我们将关注`[Multiverse](https://web.archive.org/web/20221126224013/https://github.com/pveentjer/Multiverse)` 库——它帮助我们在 Java 中实现`Software Transactional Memory`的概念。

使用这个库的构造，我们可以在共享状态上创建一个同步机制——这是一个比 Java 核心库的标准实现更优雅、可读性更好的解决方案。

## 2。Maven 依赖关系

首先，我们需要将 [`multiverse-core`](https://web.archive.org/web/20221126224013/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.multiverse%22%20AND%20a%3A%22multiverse-core%22) 库添加到我们的 pom 中:

```java
<dependency>
    <groupId>org.multiverse</groupId>
    <artifactId>multiverse-core</artifactId>
    <version>0.7.0</version>
</dependency>
```

## 3 .多元 API〔t1〕

让我们从一些基本的开始。

软件事务内存(STM)是从 SQL 数据库世界移植来的概念——其中每个操作都在满足`ACID (Atomicity, Consistency, Isolation, Durability)`属性的事务中执行。这里，**只满足原子性、一致性和隔离性，因为机制运行在内存中。**

**多元宇宙库中的主接口是**`**TxnObject**` ——每个事务性对象都需要实现它，库为我们提供了许多我们可以使用的特定子类。

每个需要放在临界区内的操作，只能由一个线程访问，并使用任何事务对象——都需要包装在`StmUtils.atomic()`方法中。临界区是程序中不能被多个线程同时执行的地方，所以对它的访问应该由某种同步机制来保护。

如果事务中的某个操作成功，该事务将被提交，并且其他线程将可以访问新的状态。如果出现错误，事务将不会被提交，因此状态不会改变。

最后，如果两个线程想要修改一个事务中的同一状态，只有一个会成功并提交其更改。下一个线程将能够在其事务中执行其操作。

## 4。使用 STM 实现账户逻辑

现在让我们来看一个例子。

假设我们想使用`Multiverse`库提供的 STM 创建一个银行账户逻辑。我们的`Account` 对象将拥有一个`TxnLong` 类型的`lastUpadate` 时间戳，以及一个`TxnInteger` 类型的`balance` 字段，该字段存储给定账户的当前余额。

`TxnLong` 和`TxnInteger` 是来自`Multiverse`的类。它们必须在一个事务中执行。否则，将引发异常。我们需要使用`StmUtils` 来创建事务对象的新实例:

```java
public class Account {
    private TxnLong lastUpdate;
    private TxnInteger balance;

    public Account(int balance) {
        this.lastUpdate = StmUtils.newTxnLong(System.currentTimeMillis());
        this.balance = StmUtils.newTxnInteger(balance);
    }
}
```

接下来，我们将创建`adjustBy()` 方法——它将按照给定的数量增加余额。该操作需要在事务中执行。

如果其中引发了任何异常，事务将结束而不提交任何更改:

```java
public void adjustBy(int amount) {
    adjustBy(amount, System.currentTimeMillis());
}

public void adjustBy(int amount, long date) {
    StmUtils.atomic(() -> {
        balance.increment(amount);
        lastUpdate.set(date);

        if (balance.get() <= 0) {
            throw new IllegalArgumentException("Not enough money");
        }
    });
}
```

如果我们想要获得给定帐户的当前余额，我们需要从 balance 字段中获得值，但是也需要使用原子语义来调用它:

```java
public Integer getBalance() {
    return balance.atomicGet();
}
```

## 5。测试账户

让我们来测试一下我们的`Account`逻辑。首先，我们想简单地从账户中减去给定的余额:

```java
@Test
public void givenAccount_whenDecrement_thenShouldReturnProperValue() {
    Account a = new Account(10);
    a.adjustBy(-5);

    assertThat(a.getBalance()).isEqualTo(5);
}
```

接下来，假设我们从账户中提款，使得余额为负。该操作应该抛出一个异常，并保持帐户不变，因为该操作是在事务中执行的，并未提交:

```java
@Test(expected = IllegalArgumentException.class)
public void givenAccount_whenDecrementTooMuch_thenShouldThrow() {
    // given
    Account a = new Account(10);

    // when
    a.adjustBy(-11);
} 
```

现在让我们测试一个并发性问题，当两个线程想同时减少一个余额时会出现这个问题。

如果一个线程想让它减 5，另一个线程想让它减 6，那么这两个操作中的一个应该会失败，因为给定帐户的当前余额等于 10。

我们将向`ExecutorService`提交两个线程，并使用`CountDownLatch` 同时启动它们:

```java
ExecutorService ex = Executors.newFixedThreadPool(2);
Account a = new Account(10);
CountDownLatch countDownLatch = new CountDownLatch(1);
AtomicBoolean exceptionThrown = new AtomicBoolean(false);

ex.submit(() -> {
    try {
        countDownLatch.await();
    } catch (InterruptedException e) {
        e.printStackTrace();
    }

    try {
        a.adjustBy(-6);
    } catch (IllegalArgumentException e) {
        exceptionThrown.set(true);
    }
});
ex.submit(() -> {
    try {
        countDownLatch.await();
    } catch (InterruptedException e) {
        e.printStackTrace();
    }
    try {
        a.adjustBy(-5);
    } catch (IllegalArgumentException e) {
        exceptionThrown.set(true);
    }
});
```

同时启动两个动作后，其中一个会抛出异常:

```java
countDownLatch.countDown();
ex.awaitTermination(1, TimeUnit.SECONDS);
ex.shutdown();

assertTrue(exceptionThrown.get());
```

## 6。从一个账户转移到另一个账户

假设我们想把钱从一个账户转到另一个账户。我们可以在`Account` 类上实现`transferTo()` 方法，方法是传递另一个`Account` ,我们想把给定金额的钱转移到这个【】:

```java
public void transferTo(Account other, int amount) {
    StmUtils.atomic(() -> {
        long date = System.currentTimeMillis();
        adjustBy(-amount, date);
        other.adjustBy(amount, date);
    });
}
```

所有逻辑都在一个事务中执行。这将保证当我们想要转移的金额高于给定账户的余额时，两个账户都不会受到影响，因为交易不会提交。

让我们测试传输逻辑:

```java
Account a = new Account(10);
Account b = new Account(10);

a.transferTo(b, 5);

assertThat(a.getBalance()).isEqualTo(5);
assertThat(b.getBalance()).isEqualTo(15);
```

我们只需创建两个账户，把钱从一个账户转到另一个账户，一切都按预期进行。接下来，假设我们想转移比账户上可用资金更多的资金。`transferTo()` 调用将抛出`IllegalArgumentException,` ，并且不会提交更改:

```java
try {
    a.transferTo(b, 20);
} catch (IllegalArgumentException e) {
    System.out.println("failed to transfer money");
}

assertThat(a.getBalance()).isEqualTo(5);
assertThat(b.getBalance()).isEqualTo(15);
```

请注意，`a`和`b` 账户的余额与调用`transferTo()` 方法之前的余额相同。

## 7。STM 是死锁安全的

当我们使用标准的 Java 同步机制时，我们的逻辑很容易出现死锁，没有办法从中恢复。

当我们想把钱从账户`a`转移到账户`b`时，死锁就会发生。在标准的 Java 实现中，一个线程需要锁定账户`a`，然后锁定账户`b`。假设与此同时，另一个线程想要将钱从账户`b`转移到账户`a`。另一个线程锁定帐户`b`，等待帐户`a`解锁。

不幸的是，帐户`a`的锁被第一个线程持有，而帐户`b`的锁被第二个线程持有。这种情况将导致我们的程序无限期阻塞。

幸运的是，当使用 STM 实现`transferTo()`逻辑时，我们不需要担心死锁，因为 STM 是死锁安全的。让我们用我们的`transferTo()`方法来测试一下。

假设我们有两个线程。第一个线程想从账户`a`转移一些钱到账户`b`，第二个线程想从账户`b`转移一些钱到账户`a`。我们需要创建两个帐户并启动两个线程，这两个线程将同时执行`transferTo()` 方法:

```java
ExecutorService ex = Executors.newFixedThreadPool(2);
Account a = new Account(10);
Account b = new Account(10);
CountDownLatch countDownLatch = new CountDownLatch(1);

ex.submit(() -> {
    try {
        countDownLatch.await();
    } catch (InterruptedException e) {
        e.printStackTrace();
    }
    a.transferTo(b, 10);
});
ex.submit(() -> {
    try {
        countDownLatch.await();
    } catch (InterruptedException e) {
        e.printStackTrace();
    }
    b.transferTo(a, 1);

});
```

开始处理后，两个帐户都将有正确的余额字段:

```java
countDownLatch.countDown();
ex.awaitTermination(1, TimeUnit.SECONDS);
ex.shutdown();

assertThat(a.getBalance()).isEqualTo(1);
assertThat(b.getBalance()).isEqualTo(19);
```

## 8。结论

在本教程中，我们看了一下`Multiverse` 库，以及如何利用软件事务内存中的概念来创建无锁和线程安全的逻辑。

我们测试了实现的逻辑的行为，发现使用 STM 的逻辑是无死锁的。

所有这些示例和代码片段的实现都可以在 [GitHub 项目](https://web.archive.org/web/20221126224013/https://github.com/eugenp/tutorials/tree/master/libraries)中找到——这是一个 Maven 项目，因此它应该很容易导入和运行。