# 在 Java 中将字符串转换为 int 或 Integer

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-convert-string-to-int-or-integer>

## 1。简介

将`String`转换成`int`或`Integer`是 Java 中非常常见的操作。在本文中，我们将展示处理这个问题的多种方法。

有一些简单的方法来处理这种基本的转换。

## 2。`Integer.parseInt()`

一个主要的解决方案是**使用`Integer`的专用静态方法:`parseInt()`，它返回一个原始的`int`值**:

```java
@Test
public void givenString_whenParsingInt_shouldConvertToInt() {
    String givenString = "42";

    int result = Integer.parseInt(givenString);

    assertThat(result).isEqualTo(42);
}
```

默认情况下，`parseInt() `方法假设给定的`String `是一个基数为 10 的整数。此外，该方法接受另一个参数来更改默认的`radix. ` **例如，我们可以如下解析二进制`String`s:**

```java
@Test
public void givenBinaryString_whenParsingInt_shouldConvertToInt() {
    String givenString = "101010";

    int result = Integer.parseInt(givenString, 2);

    assertThat(result).isEqualTo(42);
}
```

当然，这种方法也可以用于任何其他基数，比如 16(十六进制)或 8(八进制)。

## 3。`Integer.valueOf()`

另一个选择是**使用静态的`Integer.valueOf()`方法，该方法返回一个`Integer`实例**:

```java
@Test
public void givenString_whenCallingIntegerValueOf_shouldConvertToInt() {
    String givenString = "42";

    Integer result = Integer.valueOf(givenString);

    assertThat(result).isEqualTo(new Integer(42));
}
```

类似地，`valueOf() `方法也接受一个自定义的`radix `作为第二个参数:

```java
@Test
public void givenBinaryString_whenCallingIntegerValueOf_shouldConvertToInt() {
    String givenString = "101010";

    Integer result = Integer.valueOf(givenString, 2);

    assertThat(result).isEqualTo(new Integer(42));
}
```

### 3.1.整数缓存

乍一看，`valueOf()`和`parseInt()`方法似乎完全一样。在很大程度上，这是真的——即使是`valueOf()`方法也在内部委托给了`parseInt`方法。

然而，这两种方法之间有一个微妙的区别:**`valueOf()`方法在内部使用整数缓存**。这个缓存将**为-128 和 127** 之间的数字返回相同的`Integer`实例:

```java
@Test
public void givenString_whenCallingValueOf_shouldCacheSomeValues() {
    for (int i = -128; i <= 127; i++) {
        String value = i + "";
        Integer first = Integer.valueOf(value);
        Integer second = Integer.valueOf(value);

        assertThat(first).isSameAs(second);
    }
}
```

因此，强烈推荐使用`valueOf()`而不是`parseInt()`来提取装箱的整数，因为这可能会为我们的应用程序带来更好的整体占用空间。

## 4。`Integer`的建造者

您也可以使用`Integer`的构造函数:

```java
@Test
public void givenString_whenCallingIntegerConstructor_shouldConvertToInt() {
    String givenString = "42";

    Integer result = new Integer(givenString);

    assertThat(result).isEqualTo(new Integer(42));
}
```

从 Java 9 开始，**这个构造函数已经被[弃用](https://web.archive.org/web/20220816192910/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/Integer.html#%3Cinit%3E(java.lang.String))** ，取而代之的是其他静态工厂方法，比如`valueOf() `或`parseInt()`。即使在此之前，使用这个构造函数也是不合适的。我们应该使用`parseInt()`将字符串转换为`int`原语，或者使用`valueOf()`将其转换为`Integer`对象。

## 5。`Integer.decode()`

另外， `Integer.decode()`的工作方式与`Integer.valueOf(),` 类似，但也可以接受不同的[数字表示](https://web.archive.org/web/20220816192910/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/Integer.html#decode(java.lang.String)):

```java
@Test
public void givenString_whenCallingIntegerDecode_shouldConvertToInt() {
    String givenString = "42";

    int result = Integer.decode(givenString);

    assertThat(result).isEqualTo(42);
}
```

## 6。`NumberFormatException`

上面提到的所有方法在遇到意外的`String`值时都会抛出一个`NumberFormatException,`。这里你可以看到这种情况的一个例子:

```java
@Test(expected = NumberFormatException.class)
public void givenInvalidInput_whenParsingInt_shouldThrow() {
    String givenString = "nan";
    Integer.parseInt(givenString);
}
```

## 7。有番石榴

当然，我们不需要坚持核心 Java 本身。这就是如何使用 Guava 的`Ints.tryParse(),` 实现同样的事情，如果它不能解析输入，它将返回一个`null`值:

```java
@Test
public void givenString_whenTryParse_shouldConvertToInt() {
    String givenString = "42";

    Integer result = Ints.tryParse(givenString);

    assertThat(result).isEqualTo(42);
}
```

此外，`tryParse()` 方法还接受类似于`parseInt() `和`valueOf().`的第二个`radix `参数

## 8。结论

在本文中，我们探索了多种将`String`实例转换为`int`或`Integer`实例的方法。

当然，所有代码示例都可以在 GitHub 上找到[。](https://web.archive.org/web/20220816192910/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-string-conversions)