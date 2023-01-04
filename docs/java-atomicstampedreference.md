# Java 中的 AtomicStampedReference 指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-atomicstampedreference>

## 1.概观

在之前的文章中，我们了解到 [`AtomicStampedReference`可以预防 ABA 问题](/web/20220628084028/https://www.baeldung.com/cs/aba-concurrency)。

在本教程中，我们将仔细看看如何最好地使用它。

## 2.我们为什么需要`AtomicStampedReference`？

首先， **`AtomicStampedReference`为我们提供了一个对象引用变量和一个我们可以原子读写的标记**。**我们可以认为这个标记有点像时间戳或版本号**。

简单地说，添加一个标记 **允许我们检测另一个线程何时改变了共享引用，从原始引用 A，到新引用 B，再回到原始引用 A** 。

让我们看看它在实践中的表现。

## 3.银行账户示例

假设一个银行账户有两个数据:余额和最后修改日期。只要余额发生变化，最后修改日期就会更新。通过观察这个最后修改日期，我们可以知道帐户已经更新。

### 3.1.读取值及其戳记

首先，让我们假设我们的引用持有一个帐户余额:

```java
AtomicStampedReference<Integer> account = new AtomicStampedReference<>(100, 0);
```

请注意，我们提供了余额 100 英镑和邮票 0 英镑。

为了访问余额，我们可以在我们的`account`成员变量上使用`AtomicStampedReference.getReference()`方法。

同样，我们可以通过`AtomicStampedReference.getStamp()`获得图章。

### 3.2.更改值及其戳记

现在，让我们回顾一下如何自动设置一个`AtomicStampedReference`的值。

如果我们想更改帐户的余额，我们需要更改余额和戳记:

```java
if (!account.compareAndSet(balance, balance + 100, stamp, stamp + 1)) {
    // retry
}
```

`compareAndSet`方法返回一个表示成功或失败的布尔值。失败意味着自从我们最后一次读取以来，天平或印记已经改变。

正如我们所见，**使用它们的 getters 可以很容易地检索引用和戳记。**

**但是，如上所述，当我们想要使用 CAS 更新它们的值时，我们需要它们的最新版本**。为了自动检索这两条信息，我们需要同时获取它们。

幸运的是，`AtomicStampedReference`为我们提供了一个基于数组的 API 来实现这一点。让我们通过为我们的`Account`类实现`withdrawal()`方法来演示它的用法:

```java
public boolean withdrawal(int funds) {
    int[] stamps = new int[1];
    int current = this.account.get(stamps);
    int newStamp = this.stamp.incrementAndGet();
    return this.account.compareAndSet(current, current - funds, stamps[0], newStamp);
}
```

同样，我们可以添加`deposit()`方法:

```java
public boolean deposit(int funds) {
    int[] stamps = new int[1];
    int current = this.account.get(stamps);
    int newStamp = this.stamp.incrementAndGet();
    return this.account.compareAndSet(current, current + funds, stamps[0], newStamp);
}
```

我们刚刚写的东西的好处是，我们可以在提取或存放之前知道，没有其他线程改变了平衡，甚至回到了我们上次读取后的状态。

**例如，考虑下面的线程交错:**

余额设置为 100 美元。线程 1 运行`deposit(100)`至以下点:

```java
int[] stamps = new int[1];
int current = this.account.get(stamps);
int newStamp = this.stamp.incrementAndGet(); 
// Thread 1 is paused here
```

意味着存款还没有完成。

然后，线程 2 运行 `deposit(100)`和`withdraw(100)`，使余额达到$200，然后回到$100。

最后，线程 1 运行:

```java
return this.account.compareAndSet(current, current + 100, stamps[0], newStamp);
```

线程 1 将成功地检测到，自上次读取以来，其他线程已经更改了帐户余额，即使余额本身与线程 1 读取它时的余额相同。

### 3.3.测试

测试起来很棘手，因为这依赖于一种非常特殊的线程交错。但是，让我们至少编写一个简单的单元测试来验证存款和取款的工作情况:

```java
public class ThreadStampedAccountUnitTest {

    @Test
    public void givenMultiThread_whenStampedAccount_thenSetBalance() throws InterruptedException {
        StampedAccount account = new StampedAccount();

        Thread t = new Thread(() -> {
            while (!account.deposit(100)) {
                Thread.yield();
            }
        });
        t.start();

        Thread t2 = new Thread(() -> {
            while (!account.withdrawal(100)) {
                Thread.yield();
            }
        });
        t2.start();

        t.join(10_000);
        t2.join(10_000);

        assertFalse(t.isAlive());
        assertFalse(t2.isAlive());

        assertEquals(0, account.getBalance());
        assertTrue(account.getStamp() > 0);
    }
}
```

### 3.4.选择下一枚邮票

从语义上来说，时间戳就像一个时间戳或版本号，**，所以它通常总是递增的**。也可以使用随机数生成器。

这样做的原因是，如果印记可以被改变成以前的样子，这可能会使`AtomicStampedReference`的目的落空。**`AtomicStampedReference`本身并不强制执行这种约束，所以由我们来遵循这种做法。**

## 4.结论

总之，`AtomicStampedReference`是一个强大的并发实用程序，它提供了一个引用和一个可以自动读取和更新的标记。它是为 A-B-A 检测而设计的，应该优先于其他并发类，比如关注 A-B-A 问题的`AtomicReference`。

和往常一样，我们可以在 GitHub 上找到可用的代码[。](https://web.archive.org/web/20220628084028/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-concurrency-advanced-3)