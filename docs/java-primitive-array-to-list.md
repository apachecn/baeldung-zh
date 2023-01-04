# 将基元数组转换为列表

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-primitive-array-to-list>

## 1.概观

在这个简短的教程中，**我们将展示如何将一个基元数组转换成一个对应类型**的`List`对象。通常，我们可能会尝试在 Java 中使用[自动装箱](/web/20220926181526/https://www.baeldung.com/java-wrapper-classes#autoboxing-and-unboxing)。然而，正如我们将在下一节看到的，我们对自动装箱如何工作的直觉经常是错误的。

## 2.问题

让我们从问题的定义开始。**我们有一个原语数组** ( `int[]`)，**，我们希望将该数组转换成一个`List`** ( `List<Integer>`)。直观的第一次尝试可能是:

```
int[] input = new int[]{1,2,3,4};
List<Integer> output = Arrays.asList(input);
```

不幸的是，由于类型不兼容，这个**无法编译。我们可能期望自动装箱能够处理原语数组。然而，这种本能的信念并不正确。**

**自动装箱只发生在单个元素**(例如从`int`到`Integer`)。没有从原始类型数组到它们的装箱引用类型数组的自动转换(例如从`int[]`到`Integer[]`)。

让我们开始实现这个问题的几个解决方案。

## 3.循环

由于自动装箱处理单个原始元素，**一个简单的解决方案是迭代数组的元素，然后将它们逐个添加到`List`**:

```
int[] input = new int[]{1,2,3,4};
List<Integer> output = new ArrayList<Integer>();
for (int value : input) {
    output.add(value);
}
```

我们已经解决了这个问题，但是解决方案相当冗长。这将我们带到下一个实现。

## 4.流

**从 Java 8 开始，我们可以使用 [`Stream` API](/web/20220926181526/https://www.baeldung.com/java-streams)** 。我们可以使用`Stream`提供一行解决方案:

```
int[] input = new int[]{1,2,3,4};
List<Integer> output = Arrays.stream(input).boxed().collect(Collectors.toList());
```

或者，我们可以使用`IntStream`:

```
int[] input = new int[]{1,2,3,4};
List<Integer> output = IntStream.of(input).boxed().collect(Collectors.toList());
```

这看起来肯定好多了。接下来，我们将看看几个外部库。

## 5.番石榴

**[番石榴](/web/20220926181526/https://www.baeldung.com/category/guava/)库为这个问题提供了一个包装器**。让我们从添加 [Maven](https://web.archive.org/web/20220926181526/https://search.maven.org/artifact/com.google.guava/guava) 依赖项开始:

```
<dependency>
    <groupId>com.google.guava</groupId>
    <artifactId>guava</artifactId>
    <version>31.0.1-jre</version>
    <type>bundle</type>
</dependency>
```

我们可以使用`Ints.asList()`，对于其他原始类型，我们可以使用类似的实用程序类:

```
int[] input = new int[]{1,2,3,4};
List<Integer> output = Ints.asList(input);
```

## 6.Apache common(Apache 公共)

另一个库是 Apache Commons Lang。同样，让我们为这个库添加 Maven 依赖项:

```
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-lang3</artifactId>
    <version>3.12.0</version>
</dependency>
```

更准确地说，我们使用了`ArrayUtils` 类:

```
int[] input = new int[]{1,2,3,4};
Integer[] outputBoxed = ArrayUtils.toObject(input);
List<Integer> output = Arrays.asList(outputBoxed);
```

## 7.结论

现在我们的工具箱中有几个选项可以将一组原语转换成一个`List`。**正如我们所见，自动装箱只发生在单个元素上。在本教程中，我们已经看到了几个应用转换**的解决方案。

和往常一样，GitHub 上的[提供了这篇文章的完整源代码。](https://web.archive.org/web/20220926181526/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-collections)