# Java 中 parseInt()和 valueOf()的区别

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-integer-parseint-vs-valueof>

## 1.概观

我们知道，将数字`String`的[转换成`int`或`Integer`](/web/20220722192545/https://www.baeldung.com/java-convert-string-to-int-or-integer) 是 Java 中非常常见的操作。

在本教程中，我们将通过两个非常流行的`static` 方法，`java.lang.Integer `类的`parseInt()`和`valueOf()`来帮助我们完成这个转换。此外，我们还将使用简单的例子来理解这两种方法之间的一些差异。

## 2。`parseInt()` 法

类`java.lang.Integer`提供了`parseInt()`方法的三种变体。让我们来看看每一个。

### 2.1.将`String`转换为整数

**`parseInt()` 的第一个变体接受一个`String` 作为参数，并返回原始数据类型`int.`** ，当它无法将`String`转换为整数时，它抛出 *NumberFormatException* 。

让我们来看看它的签名:

```java
public static int parseInt(String s) throws NumberFormatException
```

现在，我们将看到几个例子，其中我们将有符号/无符号的数字字符串作为参数传递给它，以了解从字符串到整数的解析是如何发生的:

```java
@Test
public void whenValidNumericStringIsPassed_thenShouldConvertToPrimitiveInt() {
    assertEquals(11, Integer.parseInt("11")); 
    assertEquals(11, Integer.parseInt("+11")); 
    assertEquals(-11, Integer.parseInt("-11"));
}
```

### 2.2.指定基数

**`parseInt() `方法的第二个变体接受一个`String` 和一个`int` 作为参数，并返回原始数据类型`int.`** 就像我们看到的第一个变体**，**一样，当它无法将`String`转换为整数时，它也会抛出 *NumberFormatException* :

```java
public static int parseInt(String s, int radix) throws NumberFormatException
```

默认情况下，`parseInt() `方法假设给定的`String `是一个基数为 10 的整数。这里，参数`radix`是用于字符串到整数转换的**基数或基数。**

为了更好地理解这一点，让我们看几个例子，在这些例子中，我们将一个字符串连同`radix` 参数一起传递给`parseInt()`:

```java
@Test
public void whenValidNumericStringWithRadixIsPassed_thenShouldConvertToPrimitiveInt() {
    assertEquals(17, Integer.parseInt("11", 16));
    assertEquals(10, Integer.parseInt("A", 16)); 
    assertEquals(7, Integer.parseInt("7", 8));
}
```

现在，让我们来理解用基数进行字符串转换是如何发生的。例如，在基数为 13 的数字系统中，诸如 398 的数字串表示十进制数(基数/基数为 10) 632。换句话说，在这种情况下，计算是这样进行的——3×13²+9×13¹+8×13⁰ = 632。

同样，在上面的例子中`Integer.parseInt(“11”, 16)`通过计算得出 17，1×16^¹+1×16⁰ = 17。

### 2.3.将子字符串转换为整数

最后，`parseInt() `方法的第三个变体**接受一个`CharSequence,` 子串的两个整数`beginIndex`和`endIndex`，以及另一个整数 `radix` 作为参数。**如果传递了无效字符串，它抛出 `NumberFormatException:`

```java
public static int parseInt(CharSequence s, int beginIndex, int endIndex, int radix) throws NumberFormatException
```

JDK 9 在`Integer`类中引入了这个`static`方法。现在，让我们来看看它的运行情况:

```java
@Test
public void whenValidNumericStringWithRadixAndSubstringIsPassed_thenShouldConvertToPrimitiveInt() {
    assertEquals(5, Integer.parseInt("100101", 3, 6, 2));
    assertEquals(101, Integer.parseInt("100101", 3, 6, 10));
}
```

让我们来理解如何将子串转换成给定基数的整数。这里 string 是“100101”，`beginIndex`和`endIndex`分别是 3 和 6。因此，子字符串是“101”。对于`expectedNumber1`，传递的基数是 2，这意味着它是二进制的。因此，子字符串“101”被转换为整数 5。此外，对于`expectedNumber2,`，传递的基数是 10，这意味着它是十进制的。因此，子字符串“101”被转换为整数 101。

此外，我们可以看到当传递任何无效字符串时，`Integer.parseInt()` 抛出`NumberFormatException` :

```java
@Test(expected = NumberFormatException.class)
public void whenInValidNumericStringIsPassed_thenShouldThrowNumberFormatException(){
    int number = Integer.parseInt("abcd");
}
```

## 3. `valueOf()` 法

接下来，让我们看看由类`java.lang.Integer.`提供的`valueOf()` 方法的三个变体

### 3.1.将`String`转换为`Integer`

**`valueOf()` 方法的第一个变体接受一个`String` 作为参数并返回包装类`Integer.`** 如果传递了任何非数字字符串，它抛出 `NumberFormatException`:

```java
public static Integer valueOf(String s) throws NumberFormatException
```

有趣的是，它在实现中使用了`parseInt(String s, int radix)`。

接下来，让我们看几个从有符号/无符号数字字符串到整数转换的例子:

```java
@Test
public void whenValidNumericStringIsPassed_thenShouldConvertToInteger() {
    Integer expectedNumber = 11;
    Integer expectedNegativeNumber = -11;

    assertEquals(expectedNumber, Integer.valueOf("11"));
    assertEquals(expectedNumber, Integer.valueOf("+11"));
    assertEquals(expectedNegativeNumber, Integer.valueOf("-11"));
}
```

### 3.2.将`int`转换为`Integer`

**`valueOf()` 的第二个变体接受一个`int` 作为参数并返回包装类`Integer.`** 同样，如果任何其他数据类型如`float`被传递给它`.`，它会产生一个编译时错误

这是它的签名:

```java
public static Integer valueOf(int i)
```

除了从`int`到`Integer`的转换，这个方法还可以接受一个`char` 作为参数，并返回它的 Unicode 值。

为了进一步理解这一点，让我们看几个例子:

```java
@Test
public void whenNumberIsPassed_thenShouldConvertToInteger() {
    Integer expectedNumber = 11;
    Integer expectedNegativeNumber = -11;
    Integer expectedUnicodeValue = 65;

    assertEquals(expectedNumber, Integer.valueOf(11));
    assertEquals(expectedNumber, Integer.valueOf(+11));
    assertEquals(expectedNegativeNumber, Integer.valueOf(-11));
    assertEquals(expectedUnicodeValue, Integer.valueOf('A'));
}
```

### 3.3.指定基数

**`valueOf()` 的第三个变体接受一个`String` 和一个 `int` 作为参数，并返回包装类** `**Integer.**` 同样，像我们见过的所有其他变体一样，当它不能将给定的字符串转换为`Integer`类型时，它也会抛出`*NumberFormatException*`:

```java
public static Integer valueOf(String s, int radix) throws NumberFormatException
```

这个方法在实现**时也使用了`parseInt(String s, int radix)`。**

默认情况下，`valueOf` `() `方法假设给定的`String `表示一个基数为 10 的整数。此外，该方法接受另一个参数来改变默认基数`.`

让我们解析几个`String` 对象:

```java
@Test
public void whenValidNumericStringWithRadixIsPassed_thenShouldConvertToInetger() {
    Integer expectedNumber1 = 17;
    Integer expectedNumber2 = 10;
    Integer expectedNumber3 = 7;

    assertEquals(expectedNumber1, Integer.valueOf("11", 16));
    assertEquals(expectedNumber2, Integer.valueOf("A", 16));
    assertEquals(expectedNumber3, Integer.valueOf("7", 8));
}
```

## 4.`parseInt()`和`valueOf()`的区别

综上所述，`valueOf`()和`parseInt()`方法的主要区别如下:

| `Integer.valueOf()` | `Integer.parseInt()` |
| 它返回一个`Integer`对象。 | 它返回一个原语`int`。 |
| 该方法接受`String`和`int`作为参数。 | 这个方法只接受`String`作为参数。 |
| 它在方法实现中使用了`Integer.parseInt()`。 | 它不使用任何辅助方法将字符串解析为整数。 |
| 此方法接受字符作为参数，并返回其 Unicode 值。 | 此方法在将字符作为参数传递时会产生不兼容的类型错误。 |

## 5.结论

在本文中，我们了解了`java.lang.Integer`类的`parseInt()`和`valueOf()`方法的不同实现。我们还研究了这两种方法之间的差异。

和往常一样，本文的完整代码示例可以在 GitHub 的[中找到。](https://web.archive.org/web/20220722192545/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-string-conversions-2)