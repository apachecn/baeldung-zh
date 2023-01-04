# Java 中的 SequenceInputStream 类

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-sequenceinputstream>

## 1.概观

在本教程中，我们将学习如何在 Java 中使用`SequenceInputStream`类。特别是，**这个类有助于连续读取一系列文件或流**。

关于 Java IO 和其他相关 Java 类的更多基础知识，我们可以阅读 [Java IO 教程](/web/20220627175551/https://www.baeldung.com/java-io)。

## 2.使用`SequenceInputStream`类

`SequenceInputStream`以两个或两个以上的`InputStream`物体为源。它按照给定的顺序从一个源读取另一个源。当它完成从第一个`InputStream`的读取时，它自动从第二个开始读取。这个过程一直持续到它从所有的源流中读取完为止。

### 2.1.对象创建

我们可以使用两个`InputStream`对象初始化一个`SequenceInputStream`T2:

```java
InputStream first = new FileInputStream(file1);
InputStream second = new FileInputStream(file2);
SequenceInputStream sequenceInputStream = new SequenceInputStream(first, second);
```

我们也可以使用`InputStream`对象的`Enumeration`实例化它**:**

```java
Vector<InputStream> inputStreams = new Vector<>();
for (String fileName: fileNames) {
    inputStreams.add(new FileInputStream(fileName));
}
sequenceInputStream = new SequenceInputStream(inputStreams.elements());
```

### 2.2.从溪流中阅读

`SequenceInputStream`提供了两种从输入源读取的简单方法。第一种方法读取单个字节，而第二种方法读取字节数组。

为了**读取单个字节的数据**，我们使用`read()`方法:

```java
int byteValue = sequenceInputStream.read();
```

在上面的示例中，read 方法从流中返回下一个字节(0–255)值。如果流结束了，那么它返回-1 。

我们还可以**读取一个字节数组**:

```java
byte[] bytes = new byte[100];
sequenceInputStream.read(bytes, 0, 50);
```

在上面的例子中，它读取`50`字节，并从索引`0`开始放置它们。

### 2.3.显示序列读数的示例

将两个字符串作为输入源来演示读取序列:

```java
InputStream first = new ByteArrayInputStream("One".getBytes());
InputStream second = new ByteArrayInputStream("Magic".getBytes());
SequenceInputStream sequenceInputStream = new SequenceInputStream(first, second);
StringBuilder stringBuilder = new StringBuilder();
int byteValue;
while ((byteValue = sequenceInputStream.read()) != -1) {
    stringBuilder.append((char) byteValue);
}
assertEquals("OneMagic", stringBuilder.toString());
```

从上面的例子中，如果我们打印`stringBuilder.toString()`,它会显示以下输出:

```java
OneMagic
```

## 3.结论

在这篇短文中，我们看到了如何使用`SequenceInputStream`。**它只是将所有底层的输入流合并成一个单独的流**。

完整的代码示例可以在 GitHub 上找到[。](https://web.archive.org/web/20220627175551/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-io-4)