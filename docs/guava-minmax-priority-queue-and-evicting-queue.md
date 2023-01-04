# Guava MinMaxPriorityQueue 和驱逐队列指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/guava-minmax-priority-queue-and-evicting-queue>

## 1 **。概述**

在本文中，我们将查看来自番石榴库的`[EvictingQueue](https://web.archive.org/web/20220526051748/https://google.github.io/guava/releases/19.0/api/docs/com/google/common/collect/EvictingQueue.html),` 和`[MinMaxPriorityQueue](https://web.archive.org/web/20220526051748/https://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/MinMaxPriorityQueue.html)` 构造。`EvictingQueue` 是循环缓冲区概念的一个实现。`MinMaxPriorityQueue`让我们可以使用提供的`Comparator.`访问其最低和最高的元素

## 2。`EvictingQueue`

让我们从构造开始——当构造队列的实例时，我们需要提供最大队列大小作为参数。

当我们想要向`EvictingQueue`添加一个新项目，而**队列已满时，它会自动从其头部**中移除一个元素。

与标准队列行为相比，将一个元素添加到整个队列中不会阻塞，但会删除头部元素，并在尾部添加一个新项目。

我们可以把`EvictingQueue` 想象成一个环，我们以只追加的方式向其中插入元素。如果在我们想要添加新元素的位置上有一个元素，我们只需覆盖给定位置上的现有元素。

让我们构造一个最大大小为 10 的`EvictingQueue` 实例。接下来，我们将向它添加 10 个元素:

```
Queue<Integer> evictingQueue = EvictingQueue.create(10);

IntStream.range(0, 10)
  .forEach(evictingQueue::add);

assertThat(evictingQueue)
  .containsExactly(0, 1, 2, 3, 4, 5, 6, 7, 8, 9);
```

如果我们有标准的队列实现，向满队列中添加一个新项会阻塞生产者。

对于`EvictingQueue` 实现，情况并非如此。向其添加新元素将导致头部从其中移除，并且新元素将被添加到尾部:

```
evictingQueue.add(100);

assertThat(evictingQueue)
  .containsExactly(1, 2, 3, 4, 5, 6, 7, 8, 9, 100);
```

通过使用`EvictingQueue` 作为循环缓冲区，我们可以创建非常高效的并发程序。

## 3。`MinMaxPriorityQueue`

`MinMaxPriorityQueue` 提供了对其最小和最大元素的恒定时间访问。

为了获得最少的元素，我们需要调用`peekFirst()` 方法。为了得到最大的元素，我们可以调用`peekLast()` 方法。请注意，这些操作并不从队列中删除元素，它们只是检索它。

元素的排序由需要传递给该队列的构造函数的`Comparator` 来完成。

假设我们有一个`CustomClass` 类，它有一个整数类型的`value` 字段:

```
class CustomClass {
    private Integer value;

    // standard constructor, getters and setters
}
```

让我们创建一个将在`int`类型上使用比较器的`MinMaxPriorityQueue` 。接下来，我们将向队列添加 10 个`CustomClass` 类型的对象:

```
MinMaxPriorityQueue<CustomClass> queue = MinMaxPriorityQueue
  .orderedBy(Comparator.comparing(CustomClass::getValue))
  .maximumSize(10)
  .create();

IntStream
  .iterate(10, i -> i - 1)
  .limit(10)
  .forEach(i -> queue.add(new CustomClass(i)));
```

由于`MinMaxPriorityQueue` 和传递的`Comparator,` 的特征，队列头部的元素将等于 1，队列尾部的元素将等于 10:

```
assertThat(
  queue.peekFirst().getValue()).isEqualTo(1);
assertThat(
  queue.peekLast().getValue()).isEqualTo(10);
```

由于我们的队列容量是 10，并且我们添加了 10 个元素，所以队列已满。向其添加新元素将导致队列中的最后一个元素被移除。让我们添加一个`CustomClass` ，使`value`等于-1:

```
queue.add(new CustomClass(-1));
```

在该动作之后，队列中的最后一个元素将被删除，并且在其尾部的新项目将等于 9。新的头将是-1，因为根据我们在构造队列时传递的`Comparator`,这是新的最小元素:

```
assertThat(
  queue.peekFirst().getValue()).isEqualTo(-1);
assertThat(
  queue.peekLast().getValue()).isEqualTo(9);
```

根据`MinMaxPriorityQueue,` **的规范，在队列已满的情况下，添加一个大于当前最大元素的元素将删除相同的元素——实际上忽略它。**

让我们添加一个 100 的数字，并测试该项目在该操作之后是否在队列中:

```
queue.add(new CustomClass(100));
assertThat(queue.peekFirst().getValue())
  .isEqualTo(-1);
assertThat(queue.peekLast().getValue())
  .isEqualTo(9);
```

正如我们看到的，队列中的第一个元素仍然等于-1，最后一个元素等于 9。因此，添加整数被忽略，因为它大于队列中已经最大的元素。

## 4。结论

在本文中，我们看了一下来自番石榴库的`EvictingQueue`和`MinMaxPriorityQueue` 构造。

我们看到了如何使用`EvictingQueue` 作为循环缓冲区来实现非常高效的程序。

我们将`MinMaxPriorityQueue`与`Comparator`结合使用，以获得对其最小和最大元素的恒定时间访问。

记住这两个队列的特征很重要，因为向它们添加新元素会覆盖队列中已经存在的元素。这与标准队列实现相反，在标准队列实现中，向满队列添加新元素会阻塞生产者线程或抛出异常。

所有这些示例和代码片段的实现都可以在 [GitHub 项目](https://web.archive.org/web/20220526051748/https://github.com/eugenp/tutorials/tree/master/guava-modules/guava-collections)中找到——这是一个 Maven 项目，因此它应该很容易导入和运行。