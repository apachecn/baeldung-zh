# Java 中的校验和

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-checksums>

## 1.概观

在这篇小文章中，我们将简要解释什么是校验和，并展示如何使用 Java 的一些内置特性来计算校验和。

## 2.校验和及常用算法

本质上，校验和是二进制数据流的简化表示。

校验和通常用于网络编程，以检查是否收到完整的消息。收到新消息后，可以重新计算校验和，并与收到的校验和进行比较，以确保没有任何位丢失。此外，它们还可以用于文件管理，例如，比较文件或检测更改。

**创建校验和有几种常见的算法，如 Adler32 和 CRC32** 。这些算法通过将数据或字节序列转换成更小的字母和数字序列来工作。它们的设计使得输入中的任何微小变化都会导致计算出的校验和有很大不同。

我们来看看 Java 对 CRC32 的支持。请注意，虽然 CRC32 可能对校验和有用，但不建议用于安全操作，如[散列密码](/web/20221208143830/https://www.baeldung.com/java-password-hashing)。

## 3.字符串或字节数组的校验和

我们需要做的第一件事是获得校验和算法的输入。

如果我们从一个`String`开始，我们可以使用`getBytes()` 方法从`String` 中[得到一个字节数组:](/web/20221208143830/https://www.baeldung.com/java-string-to-byte-array)

```
String test = "test";
byte[] bytes = test.getBytes();
```

接下来，我们可以使用字节数组计算校验和:

```
public static long getCRC32Checksum(byte[] bytes) {
    Checksum crc32 = new CRC32();
    crc32.update(bytes, 0, bytes.length);
    return crc32.getValue();
}
```

这里，我们使用的是 Java 内置的 [`CRC32`](https://web.archive.org/web/20221208143830/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/zip/CRC32.html) 类。一旦类被实例化，我们使用`update `方法用来自输入的字节更新`Checksum` 实例。

简单地说，`update `方法替换了由`CRC32` `Object`保存的字节——这有助于代码重用，并消除了创建新的`Checksum. `实例的需要。`CRC32`类提供了一些被覆盖的方法来替换整个字节数组或其中的几个字节。

最后，在设置字节`,` 后，我们用`getValue `方法导出校验和。

## 4.来自`InputStream`的校验和

当处理二进制数据的较大数据集时，上述方法不是很节省内存，因为每个字节都被加载到内存中。

当我们有一个`InputStream`时，我们可以选择使用`CheckedInputStream` 来创建我们的校验和**。**通过使用这种方法，我们可以定义一次处理多少字节。

在这个例子中，我们一次处理给定数量的字节，直到到达流的末尾。

校验和值可从`CheckedInputStream`获得:

```
public static long getChecksumCRC32(InputStream stream, int bufferSize) 
  throws IOException {
    CheckedInputStream checkedInputStream = new CheckedInputStream(stream, new CRC32());
    byte[] buffer = new byte[bufferSize];
    while (checkedInputStream.read(buffer, 0, buffer.length) >= 0) {}
    return checkedInputStream.getChecksum().getValue();
}
```

## 5.结论

在本教程中，我们看看如何使用 Java 的 CRC32 支持从字节数组和`InputStream`生成校验和。

和往常一样，代码可以在 GitHub 的[上获得。](https://web.archive.org/web/20221208143830/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-security-2)