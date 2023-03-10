# 新的番石榴 21 common.util.concurrent

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/guava-21-util-concurrent>

## 1。简介

在[上一篇文章](/web/20220625234753/https://www.baeldung.com/guava-21-new)中，我们开始探索`common.collect`包中引入的新功能。

在这篇简短的文章中，让我们看看对`common.util.concurrent`包的补充。

## T2`2\. AtomicLongMap`

在并发场景中，标准的`HashMap`可能不会很好地工作，因为它根本不是并发的。在这个特定的场景中，`AtomicLongMap`通过以线程安全的方式存储`Long`值来拯救你。

很久以前在番石榴中被引进。现在，添加了四个新方法。

### `**2.1\. accumulateAndGet()**`

`accumulateAndGet()`方法通过使用累加器函数将与键链接的值与现有值合并来更新该值。然后，它返回更新后的值:

```java
@Test
public void accumulateAndGet_withLongBinaryOperator_thenSuccessful() {
    long noOfStudents = 56;
    long oldValue = courses.get(SPRING_COURSE_KEY);

    long totalNotesRequired = courses.accumulateAndGet(
      "Guava", 
      noOfStudents, 
      (x, y) -> (x * y));

    assertEquals(totalNotesRequired, oldValue * noOfStudents);
}
```

### `**2.2\. getAndAccumulate()**`

这个方法具有与上面定义的相似的功能，但是它返回旧的值而不是更新的值(如同中的操作顺序所示)。

### `**2.3\. updateAndGet()**`

方法使用作为第二个参数提供的指定函数更新键的当前值。然后，它返回键的更新值:

```java
@Test
public void updateAndGet_withLongUnaryOperator_thenSuccessful() {
    long beforeUpdate = courses.get(SPRING_COURSE_KEY);
    long onUpdate = courses.updateAndGet(
      "Guava",
      (x) -> (x / 2));
    long afterUpdate = courses.get(SPRING_COURSE_KEY);

    assertEquals(onUpdate, afterUpdate);
    assertEquals(afterUpdate, beforeUpdate / 2);
}
```

### `**2.4\. getAndUpdate()**`

这个方法的工作方式与`updateAndGet()` 非常相似，但是它返回的是键的旧值，而不是更新后的值。

## `**3\. Monitor**`

monitor 类被认为是对`ReentrantLock`的替代，它可读性更好，不容易出错。

### T2`3.1\. Monitor.newGuard()`

Guava 21 增加了一个新方法——`newGuard()`——返回一个`Monitor.Guard`实例，作为一个线程可以等待的布尔条件:

```java
public class MonitorExample {
    private List<String> students = new ArrayList<String>();
    private static final int MAX_SIZE = 100;

    private Monitor monitor = new Monitor();

    public void addToCourse(String item) throws InterruptedException {
        Monitor.Guard studentsBelowCapacity = monitor.newGuard(this::isStudentsCapacityUptoLimit);
        monitor.enterWhen(studentsBelowCapacity);
        try {
            students.add(item);
        } finally {
            monitor.leave();
        }
    }

    public Boolean isStudentsCapacityUptoLimit() {
        return students.size() > MAX_SIZE;
    }
}
```

## `**4\. MoreExecutors**`

这个类中没有增加任何东西，但是已经删除了`sameThreadExecutor()` API。从 18.0 版开始，这种方法被弃用，建议使用`directExecutor()`或`newDirectExecutorService()`来代替。

## `**5\. ForwardingBlockingDeque**`

`ForwardingBlockingDeque`是一个已经从`common.collect`移走的现有类，因为`BlockingQueue`更像是一个并发集合类型，而不是一个标准集合。

## 6。结论

Guava 21 不仅试图引入新的实用程序以跟上 Java 8 的步伐，还改进了现有的模型以使其更有意义。

和往常一样，本文中的代码示例可以在 [GitHub 库](https://web.archive.org/web/20220625234753/https://github.com/eugenp/tutorials/tree/master/guava-modules/guava-21)中找到。