# 如何计算数组列表中的重复元素

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-count-duplicate-elements-arraylist>

## 1.概观

在这个简短的教程中，我们将看看一些不同的方法来计算一个`ArrayList`中的重复元素。

## 2.用`Map.put()`循环

我们期望的结果是一个`Map` 对象，它包含来自输入列表的所有元素作为键，每个元素的计数作为值。

实现这一点的最直接的解决方案是遍历输入列表，对于每个元素:

*   如果`resultMap`包含元素，我们将计数器加 1
*   否则，我们`put`一个新的地图条目`(element, 1)` 到该地图

```
public <T> Map<T, Long> countByClassicalLoop(List<T> inputList) {
    Map<T, Long> resultMap = new HashMap<>();
    for (T element : inputList) {
        if (resultMap.containsKey(element)) {
            resultMap.put(element, resultMap.get(element) + 1L);
        } else {
            resultMap.put(element, 1L);
        }
    }
    return resultMap;
}
```

这个实现具有最好的兼容性，因为它适用于所有现代 Java 版本。

如果我们不需要 Java 8 之前的兼容性，我们可以进一步简化我们的方法:

```
public <T> Map<T, Long> countByForEachLoopWithGetOrDefault(List<T> inputList) {
    Map<T, Long> resultMap = new HashMap<>();
    inputList.forEach(e -> resultMap.put(e, resultMap.getOrDefault(e, 0L) + 1L));
    return resultMap;
}
```

接下来，让我们创建一个输入列表来测试这个方法:

```
private List<String> INPUT_LIST = Lists.list(
  "expect1",
  "expect2", "expect2",
  "expect3", "expect3", "expect3",
  "expect4", "expect4", "expect4", "expect4"); 
```

现在让我们来验证一下:

```
private void verifyResult(Map<String, Long> resultMap) {
    assertThat(resultMap)
      .isNotEmpty().hasSize(4)
      .containsExactly(
        entry("expect1", 1L),
        entry("expect2", 2L),
        entry("expect3", 3L),
        entry("expect4", 4L));
} 
```

我们将在剩下的方法中重用这个测试工具。

## 3.用`Map.compute()`循环

在 Java 8 中，方便的 [`compute()`](https://web.archive.org/web/20221127122833/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/Map.html#compute(K,java.util.function.BiFunction)) 方法被引入到了`Map`接口中。我们也可以利用这种方法:

```
public <T> Map<T, Long> countByForEachLoopWithMapCompute(List<T> inputList) {
    Map<T, Long> resultMap = new HashMap<>();
    inputList.forEach(e -> resultMap.compute(e, (k, v) -> v == null ? 1L : v + 1L));
    return resultMap;
}
```

注意`(k, v) -> v == null ? 1L : v + 1L`是实现`BiFunction<T, Long, Long>`接口的重映射函数。对于给定的键，它或者返回其当前值加 1(如果该键已经存在于映射中),或者返回默认值 1。

为了使代码更具可读性，**我们可以将重映射函数提取到它的变量中，甚至将它作为`countByForEachLoopWithMapCompute.`** 的输入参数

## 4.用`Map.merge()`循环

**当使用`Map.compute()`时，我们必须显式地处理`null`值——例如，如果给定键的映射不存在。**这就是为什么我们在重映射函数中实现了一个`null`检查。然而，这看起来并不漂亮。

让我们借助 [`Map.merge()`](https://web.archive.org/web/20221127122833/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/Map.html#merge(K,V,java.util.function.BiFunction)) 方法进一步清理我们的代码:

```
public <T> Map<T, Long> countByForEachLoopWithMapMerge(List<T> inputList) {
    Map<T, Long> resultMap = new HashMap<>();
    inputList.forEach(e -> resultMap.merge(e, 1L, Long::sum));
    return resultMap;
}
```

现在代码看起来干净简洁。

**让我们解释一下`merge()`是如何工作的。**如果给定键的映射不存在，或者它的值是`null`，它将该键与提供的值相关联。否则，它使用重新映射函数计算新值，并相应地更新映射。

注意，这次我们使用了`Long::sum`作为`BiFunction<T, Long, Long>`接口实现。

## 5 .蒸汽 api `Collectors.toMap()`

既然已经讲了 Java 8，就不能忘了强大的 Stream API。感谢 Stream API，我们可以用非常简洁的方式解决问题。

[`toMap()`](https://web.archive.org/web/20221127122833/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/stream/Collectors.html#toMap(java.util.function.Function,java.util.function.Function,java.util.function.BinaryOperator)) 收集器帮助我们将输入列表转换成一个`Map`:

```
public <T> Map<T, Long> countByStreamToMap(List<T> inputList) {
    return inputList.stream().collect(Collectors.toMap(Function.identity(), v -> 1L, Long::sum));
}
```

`toMap()` 是一个[方便的收集器](/web/20221127122833/https://www.baeldung.com/java-collectors-tomap)，它可以帮助我们将流转换成不同的`Map`实现。

## 6.流 API `Collectors.groupingBy()`和`Collectors.counting()`

除了`toMap()`之外，我们的问题可以由另外两个收藏家来解决， [`groupingBy()`](https://web.archive.org/web/20221127122833/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/stream/Collectors.html#groupingBy(java.util.function.Function,java.util.stream.Collector)) 和`[counting()](https://web.archive.org/web/20221127122833/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/stream/Collectors.html#counting())`:

```
public <T> Map<T, Long> countByStreamGroupBy(List<T> inputList) {
    return inputList.stream().collect(Collectors.groupingBy(k -> k, Collectors.counting()));
}
```

正确使用 [Java 8 收集器](/web/20221127122833/https://www.baeldung.com/java-8-collectors)使得我们的代码简洁易读。

## 7.结论

在这篇简短的文章中，我们展示了计算列表中重复元素数量的各种方法。

如果你想重温数组列表本身，你可以查看参考文章。

和往常一样，完整的源代码可以在 GitHub 上找到[。](https://web.archive.org/web/20221127122833/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-collections-list-3)