# Java–写入器的字节数组

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-convert-byte-array-to-writer>

## 1。概述

在这个快速教程中，我们将讨论如何使用普通 Java、Guava 和 Commons IO 将 **byte[]转换为 Writer** 。

## 2。用普通 Java

让我们从一个简单的 Java 解决方案开始:

```java
@Test
public void givenPlainJava_whenConvertingByteArrayIntoWriter_thenCorrect() 
  throws IOException {
    byte[] initialArray = "With Java".getBytes();
    Writer targetWriter = new StringWriter().append(new String(initialArray));

    targetWriter.close();

    assertEquals("With Java", targetWriter.toString());
}
```

注意，我们通过中间的`String`将`byte[]`转换成了`Writer`。

## 3。有番石榴

接下来，让我们看看一个更复杂的番石榴解决方案:

```java
@Test
public void givenUsingGuava_whenConvertingByteArrayIntoWriter_thenCorrect() 
  throws IOException {
    byte[] initialArray = "With Guava".getBytes();

    String buffer = new String(initialArray);
    StringWriter stringWriter = new StringWriter();
    CharSink charSink = new CharSink() {
        @Override
        public Writer openStream() throws IOException {
            return stringWriter;
        }
    };
    charSink.write(buffer);

    stringWriter.close();

    assertEquals("With Guava", stringWriter.toString());
}
```

注意，在这里，我们通过使用一个`CharSink`将`byte[]`转换成了一个`Writer`。

## 4。带公共 IO

最后，让我们检查一下我们的 Commons IO 解决方案:

```java
@Test
public void givenUsingCommonsIO_whenConvertingByteArrayIntoWriter_thenCorrect() 
  throws IOException {
    byte[] initialArray = "With Commons IO".getBytes();

    Writer targetWriter = new StringBuilderWriter(
      new StringBuilder(new String(initialArray)));

    targetWriter.close();

    assertEquals("With Commons IO", targetWriter.toString());
}
```

注意:我们使用一个`StringBuilder`将我们的`byte[]`转换为`StringBuilderWriter`。

## 5。结论

在这篇简短扼要的教程中，我们举例说明了将`byte[]`转换成`Writer`的 3 种不同方法。

本文的代码可以在 GitHub 库中找到。