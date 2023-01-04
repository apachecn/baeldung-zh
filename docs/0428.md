# Java 可选 orelse()与 Oregon()方法

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-optional-or-else-vs-or-else-get>

## 1.介绍

`Optional`的 API 通常有两个会引起混淆的方法:`orElse() `和`orElseGet()`。

在这个快速教程中，我们将看看这两者之间的区别，并探索何时使用每一个。

## 2.签名

首先，让我们从基础开始，看看他们的签名:

```
public T orElse(T other)

public T orElseGet(Supplier<? extends T> other)
```

显然，`orElse() `接受类型`T,` 的任何参数，而`orElseGet() `接受类型`Supplier` 的函数接口，返回类型`T`的对象。

基于他们的[javadoc](https://web.archive.org/web/20220823125659/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/Optional.html#orElse(T)):

*   `orElse()`:如果存在则返回值，否则返回`other`
*   如果存在，则`orElseGet():`返回该值，否则调用`other `并返回其调用的结果

## 3.差异

很容易被这些简化的定义搞得有点晕头转向，所以我们再深入一点，看看一些实际的使用场景。

### 3.1.`orElse()`

假设我们已经正确配置了[记录器](/web/20220823125659/https://www.baeldung.com/java-logging-intro)，让我们从编写一段简单的代码开始:

```
String name = Optional.of("baeldung")
  .orElse(getRandomName());
```

注意，`getRandomName() `是一个从`names:`的`List<String>`返回随机`name `的方法

```
public String getRandomName() {
    LOG.info("getRandomName() method - start");

    Random random = new Random();
    int index = random.nextInt(5);

    LOG.info("getRandomName() method - end");
    return names.get(index);
}
```

在执行我们的代码时，我们会在控制台中看到以下消息:

```
getRandomName() method - start
getRandomName() method - end
```

变量`name `将在代码执行结束时保存` “baeldung” `。

有了它，我们可以很容易地推断出`orElse()` 的**参数被求值，即使有一个非空的`Optional`。**

### 3.2.`orElseGet()`

现在让我们尝试使用`orElseGet()`编写类似的代码:

```
String name = Optional.of("baeldung")
  .orElseGet(() -> getRandomName());
```

上面的代码不会调用`getRandomName() `方法。

**记住(来自 Javadoc)作为参数传递的 S `upplier `方法只在`an Optional `值不存在时执行。**

因此，在我们的例子中使用`orElseGet() `将节省我们计算随机`name`的时间。

## 4.衡量绩效影响

现在，为了了解性能上的差异，让我们使用 [JMH](/web/20220823125659/https://www.baeldung.com/java-microbenchmark-harness) 来看一些实际数字:

```
@Benchmark
@BenchmarkMode(Mode.AverageTime)
public String orElseBenchmark() {
    return Optional.of("baeldung").orElse(getRandomName());
}
```

和`orElseGet()`:

```
@Benchmark
@BenchmarkMode(Mode.AverageTime)
public String orElseGetBenchmark() {
    return Optional.of("baeldung").orElseGet(() -> getRandomName());
}
```

在执行我们的基准方法时，我们得到:

```
Benchmark           Mode  Cnt      Score       Error  Units
orElseBenchmark     avgt   20  60934.425 ± 15115.599  ns/op
orElseGetBenchmark  avgt   20      3.798 ±     0.030  ns/op
```

正如我们所看到的，性能影响可能是巨大的，即使对于这样一个简单的用例场景。

以上数字可能略有不同；然而，对于我们的特殊例子来说， **`orElseGet()` 的表现明显优于`orElse() `。**

毕竟，`orElse() `涉及到每次运行的`getRandomName() `方法的计算。

## 5.什么重要？

除了性能方面，其他值得考虑的因素包括:

*   如果这个方法会执行一些额外的逻辑会怎么样呢？例如，进行一些数据库插入或更新
*   即使我们将一个对象赋给了`orElse() `参数，我们仍然会无缘无故地创建`“Other”` 对象`:`

    ```
    String name = Optional.of("baeldung").orElse("Other")
    ```

这就是为什么根据我们的需求在`orElse() `和`orElseGet() `之间做出谨慎的决定对我们来说很重要。**默认情况下，每次都使用`orElseGet() `更有意义，除非默认对象已经构造好并可以直接访问。**

## 6.结论

在本文中，我们学习了`Optional orElse() `和`OrElseGet() `方法之间的细微差别。我们还讨论了这样简单的概念有时会有更深的含义。

和往常一样，完整的源代码可以在 Github 上找到[。](https://web.archive.org/web/20220823125659/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-optional)