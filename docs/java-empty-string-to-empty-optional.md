# 将空字符串转换为空的可选

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-empty-string-to-empty-optional>

## 1.介绍

在这个快速教程中，我们将介绍不同的方法来用**将一个`null `或空的`String`转换成一个空的`[Optional](/web/20221126234405/https://www.baeldung.com/java-optional).`**

从`null`中获取一个空的`Optional`非常简单——我们只需使用 [`Optional.ofNullable()`](https://web.archive.org/web/20221126234405/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/Optional.html#ofNullable(T)) 。但是，如果我们希望空的`String`也这样工作呢？

因此，让我们探索一下将空的`String`转换成空的`Optional.`的不同选择

## 2.使用 Java 8

在 Java 8 中，我们可以利用这样一个事实:如果不满足`Optional#filter`的谓词，那么**将返回一个空的`Optional` :**

```java
@Test
public void givenEmptyString_whenFilteringOnOptional_thenEmptyOptionalIsReturned() {
    String str = "";
    Optional<String> opt = Optional.ofNullable(str).filter(s -> !s.isEmpty());
    Assert.assertFalse(opt.isPresent());
}
```

我们甚至不需要在这里检查`null `，因为 **`ofNullable `会在`str`为空的情况下为我们短路。**

不过，为谓词创建一个特殊的 [lambda](/web/20221126234405/https://www.baeldung.com/java-8-lambda-expressions-tips) 有点麻烦。我们不能想办法摆脱它吗？

## 3.使用 Java 11

上述愿望的答案实际上直到 Java 11 才出现。

在 Java 11 中，我们仍将使用`[Optional.filter()](https://web.archive.org/web/20221126234405/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/Optional.html#filter(java.util.function.Predicate)),`，但是 Java 11 引入了一个新的`[Predicate.not()](https://web.archive.org/web/20221126234405/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/function/Predicate.html#not(java.util.function.Predicate))` API，使得否定方法引用变得容易。

因此，让我们简化我们之前所做的，现在使用方法引用来代替:

```java
@Test
public void givenEmptyString_whenFilteringOnOptionalInJava11_thenEmptyOptionalIsReturned() {
    String str = "";
    Optional<String> opt = Optional.ofNullable(str).filter(Predicate.not(String::isEmpty));
    Assert.assertFalse(opt.isPresent());
}
```

## 4.用番石榴

我们也可以用番石榴来满足我们的需求。然而，在这种情况下，我们将使用稍微不同的方法。

我们不是对`Optional#ofNullable`的结果调用`filter`方法，而是首先使用 Guava 的`String#emptyToNull` 将一个空的`String`转换为`null`，然后才将其传递给`Optional#ofNullable`:

```java
@Test
public void givenEmptyString_whenPassingResultOfEmptyToNullToOfNullable_thenEmptyOptionalIsReturned() {
    String str = "";
    Optional<String> opt = Optional.ofNullable(Strings.emptyToNull(str));
    Assert.assertFalse(opt.isPresent());
}
```

## 5.结论

在这篇短文中，我们探索了将空的`String`转换成空的`Optional.`的不同方法

像往常一样，本文中使用的例子可以在我们的 [GitHub 项目](https://web.archive.org/web/20221126234405/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-optional)中找到。