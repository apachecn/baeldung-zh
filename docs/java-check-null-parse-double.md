# 在调用 Double.parseDouble 中的 Parse 之前检查是否为 null

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-check-null-parse-double>

## 1.概观

当[将一个 Java `String`转换成一个`double`](/web/20221208143926/https://www.baeldung.com/java-string-to-double) `,`时，我们通常会使用`Double.parseDouble(String value)` 方法。这个方法允许我们将一个给定的`double`的 `String`表示——例如，“2.0”——转换成一个原始的`double`值。

与大多数方法调用一样，避免传递 `null` 引用是一个好习惯，这很可能会在运行时导致 `NullPointerException` 。

**在本教程中，我们将探索几种在调用`Double.parseDouble`** 之前检查`null ` **的方法。在查看一些外部库之前，我们将首先考虑使用核心 Java 的解决方案。**

## 2.为什么要检查

首先，让我们了解一下**如果我们在解析一个 S `tring`** 时不检查`null`值会发生什么。让我们从路过一个空的`String`开始:

```java
double emptyString = Double.parseDouble("");
```

当我们运行这段代码时，它会抛出一个`java.lang.NumberFormatException`:

```java
Exception in thread "main" java.lang.NumberFormatException: empty String
	at sun.misc.FloatingDecimal.readJavaFormatString(FloatingDecimal.java:1842)
	at sun.misc.FloatingDecimal.parseDouble(FloatingDecimal.java:110)
	at java.lang.Double.parseDouble(Double.java:538)
	...
```

现在让我们考虑传递一个`null`引用:

```java
double nullString = Double.parseDouble(null);
```

不出所料，这次会抛出一个`java.lang.NullPointerException`:

```java
Exception in thread "main" java.lang.NullPointerException
	at sun.misc.FloatingDecimal.readJavaFormatString(FloatingDecimal.java:1838)
	at sun.misc.FloatingDecimal.parseDouble(FloatingDecimal.java:110)
	at java.lang.Double.parseDouble(Double.java:538)
	...
```

众所周知，在我们的应用程序代码中使用异常是一个很好的实践。但是一般来说，**我们应该避免这种[未检查的](/web/20221208143926/https://www.baeldung.com/java-checked-unchecked-exceptions)异常，这很可能是编程错误**的结果。

## 3.如何用核心 Java 检查

在这一节中，我们将看看使用核心 Java 检查`null`或空值的几个选项。

### 3.1.使用普通 Java

让我们从定义一个简单的方法开始，该方法将检查我们传递的值是`null`还是空的`String`:

```java
private static double parseStringToDouble(String value) {
    return value == null || value.isEmpty() ? Double.NaN : Double.parseDouble(value);
}
```

正如我们看到的，如果我们试图解析的值是`null`或空，这个方法返回[而不是数字](/web/20221208143926/https://www.baeldung.com/java-not-a-number)。否则，我们调用`Double.parseDouble`方法。

**我们可以更进一步，提供预定义默认值**的可能性:

```java
private static double parseStringToDouble(String value, double defaultValue) {
    return value == null || value.isEmpty() ? defaultValue : Double.parseDouble(value);
}
```

当我们调用这个方法时，如果提供的值为`null`或空，我们提供一个适当的默认值来返回:

```java
assertThat(parseStringToDouble("1", 2.0d)).isEqualTo(1.0d);
assertThat(parseStringToDouble(null, 1.0d)).isEqualTo(1.0d);
assertThat(parseStringToDouble("", 1.0d)).isEqualTo(1.0d);
```

### 3.2.使用`Optional`

现在让我们来看看 [`Optional`](/web/20221208143926/https://www.baeldung.com/java-optional) 的**用法的不同解决方案:**

```java
private static Optional parseStringToOptionalDouble(String value) {
    return value == null || value.isEmpty() ? Optional.empty() : Optional.of(Double.valueOf(value));
}
```

这一次，我们使用`Optional`作为[返回类型](/web/20221208143926/https://www.baeldung.com/java-optional-return)。因此，当我们调用这个方法时，我们就有可能调用标准方法，比如`isPresent()`和`isEmpty()`，来确定一个值是否存在:

```java
parseStringToOptionalDouble("2").isPresent()
```

我们也可以使用`Optional`的`orElse`方法返回一个默认值:

```java
parseStringToOptionalDouble("1.0").orElse(2.0d) 
parseStringToOptionalDouble(null).orElse(2.0d) 
parseStringToOptionalDouble("").orElse(2.0d)
```

## 4.外部库

现在我们已经很好地理解了如何使用核心 Java 来检查`null`和空值，让我们来看看一些外部库。

### 4.1。谷歌番石榴

我们要看的第一个外部解决方案是[谷歌番石榴](/web/20221208143926/https://www.baeldung.com/whats-new-in-guava-19)，它可以在 [Maven Central](https://web.archive.org/web/20221208143926/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22com.google.guava%22%20AND%20a%3A%22guava%22) 上获得:

```java
<dependency>
    <groupId>com.google.guava</groupId>
    <artifactId>guava</artifactId>
    <version>31.0.1-jre</version>
</dependency>
```

我们可以简单地使用`Doubles.tryParse`方法:

```java
Doubles.tryParse(MoreObjects.firstNonNull("1.0", "2.0"))
Doubles.tryParse(MoreObjects.firstNonNull(null, "2.0"))
```

在这个例子中，我们还使用了`MoreObjects.firstNonNull` 方法，它将返回两个给定参数中不是`null`的第一个。

这段代码在大多数情况下都能正常工作，但是让我们想象一个不同的例子:

```java
Doubles.tryParse(MoreObjects.firstNonNull("", "2.0"))
```

在这种情况下，由于空的`String`不是`null`，该方法将返回`null`，而不是抛出一个`NumberFormatException`。**我们避免了异常，但是我们仍然必须在应用程序代码的某个地方处理一个`null`值。**

### 4.2 .Apache common lang`NumberUtils`

`NumberUtils`类提供了许多有用的工具，使数字处理变得更加容易。

从 [Maven Central](https://web.archive.org/web/20221208143926/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22org.apache.commons%22%20AND%20a%3A%22commons-lang3%22) 可以获得 [Apache Commons Lang](https://web.archive.org/web/20221208143926/https://commons.apache.org/proper/commons-lang/) 工件:

```java
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-lang3</artifactId>
    <version>3.12.0</version>
</dependency>
```

那么我们可以简单地使用来自`NumberUtils`的方法`toDouble`:

```java
NumberUtils.toDouble("1.0")
NumberUtils.toDouble("1.0", 1.0d) 
```

这里，我们有两个选择:

*   将一个`String`转换成一个`double`，如果转换失败，返回`0.0d`
*   将一个`String`转换为一个`double`，如果转换失败，提供一个定义的默认值

如果我们传递一个空值或`null`值，默认情况下会返回`0.0d`:

```java
assertThat(NumberUtils.toDouble("")).isEqualTo(0.0d);
assertThat(NumberUtils.toDouble(null)).isEqualTo(0.0d);
```

**这比前一个例子好，因为无论转换过程中发生什么，我们总是得到一个`double`返回值。**

### 4.3。Vavr

最后，但同样重要的是，让我们看看 **[vavr.io](/web/20221208143926/https://www.baeldung.com/vavr) ，它提供了一种函数式方法**。

和往常一样，这个神器可以在 [Maven Central](https://web.archive.org/web/20221208143926/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22io.vavr%22%20AND%20a%3A%22vavr%22) 上找到:

```java
<dependency>
    <groupId>io.vavr</groupId>
    <artifactId>vavr</artifactId>
    <version>0.10.2</version>
</dependency>
```

同样，我们将定义一个使用 vavr [`Try`](/web/20221208143926/https://www.baeldung.com/vavr-try) 类的简单方法:

```java
public static double tryStringToDouble(String value, double defaultValue) {
    return Try.of(() -> Double.parseDouble(value)).getOrElse(defaultValue);
} 
```

我们将以与其他示例完全相同的方式调用该方法:

```java
assertThat(tryStringToDouble("1", 2.0d)).isEqualTo(1.0d);
assertThat(tryStringToDouble(null, 2.0d)).isEqualTo(2.0d);
assertThat(tryStringToDouble("", 2.0d)).isEqualTo(2.0d);
```

## 5.结论

在这个快速教程中，我们探索了几种在调用`Double.parseDouble`方法之前检查`null`和空字符串的方法。

和往常一样，GitHub 上的[提供了这篇文章的完整源代码。](https://web.archive.org/web/20221208143926/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-numbers-3)