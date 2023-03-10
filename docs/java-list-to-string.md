# 在 Java 中将列表转换为字符串

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-list-to-string>

## 1。简介

在这个快速教程中，我们将解释如何将元素的`List`转换成`String`。这在某些情况下很有用，比如以人类可读的形式将内容打印到控制台，以便检查/调试。

## 2。标准`toString()`上有`List`下有

最简单的方法之一是在`List`上调用`toString()`方法:

```java
@Test
public void whenListToString_thenPrintDefault() {
    List<Integer> intLIst = Arrays.asList(1, 2, 3);

    System.out.println(intLIst);
}
```

输出:

```java
[1, 2, 3]
```

这种技术在内部利用了`List`中元素类型的`toString()`方法。在我们的例子中，我们使用的是`Integer`类型，它正确实现了`toString()`方法。

如果我们使用自定义类型，比如`Person`，那么我们需要确保`Person`类覆盖了`toString()`方法，并且不依赖于默认实现。如果我们没有正确实现`toString()`方法，我们可能会得到意想不到的结果:

```java
[[[email protected]](/web/20220929201254/https://www.baeldung.com/cdn-cgi/l/email-protection),
  [[email protected]](/web/20220929201254/https://www.baeldung.com/cdn-cgi/l/email-protection),
  [[email protected]](/web/20220929201254/https://www.baeldung.com/cdn-cgi/l/email-protection)]
```

## 3。使用`Collectors` 自定义实现

通常，我们可能需要以不同的格式显示输出。

与前面的示例相比，让我们用连字符(-)替换逗号(，)，用一组花括号({，})替换方括号([，]):

```java
@Test
public void whenCollectorsJoining_thenPrintCustom() {
    List<Integer> intList = Arrays.asList(1, 2, 3);
    String result = intList.stream()
      .map(n -> String.valueOf(n))
      .collect(Collectors.joining("-", "{", "}"));

    System.out.println(result);
}
```

输出:

```java
{1-2-3}
```

`Collectors.joining()`方法需要一个`CharSequence`，所以我们需要将`Integer`的`map`转换为`String`。我们可以在其他类中利用同样的思想，即使我们无法访问该类的代码。

## 4。使用外部库

现在我们将使用 Apache Commons 的`StringUtils`类来实现类似的结果。

### 4.1。Maven 依赖关系

```java
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-lang3</artifactId>
    <version>3.11</version>
</dependency>
```

依赖关系的最新版本可以在[这里](https://web.archive.org/web/20220929201254/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.apache.commons%22%20AND%20a%3A%22commons-lang3%22)找到。

### 4.2。实施

该实现实际上是一个方法调用:

```java
@Test
public void whenStringUtilsJoin_thenPrintCustom() {
    List<Integer> intList = Arrays.asList(1, 2, 3);

    System.out.println(StringUtils.join(intList, "|"));
}
```

输出:

```java
1|2|3
```

同样，这个实现在内部依赖于我们正在考虑的类型的`toString()`实现。

## 5。结论

在本文中，我们了解了使用不同的技术将`List`转换成`String`是多么容易。

和往常一样，本文的完整源代码可以在 GitHub 上找到[。](https://web.archive.org/web/20220929201254/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-collections-conversions)