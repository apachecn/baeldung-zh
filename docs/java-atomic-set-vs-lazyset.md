# Java 原子变量中 set()和 lazySet()的区别

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-atomic-set-vs-lazyset>

## 1.概观

在本教程中，我们将看看 Java [原子](/web/20221208143845/https://www.baeldung.com/java-atomic-variables)类`AtomicInteger`和`AtomicReference`的方法`set()`和`lazySet()`之间的区别。

## 2.原子变量——快速回顾

Java 中的原子变量允许我们轻松地对类引用或字段执行线程安全的操作，而不必添加像监视器或互斥这样的并发原语。

它们是在`java.util.concurrent.atomic`包下定义的，尽管它们的 API 因原子类型不同而不同，但大多数都支持`set()`和`lazySet()` 方法。

为了简单起见，我们将在整篇文章中使用`AtomicReference`和`AtomicInteger`，但是同样的原则也适用于其他原子类型。

## 3.`set()`法

**`set()`方法相当于写一个 [`volatile`](/web/20221208143845/https://www.baeldung.com/java-volatile) 字段**。

在调用了`set(),` 之后，当我们从一个不同的线程中使用`get()`方法访问该字段时，变化立即可见。这意味着该值从 CPU 缓存刷新到所有 CPU 内核共用的内存层。

为了展示上述功能，让我们创建一个最小的[生产者-消费者](/web/20221208143845/https://www.baeldung.com/java-producer-consumer-problem)控制台应用程序:

```java
public class Application {

    AtomicInteger atomic = new AtomicInteger(0);

    public static void main(String[] args) {
        Application app = new Application();
        new Thread(() -> {
            for (int i = 0; i < 10; i++) {
                app.atomic.set(i);
                System.out.println("Set: " + i);
                Thread.sleep(100);
            }
        }).start();

        new Thread(() -> {
            for (int i = 0; i < 10; i++) {
                synchronized (app.atomic) {
                    int counter = app.atomic.get();
                    System.out.println("Get: " + counter);
                }
                Thread.sleep(100);
            }
        }).start();
    }
}
```

在控制台中，我们应该看到一系列“设置”和“获取”消息:

```java
Set: 3
Set: 4
Get: 4
Get: 5
```

**表明[高速缓存一致性](https://web.archive.org/web/20221208143845/https://en.wikipedia.org/wiki/Cache_coherence)的事实是“Get”语句中的值总是等于或大于它们上面的“Set”语句中的值。**

这种行为虽然非常有用，但会影响性能。如果我们可以在不需要缓存一致性的情况下避免它，那就太好了。

## 4.`lazySet()`法

`lazySet()` 方法与`set()`方法相同，但没有缓存刷新。

**换句话说，我们的改变最终只能被其他线程看到**。**这意味着从不同的线程调用更新的`AtomicReference`上的`get()`可能会给我们旧的值。**

为了看到这一点，让我们在之前的控制台应用程序中更改第一个线程的`Runnable`:

```java
for (int i = 0; i < 10; i++) {
    app.atomic.lazySet(i);
    System.out.println("Set: " + i);
    Thread.sleep(100);
}
```

新的“设置”和“获取”消息可能不总是递增的:

```java
Set: 4
Set: 5
Get: 4
Get: 5
```

由于线程的性质，我们可能需要重新运行几次应用程序才能触发此行为。消费者线程首先检索值 4，即使生产者线程已经将`AtomicInteger`设置为 5，这意味着当使用`lazySet()`时，系统最终是一致的。

**用更专业的术语来说，我们说`lazySet()`方法不像代码中的发生前边缘**那样，与它们的`set()`对应物相反。

## 5.何时使用`lazySet()`

现在还不清楚我们什么时候应该使用`lazySet()`，因为它和`set()`的区别很微妙。我们需要仔细分析问题，不仅要确保我们将获得性能提升，还要确保在多线程环境中的正确性。

**我们可以使用它的一种方法是，一旦我们不再需要它，就用`null`替换对象引用。**通过这种方式，我们表明该对象有资格进行垃圾收集，而不会招致任何性能损失。我们假设其他线程可以使用这个被否决的值，直到它们看到`AtomicReference`是`null`。

不过，一般来说，当我们想要对原子变量进行更改时，我们应该使用**,我们知道这种更改不需要立即对其他线程可见。**

## 6.结论

在本文中，我们看了一下原子类的`set()`和`lazySet()`方法之间的区别。我们还学习了何时使用哪种方法。

和往常一样，例子的源代码可以在 GitHub 上找到[。](https://web.archive.org/web/20221208143845/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-concurrency-advanced-4)