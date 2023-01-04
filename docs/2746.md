# 在 Java 中将 ByteBuffer 转换为字符串

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-bytebuffer-to-string>

## 1.概观

`ByteBuffer`是 [`java.nio`](/web/20220930184110/https://www.baeldung.com/java-nio-2-file-api) 套装中众多有益职业之一。它用于从通道读取数据，并直接将数据写入通道。

在这个简短的教程中，我们将学习如何在 Java 中将`ByteBuffer`转换成`String`**。**

## 2.将一个`ByteBuffer`转换为`String`

将一个`ByteBuffer` 转换成一个`String`的过程就是解码。这个过程需要一个`Charset`。

有三种方法可以将`ByteBuffer`转换为`String`:

*   从`bytebuffer.array()`创建新的`String`
*   从`bytebuffer.get(bytes)`创建新的`String`
*   使用`charset.decode()`

我们将用一个简单的例子来展示将一个`ByteBuffer`转换成一个`String`的所有三种方式。

## 3.实际例子

### 3.1.从`bytebuffer.array()`创建新的`String`

第一步是从`ByteBuffer`获取字节数组。为此，我们将调用 `ByteBuffer.array()`方法。这将返回支持数组。

然后，我们可以调用`String`构造函数，它接受一个字节数组和字符编码来创建新的`String`:

```
@Test
public void convertUsingNewStringFromBufferArray_thenOK() {
    String content = "baeldung";
    ByteBuffer byteBuffer = ByteBuffer.wrap(content.getBytes());

    if (byteBuffer.hasArray()) {
        String newContent = new String(byteBuffer.array(), charset);

        assertEquals(content, newContent);
    }
}
```

### 3.2.从`bytebuffer.get(bytes)`创建新的`String`

在 Java 中，我们可以使用`new String(bytes, charset)`将一个`byte[]`转换成一个`String`。

对于字符数据，我们可以使用`UTF_8 charset` 将一个`byte[]`转换成一个`String`。然而，当`byte[]`保存非文本二进制数据时，最佳实践是将`byte[]`转换为 [Base64 编码的](/web/20220930184110/https://www.baeldung.com/java-base64-encode-and-decode) `String`:

```
@Test
public void convertUsingNewStringFromByteBufferGetBytes_thenOK() {
    String content = "baeldung";
    ByteBuffer byteBuffer = ByteBuffer.wrap(content.getBytes());

    byte[] bytes = new byte[byteBuffer.remaining()];
    byteBuffer.get(bytes);
    String newContent = new String(bytes, charset);

    assertEquals(content, newContent);
}
```

### 3.3.使用`charset.decode()`

这是将`ByteBuffer`转换成`String`的最简单的方法，没有任何问题:

```
@Test
public void convertUsingCharsetDecode_thenOK() {
    String content = "baeldung";
    ByteBuffer byteBuffer = ByteBuffer.wrap(content.getBytes());

    String newContent = charset.decode(byteBuffer).toString();

    assertEquals(content, newContent);
} 
```

## 4.结论

在本教程中，我们学习了在`Java`中将`ByteBuffer`转换为`String`的三种方法。记住使用正确的[字符编码](/web/20220930184110/https://www.baeldung.com/java-char-encoding)，在我们的例子中，我们使用了`UTF-8`。

与往常一样，GitHub 上的[提供了示例的完整源代码。](https://web.archive.org/web/20220930184110/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-string-conversions-2)