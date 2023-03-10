# HashSet 与 ArrayList 中 contains()的性能

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-hashset-arraylist-contains-performance>

## 1。简介

在本快速指南中，**我们将讨论`java.util.` `HashSet`和`java.util.` `ArrayList`** 中可用的`contains()`方法的性能。它们都是用于存储和操作对象的集合。

`HashSet`是用于存储唯一元素的集合。要了解更多关于`HashSet,`的信息，请点击[链接](/web/20221208143940/https://www.baeldung.com/java-hashset)。

`ArrayList`是`java.util.List`接口的一个流行实现。

我们有一篇关于`ArrayList`可用[的扩展文章在这里](/web/20221208143940/https://www.baeldung.com/java-arraylist)。

## 2。`HashSet.contains()`

在内部，`HashSet`实现基于一个`HashMap `实例。`contains()`方法调用`HashMap.containsKey(object)`。

这里，它检查`object`是否在内部映射中。内部映射将数据存储在节点内部，称为存储桶。每个桶对应一个用`hashCode() `方法生成的哈希码。所以`contains()`实际上是用`hashCode() `的方法找到了`object's `的位置。

现在让我们来确定查找时间复杂度。在继续之前，请确保您熟悉 [Big-O 符号](/web/20221208143940/https://www.baeldung.com/big-o-notation)。

平均而言，`HashSet`的**在`O(1)`时间**内运行。**获取`object's`铲斗位置是一个恒定时间操作。考虑到可能的冲突，查找时间可能会上升到`log(n)` ，因为内部桶结构是一个`TreeMap`。**

这是对 Java 7 的改进，Java 7 使用了一个`LinkedList`作为内部桶结构。一般来说，哈希代码冲突很少发生。所以我们可以把元素查找的复杂度看作是`O(1)`。

## 3。`ArrayList.c` `**ontains()**`

在内部， **`ArrayList`使用`indexOf(object)`方法检查对象是否在列表**中。`indexOf(object)`方法迭代整个数组，并将每个元素与`equals(object)`方法进行比较。

回到复杂性分析，即`ArrayList`。`contains()`方法需要`O(n)`时间。**所以我们在这里寻找一个特定对象所花费的时间取决于数组中的项目数量。**

## 4。基准测试

现在，让我们用性能基准测试来预热 JVM。我们将使用 JMH 的 OpenJDK 产品。要了解更多关于设置和执行的信息，请查看我们的[实用指南](/web/20221208143940/https://www.baeldung.com/java-microbenchmark-harness)。

首先，让我们创建一个简单的`CollectionsBenchmark`类:

```java
@BenchmarkMode(Mode.AverageTime)
@OutputTimeUnit(TimeUnit.NANOSECONDS)
@Warmup(iterations = 5)
public class CollectionsBenchmark {

    @State(Scope.Thread)
    public static class MyState {
        private Set<Employee> employeeSet = new HashSet<>();
        private List<Employee> employeeList = new ArrayList<>();

        private long iterations = 1000;

        private Employee employee = new Employee(100L, "Harry");

        @Setup(Level.Trial)
        public void setUp() {

            for (long i = 0; i < iterations; i++) {
                employeeSet.add(new Employee(i, "John"));
                employeeList.add(new Employee(i, "John"));
            }

            employeeList.add(employee);
            employeeSet.add(employee);
        }
    }
}
```

在这里，我们创建并初始化`Employee`对象的`HashSet`和`ArrayList`:

```java
public class Employee {

    private Long id;
    private String name;

    // constructor and getter setters go here
}
```

我们将`employee = new Employee(100L, “Harry”) `实例作为最后的元素添加到两个集合中。所以我们测试了 `employee`对象在最坏情况下的查找时间。

`@OutputTimeUnit(TimeUnit.NANOSECONDS)`表示我们想要以纳秒为单位的结果。在我们的例子中，默认的`@Warmup`迭代次数是 5。`@BenchmarkMode`被设置为`Mode.AverageTime`，这意味着我们对计算平均运行时间感兴趣。对于第一次执行，我们将`iterations = 1000`项放入我们的集合中。

之后，我们将基准方法添加到`CollectionsBenchmark`类中:

```java
@Benchmark
public boolean testArrayList(MyState state) {
    return state.employeeList.contains(state.employee);
}
```

这里我们检查`employeeList`是否包含`employee`对象。

同样，我们对`employeeSet`进行了熟悉的测试:

```java
@Benchmark
public boolean testHashSet(MyState state) {
    return state.employeeSet.contains(state.employee);
}
```

最后，我们可以运行测试:

```java
public static void main(String[] args) throws Exception {
    Options options = new OptionsBuilder()
      .include(CollectionsBenchmark.class.getSimpleName())
      .forks(1).build();
    new Runner(options).run();
}
```

结果如下:

```java
Benchmark                           Mode  Cnt     Score     Error  Units
CollectionsBenchmark.testArrayList  avgt   20  4035.646 ± 598.541  ns/op
CollectionsBenchmark.testHashSet    avgt   20     9.456 ±   0.729  ns/op
```

**我们可以清楚地看到，`testArrayList`方法的平均查找分数为`4035.646 ns`，而`testHashSet`的平均查找分数比`9.456 ns`快。**

现在，让我们增加测试中的元素数量，并运行迭代次数= 10.000 的测试:

```java
Benchmark                           Mode  Cnt      Score       Error  Units
CollectionsBenchmark.testArrayList  avgt   20  57499.620 ± 11388.645  ns/op
CollectionsBenchmark.testHashSet    avgt   20     11.802 ±     1.164  ns/op
```

这里，`HashSet`中的`contains()`也比`ArrayList`有巨大的性能优势。

## 5。结论

这篇简短的文章解释了`HashSet`和`ArrayList` 系列的`contains()`方法的性能。在 JMH 基准测试的帮助下，我们展示了`contains()`在每种产品系列中的表现。

作为结论，我们可以了解到，**`contains()`方法在`HashSet`中比`ArrayList`更快。**

和往常一样，本文的完整代码在 GitHub 项目的[上。](https://web.archive.org/web/20221208143940/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-collections-3)