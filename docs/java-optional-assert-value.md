# 断言一个 Java 可选的有一个确定的值

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-optional-assert-value>

## 1.概观

当我们测试返回 `[Optional](/web/20220525122939/https://www.baeldung.com/java-optional)` 对象的方法时，我们可能需要编写断言来检查`Optional`是否有值或者检查其中的值。

在这个简短的教程中，我们将看看如何使用来自 [JUnit](/web/20220525122939/https://www.baeldung.com/junit-assertions) 和 [AssertJ](/web/20220525122939/https://www.baeldung.com/introduction-to-assertj) 的函数来编写这些断言。

## 2.测试可选的是否为空

如果我们只需要找出`Optional `是否有值，我们可以断言`isPresent`或`isEmpty`。

### 2.1.测试`Optional`有一个值

如果一个`Optional`有一个值，我们可以在`Optional.isPresent`上断言:

```java
assertTrue(optional.isPresent());
```

然而，AssertJ 库提供了一种更流畅的表达方式:

```java
assertThat(optional).isNotEmpty();
```

### 2.2.可选测试为空

使用 JUnit 时，我们可以颠倒一下逻辑:

```java
assertFalse(optional.isPresent());
```

同样，在 Java 11 以后，我们可以使用`Optional.isEmpty`:

```java
assertTrue(optional.isEmpty());
```

然而，AssertJ 也为我们提供了一个简洁的选择:

```java
assertThat(optional).isEmpty();
```

## 3.测试一个`Optional`的值

通常我们想要测试一个`Optional `内部的值，而不仅仅是存在或不存在。

### 3.1.使用`JUnit`断言

我们可以使用`Optional.get`来提供值，然后在那个`:`上写一个断言

```java
Optional<String> optional = Optional.of("SOMEVALUE");
assertEquals("SOMEVALUE", optional.get());
```

然而，使用 `get` 可能会导致异常，这使得理解测试失败变得更加困难。所以，我们可能更喜欢先断言值是否存在:

```java
assertTrue(optional.isPresent());
assertEquals("SOMEVALUE", optional.get());
```

然而， **`Optional`支持 equals 方法**，所以我们可以使用带有正确值的`Optional`作为一般等式断言的一部分:

```java
Optional<String> expected = Optional.of("SOMEVALUE");
Optional<String> actual = Optional.of("SOMEVALUE");
assertEquals(expected, actual);
```

### 3.2.使用`AssertJ`

对于 AssertJ `,` **我们可以使用`hasValue`流畅的断言**:

```java
assertThat(Optional.of("SOMEVALUE")).hasValue("SOMEVALUE"); 
```

## 4.结论

在本文中，我们看了几种测试`Optional`的方法。

我们看到了内置的 JUnit 断言如何与`isPresent`和`get`一起使用。我们还看到了`Optional.equals`如何在我们的断言中为我们提供了一种比较`Optional`对象的方法。

最后，我们查看了 AssertJ 断言，它为我们提供了检查我们的`Optional`值的流畅语言。

和往常一样，本文中的代码可以从 GitHub 上的[处获得。](https://web.archive.org/web/20220525122939/https://github.com/eugenp/tutorials/tree/master/testing-modules/testing-assertions)