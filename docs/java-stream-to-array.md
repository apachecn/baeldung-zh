# Java 中流和数组之间的转换

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-stream-to-array>

## 1。简介

通常需要[将](/web/20221205155208/https://www.baeldung.com/convert-map-values-to-array-list-set)各种[动态数据结构](/web/20221205155208/https://www.baeldung.com/convert-array-to-list-and-list-to-array)转换成[数组](/web/20221205155208/https://www.baeldung.com/convert-array-to-set-and-set-to-array)。

在本教程中，我们将演示如何在 Java 中将 [`Stream`](/web/20221205155208/https://www.baeldung.com/java-8-streams-introduction) 转换为数组，反之亦然。

## 2.将`Stream`转换为数组

### 2.1.方法参考

**将`Stream`转换成数组的最好方法是使用`Stream'`的`toArray()`方法:**

```java
public String[] usingMethodReference(Stream<String> stringStream) {
    return stringStream.toArray(String[]::new);
}
```

现在，我们可以轻松测试转换是否成功:

```java
Stream<String> stringStream = Stream.of("baeldung", "convert", "to", "string", "array");
assertArrayEquals(new String[] { "baeldung", "convert", "to", "string", "array" },
    usingMethodReference(stringStream));
```

### 2.2.λ表达式

另一个等价方法是**将一个 lambda 表达式**传递给`toArray`()方法:

```java
public static String[] usingLambda(Stream<String> stringStream) {
    return stringStream.toArray(size -> new String[size]);
}
```

这将给出与使用方法引用相同的结果。

### 2.3.自定义类别

或者，我们可以全力以赴，打造一个全面发展的班级。

从[的`Stream` 文档](https://web.archive.org/web/20221205155208/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/stream/Stream.html#toArray(java.util.function.IntFunction))中我们可以看到，它以一个`IntFunction`作为参数。它将数组大小作为输入，并返回该大小的数组。

当然，`IntFunction `是一个接口，所以我们可以实现它:

```java
class MyArrayFunction implements IntFunction<String[]> {
    @Override
    public String[] apply(int size) {
        return new String[size];
    }
};
```

然后我们可以正常地构造和使用:

```java
public String[] usingCustomClass(Stream<String> stringStream) {
    return stringStream.toArray(new MyArrayFunction());
}
```

因此，我们可以作出与前面相同的断言。

### 2.4.原始数组

在前面的章节中，我们探讨了如何将一个`String Stream`转换成一个`String` 数组。事实上，我们可以对任何`Object`执行这种转换，它看起来与上面的`String`示例非常相似。

不过，对于原语来说，这有点不同。例如，如果我们有一个要转换成`int[]`的`Integer`的`Stream`，我们首先需要调用`mapToInt()`方法:

```java
public int[] intStreamToPrimitiveIntArray(Stream<Integer> integerStream) {
    return integerStream.mapToInt(i -> i).toArray();
}
```

还有`mapToLong()`和`mapToDouble()`方法供我们使用。另外，请注意，这次我们没有向`toArray()`传递任何参数。

最后，让我们做等式断言，并确认我们已经正确地得到了我们的`int`数组:

```java
Stream<Integer> integerStream = IntStream.rangeClosed(1, 7).boxed();
assertArrayEquals(new int[]{1, 2, 3, 4, 5, 6, 7}, intStreamToPrimitiveIntArray(integerStream));
```

但是，如果我们需要做相反的事情呢？让我们来看看。

## 3.将数组转换为`Stream`

当然，我们也可以走另一条路。Java 有一些专门的方法。

### 3.1.`Object`的数组

**我们可以使用`Arrays.stream()`或`Stream.of()`方法**将数组转换为`Stream`:

```java
public Stream<String> stringArrayToStreamUsingArraysStream(String[] stringArray) {
    return Arrays.stream(stringArray);
}

public Stream<String> stringArrayToStreamUsingStreamOf(String[] stringArray) {
    return Stream.of(stringArray);
}
```

**我们应该注意，在这两种情况下，我们的`Stream`和我们的数组是同一时间的。**

### 3.2.图元数组

类似地，我们可以转换基元数组:

```java
public IntStream primitiveIntArrayToStreamUsingArraysStream(int[] intArray) {
    return Arrays.stream(intArray);
}

public Stream<int[]> primitiveIntArrayToStreamUsingStreamOf(int[] intArray) {
    return Stream.of(intArray);
}
```

但是，与转换`Object` s 数组相比，有一个重要的区别。**转换图元数组时，`Arrays.stream()`返回`IntStream`，而`Stream.of()`返回`Stream<int[]>`。**

### 3.3.`Arrays.stream`对`Stream.of`

为了理解前面几节提到的区别，我们来看看相应方法的实现。

让我们先来看看 Java 对这两种方法的实现:

```java
public <T> Stream<T> stream(T[] array) {
    return stream(array, 0, array.length);
}

public <T> Stream<T> of(T... values) {
    return Arrays.stream(values);
}
```

我们可以看到,`Stream.of()`实际上在内部调用了`Arrays.stream()`,这显然是我们得到相同结果的原因。

现在，当我们想要转换一个基元数组时，我们将检查该案例中的方法:

```java
public IntStream stream(int[] array) {
    return stream(array, 0, array.length);
}

public <T> Stream<T> of(T t) {
    return StreamSupport.stream(new Streams.StreamBuilderImpl<>(t), false);
}
```

这一次，`Stream.of()`没有调用`Arrays.stream()`。

## 4.结论

在本文中，我们看到了如何在 Java 中将`Stream`转换成数组，反之亦然。我们也解释了为什么我们在转换一组`Object`和使用一组原语时会得到不同的结果。

和往常一样，完整的源代码可以在 GitHub 上找到[。](https://web.archive.org/web/20221205155208/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-arrays-convert)