# Java 优先级队列指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-priorityqueue>

## 1。简介

在这个简短的教程中，我们将讨论优先级队列的 Java 实现。首先，我们将看到标准用法，并通过以自然和相反的顺序对队列进行排序来展示一些例子。

最后，我们将看到如何使用 [Java `Comparator` s](/web/20221208024909/https://www.baeldung.com/java-comparator-comparable#comparator) 来定义定制订单。

## 2。`java.util.PriorityQueue`

从 JDK 1.5 开始提供了`java.util.PriorityQueue`类，它还包含了 [`AbstractQueue`](/web/20221208024909/https://www.baeldung.com/java-queue#abstract_queue) 的其他实现。**正如我们可以从它的名字推断的那样，我们使用`PriorityQueue`来维护给定集合中定义的顺序:队列的第一个元素(`head`)是相对于我们指定的顺序最次要的元素。**队列的每次检索操作(`poll`、`remove`或`peek`)都会读取队列的头部。

在内部，`PriorityQueue`依赖于一个对象数组。如果初始指定容量(在 JDK 17 中默认为 11)不足以存储所有项目，该数组会自动调整大小。虽然不强制给一个`PriorityQueue`一个初始容量，但是如果我们已经知道了我们的`collection`的大小，就有可能避免自动调整大小，这会消耗 CPU 周期，我们最好节省下来。

在 [Javadoc](https://web.archive.org/web/20221208024909/https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/util/PriorityQueue.html) 中，规定了入队和出队方法(`offer`、`poll`、`remove`和`add`)的实现时间为 O(log(n))。这要归功于平衡的二进制堆数据结构，这种结构在每次编辑`Queue`时都会被不断地维护。取而代之的是`remove(Object)`和 `contains(Object)`方法的线性时间和检索方法(`peek`、`element`和`size`)的恒定时间。

## 3。自然和逆排序

在上一篇文章中，我们介绍了插入到 `PriorityQueue`中的 [元素是如何基于它们的自然排序](/web/20221208024909/https://www.baeldung.com/java-queue#priority_queues) 进行排序的。这是因为用空值`Comparator`初始化优先级队列将使用 [`compare`](/web/20221208024909/https://www.baeldung.com/java-comparator-comparable#comparable) 操作直接对元素进行排序。

**作为一个例子，现在让我们看看，通过提供一个标准的`Integer`自然排序比较器或 null，队列将以同样的方式排序**:

```java
PriorityQueue<Integer> integerQueue = new PriorityQueue<>();
PriorityQueue<Integer> integerQueueWithComparator = new PriorityQueue<>((Integer c1, Integer c2) -> Integer.compare(c1, c2));

integerQueueWithComparator.add(3);
integerQueue.add(3);

integerQueueWithComparator.add(2);
integerQueue.add(2);

integerQueueWithComparator.add(1);
integerQueue.add(1);

assertThat(integerQueue.poll())
     .isEqualTo(1)
     .isEqualTo(integerQueueWithComparator.poll());

assertThat(integerQueue.poll())
     .isEqualTo(2)
     .isEqualTo(integerQueueWithComparator.poll());

assertThat(integerQueue.poll())
     .isEqualTo(3)
     .isEqualTo(integerQueueWithComparator.poll());
```

现在让我们创建一个按逆自然顺序排序的`PriorityQueue`。我们可以通过使用`static`方法`java.util.Collections.reverseOrder()`来实现:

```java
PriorityQueue<Integer> reversedQueue = new PriorityQueue<>(Collections.reverseOrder());

reversedQueue.add(1);
reversedQueue.add(2);
reversedQueue.add(3);

assertThat(reversedQueue.poll()).isEqualTo(3);
assertThat(reversedQueue.poll()).isEqualTo(2);
assertThat(reversedQueue.poll()).isEqualTo(1);
```

## 4。定制订单

现在让我们试着为一个定制类`.`定义一个特殊的顺序。首先，这个类应该实现`Comparable`接口，或者我们应该在`Queue`的实例化中提供一个`Comparator`，否则将抛出一个`[ClassCastException](/web/20221208024909/https://www.baeldung.com/java-classcastexception)`。

例如，让我们创建一个`ColoredNumber`类来演示这种行为:

```java
public class ColoredNumber {

   private int value;
   private String color;

   public ColoredNumber(int value, String color) {
       this.value = value;
       this.color = color;
   }
 // getters and setters...
}
```

**当我们试图在`PriorityQueue`中使用这个类时，它会抛出一个异常**:

```java
PriorityQueue<ColoredNumber> queue = new PriorityQueue<>();
queue.add(new ColoredNumber(3,"red"));
queue.add(new ColoredNumber(2, "blue"));
```

**这是因为`PriorityQueue`不知道如何通过将`ColoredNumber`对象与同类的其他对象进行比较来对其进行排序。**

我们可以通过在构造函数中提供一个`Comparator`来提供排序，就像我们在前面的例子中所做的那样，或者我们可以实现`Comparable`接口:

```java
public final class ColoredNumberComparable implements Comparable<ColoredNumber> {
// ...
@Override
public int compareTo(ColoredNumberComparable o) {
   if ((this.color.equals("red") && o.color.equals("red")) ||
           (!this.color.equals("red") && !o.color.equals("red"))) {
       return Integer.compare(this.value, o.value);
   }
   else if (this.color.equals("red")) {
       return -1;
   }
   else {
       return 1;
   }
}
```

这将允许首先考虑“红色”来对每个项目进行排序，然后以自然排序的方式对值进行排序，这意味着将首先返回所有红色的对象:

```java
PriorityQueue<ColoredNumberComparable> queue = new PriorityQueue<>();
queue.add(new ColoredNumberComparable(10, "red"));
queue.add(new ColoredNumberComparable(20, "red"));
queue.add(new ColoredNumberComparable(1, "blue"));
queue.add(new ColoredNumberComparable(2, "blue"));

ColoredNumberComparable first = queue.poll();
assertThat(first.getColor()).isEqualTo("red");
assertThat(first.getValue()).isEqualTo(10);

queue.poll();

ColoredNumberComparable third = queue.poll();
assertThat(third.getColor()).isEqualTo("blue");
assertThat(third.getValue()).isEqualTo(1);
```

关于多线程的最后一点:优先级队列的 Java 实现不是`synchronized`，这意味着多个线程不应该同时使用 Java `PriorityQueue`的同一个实例。

如果有多个线程需要访问一个`PriorityQueue`实例，我们应该使用线程安全的 [`java.util.concurrent.PriorityBlockingQueue`](/web/20221208024909/https://www.baeldung.com/java-priority-blocking-queue) 类来代替。

## 5。结论

在本文中，我们已经看到了 Java `PriorityQueue`实现是如何工作的。我们从 JDK 内部的课程和他们的表演写作和阅读元素开始。然后，我们用自然排序和逆排序演示了一个`PriorityQueue`。最后，我们提供了一个用户定义类的定制`Comparable`实现，并验证了它的排序行为。

与往常一样，代码可以在 GitHub 上的[而不是](https://web.archive.org/web/20221208024909/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-collections-4) [中获得。](https://web.archive.org/web/20221208024909/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-collections-4)