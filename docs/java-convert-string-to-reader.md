# Java–字符串到阅读器

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-convert-string-to-reader>

在这个快速教程中，我们将看看如何**将一个字符串转换成一个`Reader`** ，首先使用普通 Java，然后是 Guava，最后是 Commons IO 库。

本文是 Baeldung 网站上的“Java 回归基础”系列文章的一部分。

## 1。用普通 Java

让我们从 Java 解决方案开始:

```
@Test
public void givenUsingPlainJava_whenConvertingStringIntoReader_thenCorrect() throws IOException {
    String initialString = "With Plain Java";
    Reader targetReader = new StringReader(initialString);
    targetReader.close();
}
```

如您所见，`StringReader`可用于这种简单的转换。

## 2。有番石榴

接下来——番石榴溶液:

```
@Test
public void givenUsingGuava_whenConvertingStringIntoReader_thenCorrect() throws IOException {
    String initialString = "With Google Guava";
    Reader targetReader = CharSource.wrap(initialString).openStream();
    targetReader.close();
}
```

我们在这里使用了通用的`CharSource`抽象，允许我们从中打开一个阅读器。

## 3。使用 Apache Commons IO

最后，这是 Commons IO 解决方案，也使用了现成的`Reader`实现:

```
@Test
public void givenUsingCommonsIO_whenConvertingStringIntoReader_thenCorrect() throws IOException {
    String initialString = "With Apache Commons IO";
    Reader targetReader = new CharSequenceReader(initialString);
    targetReader.close();
}
```

现在我们有了——**3 种非常简单的方法将一个字符串转换成 Java 中的阅读器。确保在 GitHub 上查看样本[。](https://web.archive.org/web/20221013193920/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-io-conversions)**