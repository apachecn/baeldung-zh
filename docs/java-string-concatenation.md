# Java 中的字符串连接

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-string-concatenation>

## 1.概观

Java 中的字符串连接是最常见的操作之一。在本教程中，我们将介绍一些字符串连接的方法。但是，我们将重点描述如何使用`concat()`和`+`操作符方法。最后，我们将讨论如何根据我们需要做的事情来选择正确的选项。

## 2.拼接方法

一般来说，在 Java 中连接两个或多个字符串有不同的方法。此外，我们将查看一些示例，并对每个示例进行描述。

### 2.1.使用“`+`”运算符

****Java 中最常见的连接方法之一是使用“`+`”操作符**。**

 **与其他方法相比，“`+`”操作符为字符串连接提供了更大的灵活性。首先，它不会对空值抛出任何异常。其次，它将 null 转换成它的字符串表示形式。此外，我们可以用它来连接两个以上的字符串。

让我们看一个代码示例:

```java
@Test
void whenUsingPlusOperatorANull_thenAssertEquals() {
    String stringOne = "Hello ";
    String stringTwo = null;
    assertEquals("Hello null", stringOne + stringTwo);
}
```

编译器在内部将“`+`”操作符转换成一个`StringBuilder`(或`StringBuffer`)类及其`append()`方法。

自从**`+`操作符无声地将自变量转换为`String`** (使用`toString()`方法获得对象)**后，我们避开了** **`NullPointerException`** 。然而，我们需要考虑我们的最终字符串结果是否适用于字符串体中的“null”。

### 2.2.使用`concat() `方法

`String`类中的`concat()`方法在当前字符串的末尾追加一个指定的字符串，并返回新的组合字符串。假设`String`类是不可变的，那么最初的`String`不会改变。

让我们测试一下这个行为:

```java
@Test
void whenUsingConcat_thenAssertEquals() {
    String stringOne = "Hello";
    String stringTwo = " World";
    assertEquals("Hello World", stringOne.concat(stringTwo));
}
```

在前面的例子中，`stringOne`变量是基本字符串。对于`concat()`方法，`stringTwo` 被附加在`stringOne`的末尾。**`concat()`操作是不可变的**，所以我们需要一个显式的赋值。下一个例子说明了这种情况:

```java
@Test
void whenUsingConcatWithOutAssignment_thenAssertNotEquals() {
    String stringOne = "Hello";
    String stringTwo = " World";
    stringOne.concat(stringTwo);
    assertNotEquals("Hello World", stringOne); // we get only Hello
}
```

此外，在这种情况下，为了获得最终的连接字符串，我们需要将`concat()`结果赋给一个变量:

```java
stringOne = stringOne.concat(stringTwo);
assertEquals("Hello World", stringOne);
```

`concat()`的另一个有用的特性是当我们需要连接多个`String`对象时。这个方法允许它。此外，我们还可以添加空格和特殊字符:

```java
@Test
void whenUsingConcatToMultipleStringConcatenation_thenAssertEquals() {
    String stringOne = "Hello";
    String stringTwo = "World";
    String stringThree = ", in Jav";
    stringOne = stringOne.concat(" ").concat(stringTwo).concat(stringThree).concat("@");
    assertEquals("Hello World, in [[email protected]](/web/20221118160122/https://www.baeldung.com/cdn-cgi/l/email-protection)", stringOne);
}
```

nulls 呢？当前字符串和要追加的字符串都不能为空值。否则， **`concat()` 法** **投** **一** **`NullPointerException`** :

```java
@Test
void whenUsingConcatAppendANull_thenAssertEquals() {
    String stringOne = "Hello";
    String stringTwo = null;
    assertThrows(NullPointerException.class, () -> stringOne.concat(stringTwo));
}
```

### 2.3.`StringBuilder` 类

首先，我们有 [`StringBuilder`](/web/20221118160122/https://www.baeldung.com/java-string-builder-string-buffer) 类。这个类提供了执行连接操作的`append()`方法。下一个例子向我们展示了它是如何工作的:

```java
@Test
void whenUsingStringBuilder_thenAssertEquals() {
    StringBuilder builderOne = new StringBuilder("Hello");
    StringBuilder builderTwo = new StringBuilder(" World");
    StringBuilder builder = builderOne.append(builderTwo);
    assertEquals("Hello World", builder.toString());
}
```

另一方面，类似的串联方法是 [`StringBuffe` r](/web/20221118160122/https://www.baeldung.com/java-string-builder-string-buffer) 类。与非同步(即非线程安全)的 **`StringBuilder`相反， **`StringBuffer`是同步的**(即线程安全)。但是性能比`StringBuilder`差。它有一个`append()`方法，就像`StringBuilder` 一样。**

### 2.4.字符串`format()` 方法

执行字符串连接的另一种方式是在 string 类中使用`format()`方法。使用像`%s,`这样的格式说明符，我们可以通过字符串值或对象连接多个字符串:

```java
@Test
void whenUsingStringFormat_thenAssertEquals() {
    String stringOne = "Hello";
    String stringTwo = " World";
    assertEquals("Hello World", String.format("%s%s", stringOne, stringTwo));
}
```

### 2.5.Java 8 及更高版本中的连接方法

对于 Java 8 及更高版本，`String` 类中的方法`join()`可以执行字符串连接。在这种情况下，该方法将在要连接的字符串之间使用的分隔符作为第一个参数:

```java
@Test
void whenUsingStringJoin_thenAssertEquals() {
    String stringOne = "Hello";
    String stringTwo = " World";
    assertEquals("Hello World", String.join("", stringOne, stringTwo));
}
```

从 Java 8 开始， [`StringJoiner`](/web/20221118160122/https://www.baeldung.com/java-string-joiner) 类被加入。这个类使用分隔符、前缀和后缀来连接`String`。以下代码片段是其用法的示例:

```java
@Test
void whenUsingStringJoiner_thenAssertEquals() {
    StringJoiner joiner = new StringJoiner(", ");
    joiner.add("Hello");
    joiner.add("World");
    assertEquals("Hello, World", joiner.toString());
}
```

此外，在 Java 8 中，添加了[流 API](/web/20221118160122/https://www.baeldung.com/java-8-streams) ，我们可以找到[收集器](/web/20221118160122/https://www.baeldung.com/java-8-collectors)。`Collectors`类有`joining()`方法。这个方法类似于`String`类中的`join()`方法。它是用来收藏的。以下示例代码片段向我们展示了它是如何工作的:

```java
@Test
void whenUsingCollectors_thenAssertEquals() {
    List<String> words = Arrays.asList("Hello", "World");
    String collect = words.stream().collect(Collectors.joining(", "));
    assertEquals("Hello, World", collect);
}
```

## 3.选择方法

最后，如果我们需要在 `concat()`方法和`+`操作符之间做出选择，我们需要考虑一些方面。

首先，`concat()`方法只接受字符串。同时，“`+`”操作符接受任何类型并将其转换为字符串。另一方面，`concat()`方法在空值上引发了一个`NullPointerExeption`，而“`+`操作符却不是这样。

此外，两者之间还有一个[性能](/web/20221118160122/https://www.baeldung.com/java-string-performance)的差异。`concat()`方法比“`+`操作符执行得更好。后者总是创建一个新的字符串，而不考虑字符串的长度。此外，我们需要考虑到`concat()`方法只在要追加的字符串长度大于 0 时才创建一个新字符串。否则，它返回相同的对象。

## 4.结论

在本文中，我们简要介绍了 Java 中的字符串连接。此外，我们详细讨论了使用`concat()`和`+`操作符来执行字符串连接。最后，我们对`concat()`方法和`+`操作符进行了比较分析，以及我们如何在不同的上下文中选择其中的一个。

和往常一样，本文中使用的所有片段都可以在 GitHub 上获得[。](https://web.archive.org/web/20221118160122/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-string-operations-5)**