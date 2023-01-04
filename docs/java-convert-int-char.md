# Java 中 int 和 char 之间的转换

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-convert-int-char>

## 1.概观

在本教程中，我们将看到如何在 Java 中从`int`转换到`char`并返回。

## 2.`char`在爪哇

我们将简要讨论字符是如何表示的，以便更好地理解我们将在本文后面看到的代码。

在内部， **Java 将每个`char `存储为 16 位 Unicode 编码值**:

| 性格；角色；字母 | 2 字节 | 十进制(基数为 10) | 十六进制(基数为 16) |
| A | 00000000 01000001 | Sixty-five | Forty-one |
| a | 00000000 01100001 | Sixty-one | Ninety-seven |
| one | 00000000 00110001 | forty-nine | Thirty-one |
| Z | 00000000 01011010 | Ninety | 5A |

我们可以通过将`char`值转换为`int`来轻松检查这一点:

```java
assertEquals(65, (int)'A');
```

[ASCII 码](/web/20221219195909/https://www.baeldung.com/cs/ascii-code)是 Unicode 编码的子集，主要代表英文字母。

## 3.将`int`转换为`char`

假设我们有一个值为 7 的`int` 变量，我们想把它转换成它的`char`对应变量“`7`”。我们有几个选择来做这件事。

**简单地将它转换成`char`是行不通的，因为这会将它转换成二进制表示为`0111`** `,`的字符，在 UTF-16 中是`U+0007`或字符[‘BELL’](https://web.archive.org/web/20221219195909/https://www.fileformat.info/info/unicode/char/0007/index.htm)。

### 3.1.偏移“0”

UTF-16 中的字符是按顺序表示的。所以我们可以用`7`来抵消`0`字符，得到`7`字符:

```java
@Test
public void givenAnInt_whenAdding0_thenExpectedCharType() {
    int num = 7;

    char c = (char)('0' + num);

    assertEquals('7', c);
}
```

### 3.2.使用`Character.forDigit()`方法

加'`0`'可以，但是好像有点 hackish。幸运的是，我们有一个更干净的方法来使用`[Character.forDigit()](https://web.archive.org/web/20221219195909/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/Character.html#forDigit(int,int))`方法:

```java
@Test
public void givenAnInt_whenUsingForDigit_thenExpectedCharType() {
    int num = 7;

    char c = Character.forDigit(num , 10);

    assertEquals('7', c);
}
```

我们可以注意到，`forDigit` `()`方法接受了第二个参数，`radix,`，它代表了我们想要转换的数字的基本表示。在我们这里是`10`。

### 3.3.使用`Integer.toString()`方法

我们可以使用包装类`Integer,`,它具有将给定的`int`转换成其`String`表示的`[toString()](/web/20221219195909/https://www.baeldung.com/java-tostring-valueof)`方法。当然，这可以用来将一个多位数的数字转换成`String. B` ut，我们也可以通过链接 [`charAt()`](/web/20221219195909/https://www.baeldung.com/java-convert-string-to-char) 方法并选择第一个`char`来将一位数转换成`char`:

```java
@Test
public void givenAnInt_whenUsingToString_thenExpectedCharType() {
    int num = 7;

    char c = Integer.toString(num).charAt(0);

    assertEquals('7', c);
}
```

## 4.将`char`转换为`int`

之前，我们看到了如何将一个`int`转换成`char`。让我们看看如何得到一个`char`的`int`值。正如我们可能预料的那样，**将`char`转换为`int`是行不通的，因为这给了我们字符的 UTF-16 编码的十进制表示:**

```java
@Test
public void givenAChar_whenCastingFromCharToInt_thenExpectedUnicodeRepresentation() {

    char c = '7';

    assertEquals(55, (int) c); 
}
```

### 4.1.减去“0”

如果当我们加上‘0’时，我们得到`char,` ，那么反过来减去十进制值‘0’，我们应该得到`int`:

```java
@Test
public void givenAChar_whenSubtracting0_thenExpectedNumericType() {

    char c = '7';

    int n = c - '0';

    assertEquals(7, n);
}
```

这确实有效，但是还有更好更简单的方法。

### 4.2.使用`Character.getNumericValue()`方法

`Character` 类再次提供了另一个助手方法， [`getNumericValue()`](https://web.archive.org/web/20221219195909/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/Character.html#getNumericValue(char)) `,`，它基本上实现了它所说的功能:

```java
@Test
public void givenAChar_whenUsingGetNumericValue_thenExpectedNumericType() {

    char c = '7';

    int n = Character.getNumericValue(c);

    assertEquals(7, n);
}
```

### 4.3.使用`Integer.parseInt()`

我们可以使用`[Integer.parseInt()](https://web.archive.org/web/20221219195909/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/Integer.html#parseInt(java.lang.String))`将`String`转换成数字表示。就像以前一样，虽然我们可以用它将一个多位数的整数转换成`int`表示，但我们也可以用它来表示一位数:

```java
@Test
public void givenAChar_whenUsingParseInt_thenExpectedNumericType() {

    char c = '7';

    int n = Integer.parseInt(String.valueOf(c));

    assertEquals(7, n);
}
```

事实上，语法有点麻烦，主要是因为它涉及到多次转换，但它能按预期工作。

## 5.结论

在本文中，我们学习了字符在 Java 中是如何内部表示的，以及如何在`int`和`char`之间相互转换。

和往常一样，本文的完整代码示例可以在 GitHub 的[中找到。](https://web.archive.org/web/20221219195909/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-lang-5)