# Java 字符串到输入流

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/convert-string-to-input-stream>

## 1。概述

在这个快速教程中，我们将看看如何使用普通 Java、Guava 和 Apache Commons IO 库将标准字符串转换成 T2。

本教程是 Baeldung 网站上[Java 基础系列](/web/20220812070143/https://www.baeldung.com/java-tutorial "The Java Guide on IO and Collections")的一部分。

## 2。用普通 Java 转换

让我们从一个使用 Java 进行转换的简单例子开始——使用一个中间的`byte`数组:

```
@Test
public void givenUsingPlainJava_whenConvertingStringToInputStream_thenCorrect() 
  throws IOException {
    String initialString = "text";
    InputStream targetStream = new ByteArrayInputStream(initialString.getBytes());
}
```

`getBytes()`方法使用平台的默认字符集对这个`String`进行编码，因此为了避免不良行为，我们可以使用`getBytes(Charset charset)`和**来控制编码过程。**

## 3。用番石榴转换

番石榴没有提供直接的转换方法，但是允许我们从`String`中得到一个`CharSource`，并轻松地将其转换为`ByteSource`。

那么就很容易获得`InputStream`:

```
@Test
public void givenUsingGuava_whenConvertingStringToInputStream_thenCorrect() 
  throws IOException {
    String initialString = "text";
    InputStream targetStream = 
      CharSource.wrap(initialString).asByteSource(StandardCharsets.UTF_8).openStream();
}
```

`asByteSource` 方法实际上被标记为`@Beta`。这意味着它可以在未来的番石榴发布中删除。我们需要记住这一点。

## 4。用 Commons IO 转换

最后，Apache Commons IO 库提供了一个优秀的直接解决方案:

```
@Test
public void givenUsingCommonsIO_whenConvertingStringToInputStream_thenCorrect() 
  throws IOException {
    String initialString = "text";
    InputStream targetStream = IOUtils.toInputStream(initialString);
}
```

注意，在这些例子中，我们让输入流保持打开状态，所以不要忘记关闭它。

## 5.结论

在本文中，我们展示了从简单字符串中获取`InputStream`的三种简单而简洁的方法。

和往常一样，完整的源代码可以在 GitHub 上找到[。](https://web.archive.org/web/20220812070143/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-io-conversions-2)