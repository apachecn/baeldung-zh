# 输入流的 Java 字节数组

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/convert-byte-array-to-input-stream>

## 1。概述

在这个快速教程中，我们将演示如何**将一个简单的`byte[]`转换成一个`InputStream`** ，首先使用普通 java，然后使用番石榴库。

本文是 Baeldung 上的“Java-**回到基础**系列的一部分。

## 2。使用 Java 转换

首先，让我们看看 Java 解决方案:

```java
@Test
public void givenUsingPlainJava_whenConvertingByteArrayToInputStream_thenCorrect() 
  throws IOException {
    byte[] initialArray = { 0, 1, 2 };
    InputStream targetStream = new ByteArrayInputStream(initialArray);
}
```

## 3。使用番石榴进行转换

接下来——让我们将字节数组包装到番石榴树`ByteSource`中——然后让我们**得到流**:

```java
@Test
public void givenUsingGuava_whenConvertingByteArrayToInputStream_thenCorrect() 
  throws IOException {
    byte[] initialArray = { 0, 1, 2 };
    InputStream targetStream = ByteSource.wrap(initialArray).openStream();
}
```

这就是从一个字节数组中打开一个`InputStream`的简单方法。