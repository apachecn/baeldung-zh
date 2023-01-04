# 在 Java 中切片数组

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-slicing-arrays>

## 1.概观

我们知道 Java 的`List`有 [`subList()`](https://web.archive.org/web/20221105192802/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/List.html#subList(int,int)) 方法，可以让我们对源`List`对象进行切片。然而，在阵列端没有标准的`subArray()`方法。

在本教程中，让我们探索如何在 Java 中获得给定数组的子数组。

## 2.问题简介

和往常一样，我们通过一个例子来理解问题。假设我们有一个字符串数组:

```java
String[] LANGUAGES = new String[] { "Python", "Java", "Kotlin", "Scala", "Ruby", "Go", "Rust" };
```

正如我们所见，`LANGUAGES`数组保存了一些编程语言名称。此外，由于用`“Java”, “Kotlin”,`或`“Scala”`编写的应用程序可以在 Java 虚拟机上运行，假设我们想要一个包含这三个元素的子数组。换句话说，**我们要从`LANGUAGES`数组:**中得到第二到第四个元素(索引`1`、`2`、`3`)

```java
String[] JVM_LANGUAGES = new String[] { "Java", "Kotlin", "Scala" };
```

在本教程中，我们将提出解决这个问题的不同方法。此外，为了简单起见，我们将使用单元测试断言来验证每个解决方案是否按预期工作。

接下来，让我们看看他们的行动。

## 3.使用流 API

Java 8 带给我们的一个重要新特性是[流 API](/web/20221105192802/https://www.baeldung.com/java-8-streams) 。因此，如果我们使用的 Java 版本是 8 或更高版本，我们可以使用 Stream API 分割给定的数组。

**首先，我们可以使用`Arrays.stream()`方法[将一个数组转换成一个`Stream`对象](/web/20221105192802/https://www.baeldung.com/java-stream-to-array#array-to-stream)。**我们要注意的是，我们要用 [`Arrays.stream()`](https://web.archive.org/web/20221105192802/https://docs.oracle.com/javase/8/docs/api/java/util/Arrays.html#stream-T:A-int-int-) 的方法带三个自变量:

*   `array`–在本例中，它是`LANGUAGES`
*   `startInclusive`–从上面的数组中提取的起始索引
*   `endExclusive`–要提取的结束索引，顾名思义，独占

因此，要解决我们的问题，我们可以将`LANGUAGES,` `1,`和`4`传递给`Arrays.stream()`方法。

接下来，让我们创建一个测试，看看它是否能得到我们想要的子数组:

```java
String[] result = Arrays.stream(LANGUAGES, 1, 4).toArray(String[]::new);
assertArrayEquals(JVM_LANGUAGES, result);
```

如上面的代码所示，在我们将数组转换为`Stream`后，我们可以调用 [`toArray()`](/web/20221105192802/https://www.baeldung.com/java-stream-to-array#1-method-reference) 方法将其转换回数组。

如果我们进行测试，就会通过。因此，它完成了工作。

## 4.使用`Arrays.copyOfRange()`方法

我们已经学会了使用流 API 来解决这个问题。但是，**Stream API 只有 Java 8 和更高版本**才有。

如果我们的 Java 版本是 6 或更高版本，我们可以使用`Arrays.copyOfRange()`方法解决这个问题。这个方法的参数类似于`Arrays.stream()`方法——数组、from-index(包含)和 to-index(不包含)。

接下来，让我们创建一个测试，看看`Arrays.copyOfRange()`是否能解决问题:

```java
String[] result = Arrays.copyOfRange(LANGUAGES, 1, 4);
assertArrayEquals(JVM_LANGUAGES, result);
```

如果我们试一试，测试就会通过。所以这也解决了我们的问题。

## 5.使用`System.arraycopy()`方法

`Arrays.copyOfRange()`方法通过将给定数组的一部分复制到一个新数组来解决这个问题。

当我们想从数组中复制一部分时，除了使用`Arrays.copyOfRange()`方法，我们还可以使用 [`System.arraycopy()`](https://web.archive.org/web/20221105192802/https://docs.oracle.com/javase/8/docs/api/java/lang/System.html#arraycopy-java.lang.Object-int-java.lang.Object-int-int-) 方法。那么接下来，让我们用这个方法来解决问题。

我们已经看到`Arrays.copyOfRange()`返回结果子数组。然而，**`System.arraycopy()`方法的返回类型是`void`** 。因此，我们必须创建一个新的数组对象，并将其传递给`arraycopy()`方法。方法填充数组中复制的元素:

```java
String[] result = new String[3];
System.arraycopy(LANGUAGES, 1, result, 0, 3);
assertArrayEquals(JVM_LANGUAGES, result);
```

如果我们运行它，测试就通过了。

正如我们在上面的代码中看到的，`arraycopy()`方法有五个参数。让我们来理解它们的含义:

*   源数组-`LANGUAGE`
*   要复制的源数组中的 from 索引-`1`
*   保存复制结果的目标数组-`result`
*   目标数组中存储复制结果的起始索引-`0`
*   我们要从源数组中复制的元素的数量-`3`

值得一提的是**如果结果数组已经包含数据，`arraycopy()`方法可能会覆盖数据**:

```java
String[] result2 = new String[] { "value one", "value two", "value three", "value four", "value five", "value six", "value seven" };
System.arraycopy(LANGUAGES, 1, result2, 2, 3);
assertArrayEquals(new String[] { "value one", "value two", "Java", "Kotlin", "Scala", "value six", "value seven" }, result2);
```

这次，`result2`数组包含 7 个元素。此外，当我们调用`arraycopy()`方法时，我们告诉它填充从`result2`中的索引 2 复制的数据。我们可以看到，复制的三个元素已经覆盖了`result2`中的原始元素。

另外，我们应该注意到`System.arraycopy()`是一个本地方法，而`Arrays.copyOfRange()`方法在内部调用`System.arraycopy()`。

## 6.使用 Apache Commons Lang3 库中的`ArrayUtils`

Apache Commons Lang3 是一个使用非常广泛的库。它的 [`ArrayUtils`](/web/20221105192802/https://www.baeldung.com/array-processing-commons-lang#arrayutils) 提供了很多方便的方法，让我们可以更容易地使用数组。

最后，让我们使用`ArrayUtils`类来解决问题。

在我们开始使用`ArrayUtils,`之前，让我们将依赖性添加到我们的 Maven 配置中:

```java
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-lang3</artifactId>
    <version>3.12.0</version>
</dependency>
```

当然，我们总能在 Maven 中央存储库中找到[最新版本](https://web.archive.org/web/20221105192802/https://search.maven.org/search?q=g:org.apache.commons%20AND%20a:commons-lang3&core=gav)。

`ArrayUtils`类有`subarray()`方法，它允许我们快速获得子数组:

```java
String[] result = ArrayUtils.subarray(LANGUAGES, 1, 4);
assertArrayEquals(JVM_LANGUAGES, result);
```

正如我们所见，使用`subarray()`方法解决这个问题非常简单。

## 7.结论

在本文中，我们学习了从给定数组中获取子数组的不同方法。

像往常一样，这里展示的所有代码片段都可以在 GitHub 上获得。