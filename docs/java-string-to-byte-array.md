# 将字符串转换为字节数组，并在 Java 中进行反向转换

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-string-to-byte-array>

## 1.介绍

我们经常需要在 Java 中的`String`和`byte`数组之间进行转换。在本教程中，我们将详细研究这些操作。

## 延伸阅读:

## [Java InputStream 到字节数组和字节缓冲区](/web/20220831112445/https://www.baeldung.com/convert-input-stream-to-array-of-bytes)

How to convert an InputStream to a byte[] using plain Java, Guava or Commons IO.[Read more](/web/20220831112445/https://www.baeldung.com/convert-input-stream-to-array-of-bytes) →

## [Java–读取字节数组](/web/20220831112445/https://www.baeldung.com/java-convert-reader-to-byte-array)

How to convert a Reader into a byte[] using plain Java, Guava or the Apache Commons IO library.[Read more](/web/20220831112445/https://www.baeldung.com/java-convert-reader-to-byte-array) →

## [Java 字节数组到 InputStream](/web/20220831112445/https://www.baeldung.com/convert-byte-array-to-input-stream)

How to convert a byte[] to an InputStream using plain Java or Guava.[Read more](/web/20220831112445/https://www.baeldung.com/convert-byte-array-to-input-stream) →

首先，我们来看看将`String`转换成`byte`数组的各种方法。然后我们再反过来看类似的操作。

## 2.将`String`转换为`Byte`数组

在 Java 中，`String`被存储为一个 Unicode 字符数组。为了将它转换成一个`byte`数组，我们将字符序列转换成一个字节序列。对于这个翻译，**我们使用了一个 [`Charset`](https://web.archive.org/web/20220831112445/https://docs.oracle.com/en/java/javase/12/docs/api/java.base/java/nio/charset/Charset.html) 的实例。这个类指定了一个`char`序列和一个`byte`序列**之间的映射。

我们将上述过程称为`encoding`。

在 Java 中，我们可以用多种方式将一个`String`编码成一个`byte`数组。让我们用例子详细地看一下它们中的每一个。

### 2.1.使用`String.getBytes()`

**`String`类提供了三个重载的`getBytes`方法将一个`String`编码成`a byte`数组**:

*   [`getBytes()`](https://web.archive.org/web/20220831112445/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/String.html#getBytes(java.nio.charset.Charset))–使用平台的默认字符集编码
*   [`getBytes (String charsetName)`](https://web.archive.org/web/20220831112445/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/String.html#getBytes(java.lang.String))–使用命名字符集编码
*   [`getBytes (Charset charset)`](https://web.archive.org/web/20220831112445/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/String.html#getBytes(java.nio.charset.Charset))–使用提供的字符集编码

首先，**让我们使用平台的默认字符集编码一个字符串:**

```java
String inputString = "Hello World!";
byte[] byteArrray = inputString.getBytes();
```

上述方法依赖于平台，因为它使用平台的默认字符集。我们可以通过调用`Charset.defaultCharset()`来获得这个字符集。

然后**让我们用一个命名的字符集编码一个字符串:**

```java
@Test
public void whenGetBytesWithNamedCharset_thenOK() 
  throws UnsupportedEncodingException {
    String inputString = "Hello World!";
    String charsetName = "IBM01140";

    byte[] byteArrray = inputString.getBytes("IBM01140");

    assertArrayEquals(
      new byte[] { -56, -123, -109, -109, -106, 64, -26,
        -106, -103, -109, -124, 90 },
      byteArrray);
}
```

如果不支持指定的字符集，这个方法抛出一个`UnsupportedEncodingException`。

如果输入包含字符集不支持的字符，则上述两个版本的行为是未定义的。相比之下，第三个版本使用字符集的默认替换字节数组来编码不支持的输入。

接下来，**让我们调用第三个版本的`the getBytes()`方法，并传递一个`Charset:`的实例**

```java
@Test
public void whenGetBytesWithCharset_thenOK() {
    String inputString = "Hello ਸੰਸਾਰ!";
    Charset charset = Charset.forName("ASCII");

    byte[] byteArrray = inputString.getBytes(charset);

    assertArrayEquals(
      new byte[] { 72, 101, 108, 108, 111, 32, 63, 63, 63,
        63, 63, 33 },
      byteArrray);
}
```

这里我们使用工厂方法 [`Charset.forName`](https://web.archive.org/web/20220831112445/https://docs.oracle.com/en/java/javase/12/docs/api/java.base/java/nio/charset/Charset.html#forName(java.lang.String)) 来获得`Charset`的一个实例。如果请求的字符集的名称无效，此方法将引发运行时异常。如果当前 JVM 支持该字符集，它还会抛出一个运行时异常。

然而，某些字符集保证在每个 Java 平台上都可用。 [`StandardCharsets`](https://web.archive.org/web/20220831112445/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/nio/charset/StandardCharsets.html) 类为这些字符集定义了常量。

最后，**让我们使用标准字符集之一进行编码:**

```java
@Test
public void whenGetBytesWithStandardCharset_thenOK() {
    String inputString = "Hello World!";
    Charset charset = StandardCharsets.UTF_16;

    byte[] byteArrray = inputString.getBytes(charset);

    assertArrayEquals(
      new byte[] { -2, -1, 0, 72, 0, 101, 0, 108, 0, 108, 0,
        111, 0, 32, 0, 87, 0, 111, 0, 114, 0, 108, 0, 100, 0, 33 },
      byteArrray);
}
```

因此，我们已经完成了对各种`getBytes`版本的审查。接下来，我们来看看`Charset`本身提供的方法。

### 2.2.使用`Charset.encode()`

**`Charset`类提供了`encode()`，这是一种将 Unicode 字符编码成字节的便捷方法。**该方法总是使用字符集的默认替换字节数组替换无效输入和不可映射字符。

**让我们用`encode`方法将一个`String`转换成一个`byte`数组:**

```java
@Test
public void whenEncodeWithCharset_thenOK() {
    String inputString = "Hello ਸੰਸਾਰ!";
    Charset charset = StandardCharsets.US_ASCII;

    byte[] byteArrray = charset.encode(inputString).array();

    assertArrayEquals(
      new byte[] { 72, 101, 108, 108, 111, 32, 63, 63, 63, 63, 63, 33 },
      byteArrray);
}
```

正如我们在上面看到的，不支持的字符已经被字符集的默认替换`byte` 63 所取代。

到目前为止，我们使用的方法是在内部使用 [`CharsetEncoder`](https://web.archive.org/web/20220831112445/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/nio/charset/CharsetEncoder.html) 类来执行编码。让我们在下一节检查这个类。

### 2.3.`CharsetEncoder`

**`CharsetEncoder`将 Unicode 字符转换成给定字符集**的字节序列。**此外，它提供了对编码过程的细粒度控制**。

让我们使用这个类将一个`String`转换成一个`byte`数组:

```java
@Test
public void whenUsingCharsetEncoder_thenOK()
  throws CharacterCodingException {
    String inputString = "Hello ਸੰਸਾਰ!";
    CharsetEncoder encoder = StandardCharsets.US_ASCII.newEncoder();
    encoder.onMalformedInput(CodingErrorAction.IGNORE)
      .onUnmappableCharacter(CodingErrorAction.REPLACE)
      .replaceWith(new byte[] { 0 });

    byte[] byteArrray = encoder.encode(CharBuffer.wrap(inputString))
                          .array();

    assertArrayEquals(
      new byte[] { 72, 101, 108, 108, 111, 32, 0, 0, 0, 0, 0, 33 },
      byteArrray);
}
```

这里我们通过调用一个`Charset`对象上的 `newEncoder`方法来创建一个`CharsetEncoder`的实例。

然后我们通过调用`onMalformedInput()`和`onUnmappableCharacter() `方法`.`来指定错误条件的动作。我们可以指定以下动作:

*   忽略–丢弃错误的输入
*   替换–替换错误的输入
*   报告——通过返回一个 [`CoderResult`](https://web.archive.org/web/20220831112445/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/nio/charset/CoderResult.html) 对象或抛出一个 [`CharacterCodingException`](https://web.archive.org/web/20220831112445/https://docs.oracle.com/en/java/javase/12/docs/api/java.base/java/nio/charset/CharacterCodingException.html) 来报告错误

此外，我们使用`replaceWith()`方法来指定替换的`byte`数组。

因此，我们已经完成了将字符串转换为字节数组的各种方法的回顾。接下来，我们来看反向操作。

## 3.将字节数组转换为字符串

**我们把一个`byte`数组转换成一个`String`的过程称为`decoding`** 。类似于编码，这个过程需要一个`Charset`。

然而，我们不能只使用任何字符集来解码一个字节数组。特别是，**我们应该使用将`String`编码到`byte`数组**中的字符集。

我们也可以用很多方法将字节数组转换成字符串。让我们详细检查一下它们。

### 3.1.使用`String`构造函数

**`String`类有几个构造函数，它们将一个`byte`数组作为输入**。它们都类似于`getBytes`方法，但工作原理相反。

所以**让我们使用平台的默认字符集**将一个字节数组转换成`String`

```java
@Test
public void whenStringConstructorWithDefaultCharset_thenOK() {
    byte[] byteArrray = { 72, 101, 108, 108, 111, 32, 87, 111, 114,
      108, 100, 33 };

    String string = new String(byteArrray);

    assertNotNull(string);
}
```

**注意，我们在这里没有断言任何关于解码后的字符串的内容。这是因为根据平台的默认字符集，它可能会解码出不同的内容。**

为此，我们一般应避免这种方法。

然后**让我们用一个命名的字符集来解码:**

```java
@Test
public void whenStringConstructorWithNamedCharset_thenOK()
    throws UnsupportedEncodingException {
    String charsetName = "IBM01140";
    byte[] byteArrray = { -56, -123, -109, -109, -106, 64, -26, -106,
      -103, -109, -124, 90 };

    String string = new String(byteArrray, charsetName);

    assertEquals("Hello World!", string);
}
```

如果指定的字符集在 JVM 上不可用，该方法将抛出异常。

接下来，**我们用一个`Charset`对象来做解码:**

```java
@Test
public void whenStringConstructorWithCharSet_thenOK() {
    Charset charset = Charset.forName("UTF-8");
    byte[] byteArrray = { 72, 101, 108, 108, 111, 32, 87, 111, 114,
      108, 100, 33 };

    String string = new String(byteArrray, charset);

    assertEquals("Hello World!", string);
}
```

最后，**让我们用一个标准的`Charset`来表示同样的:**

```java
@Test
public void whenStringConstructorWithStandardCharSet_thenOK() {
    Charset charset = StandardCharsets.UTF_16;

    byte[] byteArrray = { -2, -1, 0, 72, 0, 101, 0, 108, 0, 108, 0,
      111, 0, 32, 0, 87, 0, 111, 0, 114, 0, 108, 0, 100, 0, 33 };

    String string = new String(byteArrray, charset);

    assertEquals("Hello World!", string);
}
```

到目前为止，我们已经使用构造函数将一个`byte`数组转换成了一个`String`，现在我们将研究其他方法。

### 3.2.使用`Charset.decode()`

`Charset`类提供了将`ByteBuffer`转换为`String`的`decode()`方法:

```java
@Test
public void whenDecodeWithCharset_thenOK() {
    byte[] byteArrray = { 72, 101, 108, 108, 111, 32, -10, 111,
      114, 108, -63, 33 };
    Charset charset = StandardCharsets.US_ASCII;
    String string = charset.decode(ByteBuffer.wrap(byteArrray))
                      .toString();

    assertEquals("Hello �orl�!", string);
}
```

这里，**无效的输入被替换为字符集的默认替换字符。**

### 3.3.`CharsetDecoder`

注意，之前所有的内部解码方法都使用了 [`CharsetDecoder`](https://web.archive.org/web/20220831112445/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/nio/charset/CharsetDecoder.html) 类。**我们可以直接使用这个类对解码过程进行细粒度控制**:

```java
@Test
public void whenUsingCharsetDecoder_thenOK()
  throws CharacterCodingException {
    byte[] byteArrray = { 72, 101, 108, 108, 111, 32, -10, 111, 114,
      108, -63, 33 };
    CharsetDecoder decoder = StandardCharsets.US_ASCII.newDecoder();

    decoder.onMalformedInput(CodingErrorAction.REPLACE)
      .onUnmappableCharacter(CodingErrorAction.REPLACE)
      .replaceWith("?");

    String string = decoder.decode(ByteBuffer.wrap(byteArrray))
                      .toString();

    assertEquals("Hello ?orl?!", string);
}
```

这里我们用“？”替换无效输入和不支持的字符。

如果我们希望在输入无效时得到通知，我们可以更改`decoder`:

```java
decoder.onMalformedInput(CodingErrorAction.REPORT)
  .onUnmappableCharacter(CodingErrorAction.REPORT)
```

## 4.结论

在本文中，我们研究了多种将`String`转换为字节数组的方法，反之亦然。我们应该根据输入数据以及无效输入所需的控制级别来选择适当的方法。

像往常一样，完整的源代码可以在 GitHub 上找到[。](https://web.archive.org/web/20220831112445/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-string-conversions-2)