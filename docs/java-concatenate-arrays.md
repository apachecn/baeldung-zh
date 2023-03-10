# 在 Java 中连接两个数组

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-concatenate-arrays>

## 1.概观

在本教程中，我们将讨论如何在 Java 中连接两个[数组](/web/20220724034804/https://www.baeldung.com/java-common-array-operations)。

首先，我们将使用标准的 Java API 实现我们自己的方法。

然后，我们将看看如何使用常用的库来解决这个问题。

## 2.问题简介

快速的例子可以清楚地解释这个问题。

假设我们有两个数组:

```java
String[] strArray1 = {"element 1", "element 2", "element 3"};
String[] strArray2 = {"element 4", "element 5"};
```

现在，我们想加入他们并获得一个新阵列:

```java
String[] expectedStringArray = {"element 1", "element 2", "element 3", "element 4", "element 5"}
```

此外，我们不希望我们的方法只适用于`String`数组，所以**我们将寻找一个通用的解决方案**。

此外，我们不应该忘记原始数组的情况。如果我们的解决方案也适用于原始数组，那就太好了:

```java
int[] intArray1 = { 0, 1, 2, 3 };
int[] intArray2 = { 4, 5, 6, 7 };
int[] expectedIntArray = { 0, 1, 2, 3, 4, 5, 6, 7 }; 
```

在本教程中，我们将提出不同的方法来解决这个问题。

## 3.使用 Java `Collections`

当我们研究这个问题时，可能会有一个快速的解决方法。

嗯，Java 没有提供连接数组的帮助方法。但是，从 Java 5 开始，`Collections`实用程序类引入了一个 [`addAll(Collection<? super T> c, T… elements)`](https://web.archive.org/web/20220724034804/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/Collections.html#addAll(java.util.Collection,T...)) 方法。

我们可以创建一个`List`对象，然后调用这个方法两次，将两个数组添加到列表中。最后，我们将得到的`List` 转换回一个数组:

```java
static <T> T[] concatWithCollection(T[] array1, T[] array2) {
    List<T> resultList = new ArrayList<>(array1.length + array2.length);
    Collections.addAll(resultList, array1);
    Collections.addAll(resultList, array2);

    @SuppressWarnings("unchecked")
    //the type cast is safe as the array1 has the type T[]
    T[] resultArray = (T[]) Array.newInstance(array1.getClass().getComponentType(), 0);
    return resultList.toArray(resultArray);
} 
```

在上面的方法中，我们使用 Java 反射 API 来[创建一个通用数组](/web/20220724034804/https://www.baeldung.com/java-generic-array)实例:`resultArray.`

让我们编写一个测试来验证我们的方法是否有效:

```java
@Test
public void givenTwoStringArrays_whenConcatWithList_thenGetExpectedResult() {
    String[] result = ArrayConcatUtil.concatWithCollection(strArray1, strArray2);
    assertThat(result).isEqualTo(expectedStringArray);
} 
```

如果我们进行测试，就会通过。

这种方法非常简单。然而，**因为该方法接受`T[]`数组，所以它不支持连接原始数组**。

除此之外，**的效率很低，因为它创建了一个`ArrayList`对象，稍后我们调用`toArray()`方法将其转换回数组**。在这个过程中，Java `List`对象增加了不必要的开销。

接下来，我们来看看能否找到更高效的解决问题的方法。

## 4.使用数组复制技术

Java 没有提供数组拼接方法，但是提供了两种数组复制方法:`[System.arraycopy()](/web/20220724034804/https://www.baeldung.com/java-array-copy#the-system-class)`和 [`Arrays.copyOf()`](/web/20220724034804/https://www.baeldung.com/java-array-copy#the-arrays-class) 。

我们可以使用 Java 的数组复制方法来解决这个问题。

想法是，我们创建一个新的数组，比如说`result`，它有`result`。`length = array1.length + array2.length`，并将每个数组的元素复制到`result`数组中。

### 4.1.非原始数组

首先，让我们看看方法的实现:

```java
static <T> T[] concatWithArrayCopy(T[] array1, T[] array2) {
    T[] result = Arrays.copyOf(array1, array1.length + array2.length);
    System.arraycopy(array2, 0, result, array1.length, array2.length);
    return result;
} 
```

这个方法看起来很简洁。此外，整个方法只创建了一个新的数组对象:`result`。

现在，让我们编写一个测试方法来检查它是否如我们预期的那样工作:

```java
@Test
public void givenTwoStringArrays_whenConcatWithCopy_thenGetExpectedResult() {
    String[] result = ArrayConcatUtil.concatWithArrayCopy(strArray1, strArray2);
    assertThat(result).isEqualTo(expectedStringArray);
} 
```

如果我们试一试，考试就会通过。

没有不必要的对象创建。因此，**这种方法比使用 Java `Collections`** `.`的方法更高效

另一方面，这个泛型方法只接受类型为`T[]`的参数。因此，我们不能将基元数组传递给方法。

但是，我们可以修改该方法，使其支持基元数组。

接下来，让我们仔细看看如何添加基本数组支持。

### 4.2.添加原始数组支持

为了使该方法支持原始数组，我们需要将参数的类型从`T[]`更改为`T` ，并进行一些类型安全检查。

首先，让我们来看看修改后的方法:

```java
static <T> T concatWithCopy2(T array1, T array2) {
    if (!array1.getClass().isArray() || !array2.getClass().isArray()) {
        throw new IllegalArgumentException("Only arrays are accepted.");
    }

    Class<?> compType1 = array1.getClass().getComponentType();
    Class<?> compType2 = array2.getClass().getComponentType();

    if (!compType1.equals(compType2)) {
        throw new IllegalArgumentException("Two arrays have different types.");
    }

    int len1 = Array.getLength(array1);
    int len2 = Array.getLength(array2);

    @SuppressWarnings("unchecked")
    //the cast is safe due to the previous checks
    T result = (T) Array.newInstance(compType1, len1 + len2);

    System.arraycopy(array1, 0, result, 0, len1);
    System.arraycopy(array2, 0, result, len1, len2);

    return result;
} 
```

显然，`concatWithCopy2()`方法比原始版本长。但也不难理解。现在，让我们快速浏览一下，了解它是如何工作的。

由于该方法现在允许类型为`T`的参数，我们需要**确保两个参数都是数组**:

```java
if (!array1.getClass().isArray() || !array2.getClass().isArray()) {
    throw new IllegalArgumentException("Only arrays are accepted.");
}
```

如果两个参数是数组，还是不够安全。例如，我们不想连接一个`Integer[]`数组和一个`String[]`数组。所以，我们需要**确保两个数组的`ComponentType`是相同的**:

```java
if (!compType1.equals(compType2)) {
    throw new IllegalArgumentException("Two arrays have different types.");
}
```

在类型安全检查之后，我们可以使用`ConponentType`对象创建一个通用数组实例，并将参数数组复制到`result`数组中。这和以前的`concatWithCopy()`方法很相似。

### 4.3.测试`concatWithCopy2()`方法

接下来，让我们测试我们的新方法是否如我们预期的那样工作。首先，我们传递两个非数组对象，并查看该方法是否会引发预期的异常:

```java
@Test
public void givenTwoStrings_whenConcatWithCopy2_thenGetException() {
    String exMsg = "Only arrays are accepted.";
    try {
        ArrayConcatUtil.concatWithCopy2("String Nr. 1", "String Nr. 2");
        fail(String.format("IllegalArgumentException with message:'%s' should be thrown. But it didn't", exMsg));
    } catch (IllegalArgumentException e) {
        assertThat(e).hasMessage(exMsg);
    }
} 
```

在上面的测试中，我们向方法传递了两个`String`对象。如果我们执行测试，它就通过了。这意味着我们已经得到了预期的异常。

最后，让我们构建一个测试来检查新方法是否可以连接基本数组:

```java
@Test
public void givenTwoArrays_whenConcatWithCopy2_thenGetExpectedResult() {
    String[] result = ArrayConcatUtil.concatWithCopy2(strArray1, strArray2);
    assertThat(result).isEqualTo(expectedStringArray);

    int[] intResult = ArrayConcatUtil.concatWithCopy2(intArray1, intArray2);
    assertThat(intResult).isEqualTo(expectedIntArray);
} 
```

这次，我们调用了两次`concatWithCopy2()`方法。首先，我们传递两个`String[]`数组。然后，我们传递两个`int[]`原始数组。

如果我们运行它，测试将通过。现在，我们可以说，`concatWithCopy2()`方法如我们预期的那样工作。

## 5.使用 Java 流 API

如果我们正在使用的 Java 版本是 8 或更新版本，那么[流 API](/web/20220724034804/https://www.baeldung.com/java-8-streams) 是可用的。我们也可以使用流 API 来解决这个问题。

首先，我们可以通过 [`Arrays.stream()`](https://web.archive.org/web/20220724034804/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/Arrays.html#stream(T%5B%5D)) 方法从数组中获取一个`Stream`。另外，`Stream`类提供了一个静态的 [`concat()`](https://web.archive.org/web/20220724034804/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/stream/Stream.html#concat(java.util.stream.Stream,java.util.stream.Stream)) 方法来连接两个`Stream`对象。

现在，让我们看看如何用`Stream.`连接两个数组

### 5.1.连接非基元数组

使用 Java 流构建通用解决方案非常简单:

```java
static <T> T[] concatWithStream(T[] array1, T[] array2) {
    return Stream.concat(Arrays.stream(array1), Arrays.stream(array2))
      .toArray(size -> (T[]) Array.newInstance(array1.getClass().getComponentType(), size));
} 
```

首先，我们将两个输入数组转换成`Stream`对象。其次，我们使用`Stream.concat()`方法连接两个`Stream`对象。

最后，我们返回一个数组，其中包含连接在一起的`Stream.`中的所有元素

接下来，让我们构建一个简单的测试方法来检查解决方案是否有效:

```java
@Test
public void givenTwoStringArrays_whenConcatWithStream_thenGetExpectedResult() {
    String[] result = ArrayConcatUtil.concatWithStream(strArray1, strArray2);
    assertThat(result).isEqualTo(expectedStringArray);
} 
```

如果我们通过两个`String[]`数组，测试就会通过。

很可能，我们已经注意到我们的泛型方法接受`T[]`类型的参数。因此，**对原始数组**不起作用。

接下来，让我们看看如何使用 Java 流连接两个基本数组。

### 5.2.连接原始数组

Stream API 提供了不同的`Stream`类，可以将`Stream`对象转换为相应的原始数组，比如 [`IntStream,`](https://web.archive.org/web/20220724034804/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/stream/IntStream.html) `[LongStream](https://web.archive.org/web/20220724034804/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/stream/LongStream.html),`和 [`DoubleStream`](https://web.archive.org/web/20220724034804/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/stream/DoubleStream.html) 。

然而**只有`int`、`long`、`double`有它们的`Stream`类型**。也就是说，如果我们想要连接的原始数组具有类型`int[]`、`long[]`或`double[]`，我们可以选择正确的`Stream`类并调用`concat()`方法。

让我们看一个使用`IntStream`连接两个 `int[]`数组的例子:

```java
static int[] concatIntArraysWithIntStream(int[] array1, int[] array2) {
    return IntStream.concat(Arrays.stream(array1), Arrays.stream(array2)).toArray();
} 
```

如上面的方法所示，`Arrays.stream(int[])`方法将返回一个`IntStream`对象。

同样，`IntStream.toArray()`方法返回`int[]`。因此，我们不需要关心类型转换。

像往常一样，让我们创建一个测试，看看它是否适用于我们的`int[]`输入数据:

```java
@Test
public void givenTwoIntArrays_whenConcatWithIntStream_thenGetExpectedResult() {
    int[] intResult = ArrayConcatUtil.concatIntArraysWithIntStream(intArray1, intArray2);
    assertThat(intResult).isEqualTo(expectedIntArray);
} 
```

如果我们进行测试，它会通过的。

## 6.使用 Apache Commons Lang 库

Apache Commons Lang 库在现实世界中的 Java 应用程序中被广泛使用。

它附带了一个 [`ArrayUtils`](/web/20220724034804/https://www.baeldung.com/array-processing-commons-lang) 类，其中包含许多方便的数组助手方法。

**`ArrayUtils`类提供了一系列的 [`addAll()`](https://web.archive.org/web/20220724034804/https://commons.apache.org/proper/commons-lang/javadocs/api-release/org/apache/commons/lang3/ArrayUtils.html#add-T:A-T-) 方法，支持连接非原语和原语数组。**

让我们用一个测试方法来验证一下:

```java
@Test
public void givenTwoArrays_whenConcatWithCommonsLang_thenGetExpectedResult() {
    String[] result = ArrayUtils.addAll(strArray1, strArray2);
    assertThat(result).isEqualTo(expectedStringArray);

    int[] intResult = ArrayUtils.addAll(intArray1, intArray2);
    assertThat(intResult).isEqualTo(expectedIntArray);
} 
```

在内部，`ArrayUtils.addAll()`方法使用 performant `System.arraycopy()`方法进行数组连接。

## 7.使用番石榴图书馆

与 Apache Commons 库类似， [Guava](/web/20220724034804/https://www.baeldung.com/guava-guide) 是另一个受到许多开发人员喜爱的库。

Guava 还提供了方便的助手类来进行数组连接。

如果我们想要连接非原始数组，`[ObjectArrays.concat()](https://web.archive.org/web/20220724034804/https://guava.dev/releases/19.0/api/docs/com/google/common/collect/ObjectArrays.html#concat(T[],%20T[],%20java.lang.Class))`方法是一个很好的选择:

```java
@Test
public void givenTwoStringArrays_whenConcatWithGuava_thenGetExpectedResult() {
    String[] result = ObjectArrays.concat(strArray1, strArray2, String.class);
    assertThat(result).isEqualTo(expectedStringArray);
} 
```

番石榴为每个原始部落提供了[原始部落的工具](https://web.archive.org/web/20220724034804/https://github.com/google/guava/wiki/PrimitivesExplained)。所有原始实用程序都提供了一个`concat() `方法来将数组与相应的类型连接起来，例如:

*   `int[]`–番石榴:`Ints.concat(int[] … arrays)`
*   `long[]`–番石榴:`Longs.concat(long[] … arrays)`
*   `byte[]`–番石榴:`Bytes.concat(byte[] … arrays)`
*   `double[]`–番石榴:`Doubles.concat(double[] … arrays)`

我们可以选择正确的原始实用程序类来连接原始数组。

接下来，让我们使用`Ints.concat()`方法连接两个`int[]`数组:

```java
@Test
public void givenTwoIntArrays_whenConcatWithGuava_thenGetExpectedResult() {
    int[] intResult = Ints.concat(intArray1, intArray2);
    assertThat(intResult).isEqualTo(expectedIntArray);
}
```

同样，Guava 在内部使用上述方法中的`System.arraycopy()`来进行数组串联，以获得良好的性能。

## 8.结论

在本文中，我们通过例子介绍了在 Java 中连接两个数组的不同方法。

和往常一样，本文附带的完整代码示例可以在 GitHub 上获得[。](https://web.archive.org/web/20220724034804/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-arrays-operations-advanced)