# java.util.Arrays 类指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-util-arrays>

## 1。简介

在本教程中，我们将看看`java.util.Arrays`，一个从 Java 1.2 开始就成为 Java 一部分的实用程序类。

使用`Arrays, `我们可以创建、比较、排序、搜索、流式传输和转换数组。

## 2。创建

让我们来看看创建数组的一些方法:`copyOf`、`copyOfRange`和`fill.`

### 2.1。**`copyOf``copyOfRange`**

要使用`copyOfRange`，我们需要我们的原始数组以及我们想要复制的开始索引(包含)和结束索引(不包含):

```java
String[] intro = new String[] { "once", "upon", "a", "time" };
String[] abridgement = Arrays.copyOfRange(storyIntro, 0, 3); 

assertArrayEquals(new String[] { "once", "upon", "a" }, abridgement); 
assertFalse(Arrays.equals(intro, abridgement));
```

为了使用`copyOf`，我们采用 *intro* 和一个目标数组大小，我们将得到一个该长度的新数组:

```java
String[] revised = Arrays.copyOf(intro, 3);
String[] expanded = Arrays.copyOf(intro, 5);

assertArrayEquals(Arrays.copyOfRange(intro, 0, 3), revised);
assertNull(expanded[4]);
```

注意，如果我们的目标尺寸比原始尺寸大，那么 **`copyOf `用`null` s 填充数组。**

### 2.2。`fill`

另一种方法，我们可以创建一个固定长度的数组，就是`fill, `，当我们想要一个所有元素都相同的数组时，这很有用:

```java
String[] stutter = new String[3];
Arrays.fill(stutter, "once");

assertTrue(Stream.of(stutter)
  .allMatch(el -> "once".equals(el));
```

**检查`setAll `以创建一个元素不同的数组。**

注意，我们需要自己预先实例化数组——而不是像【T0，3】`;`这样的东西——因为这个特性是在语言中的泛型出现之前引入的。

## 3。比较

现在让我们切换到比较数组的方法。

### 3.1。`equals`和`deepEquals`

我们可以使用`equals`通过大小和内容进行简单的数组比较。如果我们添加一个 null 作为元素之一，内容检查将失败:

```java
assertTrue(
  Arrays.equals(new String[] { "once", "upon", "a", "time" }, intro));
assertFalse(
  Arrays.equals(new String[] { "once", "upon", "a", null }, intro));
```

当我们有嵌套或多维数组时，我们不仅可以使用`deepEquals` 来检查顶级元素，还可以递归地执行检查:

```java
Object[] story = new Object[] 
  { intro, new String[] { "chapter one", "chapter two" }, end };
Object[] copy = new Object[] 
  { intro, new String[] { "chapter one", "chapter two" }, end };

assertTrue(Arrays.deepEquals(story, copy));
assertFalse(Arrays.equals(story, copy));
```

注意`deepE` `quals`是如何通过的，而`equals `却没有通过`.`

**这是因为`deepEquals `每次遇到数组**时最终都会调用自己，而`equals `只会比较子数组的引用。

另外，这使得用自引用调用数组变得很危险！

### 3.2。`hashCode` 和`deepHashCode`

`hashCode`的实现将给我们推荐给 Java 对象的`equals` / `hashCode`契约的另一部分。我们使用`hashCode` 根据数组的内容计算一个整数:

```java
Object[] looping = new Object[]{ intro, intro }; 
int hashBefore = Arrays.hashCode(looping);
int deepHashBefore = Arrays.deepHashCode(looping);
```

现在，我们将原始数组的一个元素设置为 null，并重新计算哈希值:

```java
intro[3] = null;
int hashAfter = Arrays.hashCode(looping); 
```

或者，`deepHashCode`检查嵌套数组中元素和内容的数量是否匹配。如果我们用`deepHashCode`重新计算:

```java
int deepHashAfter = Arrays.deepHashCode(looping);
```

现在，我们可以看到这两种方法的区别:

```java
assertEquals(hashAfter, hashBefore);
assertNotEquals(deepHashAfter, deepHashBefore); 
```

**`deepHashCode`是当我们在数组**上处理像`HashMap`和`HashSet` 这样的数据结构时使用的底层计算。

## 4。分类和搜索

接下来，让我们看看排序和搜索数组。

### 4.1。`sort`

如果我们的元素是原语或者它们实现了`Comparable`，我们可以使用`sort `来执行内嵌排序:

```java
String[] sorted = Arrays.copyOf(intro, 4);
Arrays.sort(sorted);

assertArrayEquals(
  new String[]{ "a", "once", "time", "upon" }, 
  sorted);
```

**注意`sort`改变了原始引用**，这就是我们在这里执行复制的原因。

*sort* 将对不同的数组元素类型使用不同的算法。[原始类型使用双支点快速排序](https://web.archive.org/web/20220525140934/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/Arrays.html#sort(byte%5B%5D))，而[对象类型使用 Timsort](https://web.archive.org/web/20220525140934/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/Arrays.html#sort(java.lang.Object%5B%5D)) 。对于随机排序的数组，两者都有平均的情况。

从 Java 8 开始，`parallelSort `可用于并行排序-合并。它提供了一种使用几个`Arrays.sort `任务的并发排序方法。

### 4.2。`binarySearch`

在一个未排序的数组中搜索是线性的，但是如果我们有一个已排序的数组，那么我们可以在`O(log n)`中做，这就是我们可以用`binarySearch:`做的

```java
int exact = Arrays.binarySearch(sorted, "time");
int caseInsensitive = Arrays.binarySearch(sorted, "TiMe", String::compareToIgnoreCase);

assertEquals("time", sorted[exact]);
assertEquals(2, exact);
assertEquals(exact, caseInsensitive);
```

如果我们不提供一个`Comparator`作为第三个参数，那么`binarySearch`会认为我们的元素类型是`Comparable`类型。

再一次，注意**如果我们的数组没有先排序，那么`binarySearch `不会像我们期望的那样工作！**

## 5。流媒体

正如我们前面看到的，`Arrays `在 Java 8 中被更新，以包括使用流 API 的方法，如`parallelSort`(上面提到的)、`stream `和`setAll.`

### 5.1。`stream`

`stream`让我们能够完全访问阵列的流 API:

```java
Assert.assertEquals(Arrays.stream(intro).count(), 4);

exception.expect(ArrayIndexOutOfBoundsException.class);
Arrays.stream(intro, 2, 1).count();
```

我们可以为流提供包含性和排他性的索引，但是如果这些索引是无序的、负的或超出范围的，我们应该期待一个`ArrayIndexOutOfBoundsException`。

## 6。转换

最后，`toString,` `asList,`和`setAll `给了我们几种不同的方法来转换数组。

### 6.1。**`toString``deepToString`**

我们获得原始数组的可读版本的一个好方法是使用`toString:`

```java
assertEquals("[once, upon, a, time]", Arrays.toString(storyIntro)); 
```

同样**我们必须使用深度版本来打印嵌套数组的内容**:

```java
assertEquals(
  "[[once, upon, a, time], [chapter one, chapter two], [the, end]]",
  Arrays.deepToString(story));
```

### 6.2。`asList`

在所有的`Arrays`方法中，最方便我们使用的是`asList.`,我们有一个简单的方法将数组转换成列表:

```java
List<String> rets = Arrays.asList(storyIntro);

assertTrue(rets.contains("upon"));
assertTrue(rets.contains("time"));
assertEquals(rets.size(), 4);
```

然而，**返回的`List`将是固定长度的，所以我们不能添加或删除元素**。

奇怪的是，还要注意， **`java.util.Arrays`有它自己的`ArrayList`子类，该子类由`asList `返回**。这在调试时可能很有欺骗性！

### 6.3。`setAll`

使用`setAll`，我们可以用一个函数接口设置一个数组的所有元素。生成器实现将位置索引作为参数:

```java
String[] longAgo = new String[4];
Arrays.setAll(longAgo, i -> this.getWord(i)); 
assertArrayEquals(longAgo, new String[]{"a","long","time","ago"});
```

当然，异常处理是使用 lambdas 的一个比较危险的部分。所以记住这里，**如果 lambda 抛出一个异常，那么 Java 不会定义数组的最终状态。**

## 7.平行前缀

从 Java 8 开始引入的`Arrays`中的另一个新方法是`parallelPrefix`。使用`parallelPrefix`，我们可以以累积的方式对输入数组的每个元素进行操作。

### 7.1。`parallelPrefix`

如果运算符执行加法，如下例所示，`[1, 2, 3, 4]` 将导致`[1, 3, 6, 10]:`

```java
int[] arr = new int[] { 1, 2, 3, 4};
Arrays.parallelPrefix(arr, (left, right) -> left + right);
assertThat(arr, is(new int[] { 1, 3, 6, 10}));
```

此外，我们可以为操作指定一个子范围:

```java
int[] arri = new int[] { 1, 2, 3, 4, 5 };
Arrays.parallelPrefix(arri, 1, 4, (left, right) -> left + right);
assertThat(arri, is(new int[] { 1, 2, 5, 9, 5 }));
```

注意，该方法是并行执行的，因此**累积操作应该是无副作用的，[关联](https://web.archive.org/web/20220525140934/https://en.wikipedia.org/wiki/Associative_property)** 。

对于非关联函数:

```java
int nonassociativeFunc(int left, int right) {
    return left + right*left;
}
```

使用`parallelPrefix`会产生不一致的结果:

```java
@Test
public void whenPrefixNonAssociative_thenError() {
    boolean consistent = true;
    Random r = new Random();
    for (int k = 0; k < 100_000; k++) {
        int[] arrA = r.ints(100, 1, 5).toArray();
        int[] arrB = Arrays.copyOf(arrA, arrA.length);

        Arrays.parallelPrefix(arrA, this::nonassociativeFunc);

        for (int i = 1; i < arrB.length; i++) {
            arrB[i] = nonassociativeFunc(arrB[i - 1], arrB[i]);
        }

        consistent = Arrays.equals(arrA, arrB);
        if(!consistent) break;
    }
    assertFalse(consistent);
}
```

### 7.2。性能

并行前缀计算通常比顺序循环更有效，尤其是对于大型数组。在采用 [JMH](/web/20220525140934/https://www.baeldung.com/java-microbenchmark-harness) 的英特尔至强处理器(6 核)上运行微基准测试时，我们可以看到巨大的性能提升:

```java
Benchmark                      Mode        Cnt       Score   Error        Units
largeArrayLoopSum             thrpt         5        9.428 ± 0.075        ops/s
largeArrayParallelPrefixSum   thrpt         5       15.235 ± 0.075        ops/s

Benchmark                     Mode         Cnt       Score   Error        Units
largeArrayLoopSum             avgt          5      105.825 ± 0.846        ops/s
largeArrayParallelPrefixSum   avgt          5       65.676 ± 0.828        ops/s
```

下面是基准代码:

```java
@Benchmark
public void largeArrayLoopSum(BigArray bigArray, Blackhole blackhole) {
  for (int i = 0; i < ARRAY_SIZE - 1; i++) {
    bigArray.data[i + 1] += bigArray.data[i];
  }
  blackhole.consume(bigArray.data);
}

@Benchmark
public void largeArrayParallelPrefixSum(BigArray bigArray, Blackhole blackhole) {
  Arrays.parallelPrefix(bigArray.data, (left, right) -> left + right);
  blackhole.consume(bigArray.data);
}
```

## 7。结论

在本文中，我们学习了如何使用`java.util.Arrays`类创建、搜索、排序和转换数组。

这个类在最近的 Java 版本中得到了扩展，在 [Java 8](https://web.archive.org/web/20220525140934/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/Arrays.html) 中包含了流产生和消费方法，在 [Java 9](https://web.archive.org/web/20220525140934/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/Arrays.html) 中包含了不匹配方法。

一如既往，这篇文章的来源是 Github 上的[。](https://web.archive.org/web/20220525140934/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-arrays-guides)