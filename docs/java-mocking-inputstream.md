# 模仿 Java InputStream 对象

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-mocking-inputstream>

## 1.介绍

`InputStream`是一个用于处理数据的通用抽象类。数据可以来自非常不同的来源，但是使用类允许我们从来源中抽象出来，并从特定的来源独立地处理它。

然而，当我们编写测试时，我们实际上需要提供一些可靠的实现。在本教程中，我们将了解我们应该选择哪些可用的实现，或者什么时候编写自己的实现更好。

## 2.`InputStream`界面基础知识

在我们开始编写自己的代码之前，了解一下`InputStream`接口是如何构建的对我们来说是有好处的。幸运的是，这非常简单。**要实现一个简单的`InputStream,`，我们只需要考虑一个方法——**`**[read](https://web.archive.org/web/20221223190216/https://docs.oracle.com/en/java/javase/12/docs/api/java.base/java/io/InputStream.html#read()).**` 它不带参数，返回流的下一个字节作为`int`。如果`InputStream`已经结束，它返回-1，通知我们停止处理。

### 2.1.判例案件

在本教程中，我们将测试一种以`InputStream`的形式处理文本消息并返回已处理字节数的方法。然后，我们将断言读取了正确的字节数:

```java
int bytesCount = processInputStream(someInputStream);
assertThat(bytesCount).isEqualTo(expectedNumberOfBytes);
```

`processInputStream()`方法在内部做什么在这里不太相关，所以我们只是使用一个非常简单的实现:

```java
public class MockingInputStreamUnitTest { 
    int processInputStream(InputStream inputStream) throws IOException {
        int count = 0;
        while(inputStream.read() != -1) {
            count++;
        }
        return count;
    }
}
```

### 2.2.使用简单的实现

为了更好地理解`InputStream`是如何工作的，我们将编写一个带有硬编码消息的简单实现。除了消息之外，我们的实现将有一个索引，指向我们接下来应该读取消息的哪个字节。每次调用 read 方法时，我们将从消息中获取一个字节，然后递增索引。

在此之前，我们还需要检查我们是否已经从消息中读取了所有的字节。如果是这样，我们需要返回-1:

```java
public class MockingInputStreamUnitTest {

@Test
public void givenSimpleImplementation_shouldProcessInputStream() throws IOException {
    int byteCount = processInputStream(new InputStream() {
        private final byte[] msg = "Hello World".getBytes();
        private int index = 0;
        @Override
        public int read() {
            if (index >= msg.length) {
                return -1;
            }
            return msg[index++];
        }
    });
    assertThat(byteCount).isEqualTo(11);
}
```

## 3.使用`ByteArrayInputStream`

**如果我们绝对确定整个数据有效载荷将适合内存，最简单的选择是`ByteArrayInputStream`。**我们向构造函数提供一个字节数组，然后流逐个字节地遍历它，与上一节的例子类似:

```java
String msg = "Hello World";
int bytesCount = processInputStream(new ByteArrayInputStream(msg.getBytes()));
assertThat(bytesCount).isEqualTo(11);
```

## 4.使用`FileInputStream`

如果我们可以将我们的数据保存为一个文件，我们也可以以`FileInputStream`的形式加载它。**这种方法的优点是数据不会作为一个整体加载到内存中，而是在需要时从磁盘中读取。**如果我们将文件放在 resources 文件夹中，我们可以使用一个方便的`getResourceAsStream `方法直接在一行代码中从一个路径创建`InputStream`:

```java
InputStream inputStream = MockingInputStreamUnitTest.class.getResourceAsStream("/mockinginputstreams/msg.txt");
int bytesCount = processInputStream(inputStream);
assertThat(bytesCount).isEqualTo(11);
```

注意，在这个例子中，`InputStream`的实际实现将是`BufferedFileInputStream`。顾名思义，它读取更大的数据块并将它们存储在缓冲区中。因此，它限制了从磁盘读取的次数。

## 5.动态生成数据

有时我们想测试我们的系统在处理大量数据时是否能正常工作。我们可以只使用从磁盘加载的大文件，但是这种方法有一些严重的缺点。这不仅是潜在的空间浪费，而且像`git`这样的版本控制系统并不适合处理大的二进制文件。幸运的是，我们不需要事先掌握所有的数据。相反，我们可以即时生成它。

为了实现这一点，我们需要实现我们的`InputStream`。让我们从定义字段和构造函数开始:

```java
public class GeneratingInputStream extends InputStream {
    private final int desiredSize;
    private final byte[] seed;
    private int actualSize = 0;

    public GeneratingInputStream(int desiredSize, String seed) {
        this.desiredSize = desiredSize;
        this.seed = seed.getBytes();
    }
}
```

“desiredSize”变量将告诉我们何时应该停止生成数据。“种子”变量将是一个重复的数据块。最后，`“actualSize”`变量将帮助我们跟踪我们已经返回了多少字节。**我们需要它，因为我们实际上不保存任何数据。我们只返回“当前”字节。**

使用我们定义的变量，我们可以实现`read`方法:

```java
@Override
public int read() {
    if (actualSize >= desiredSize) {
        return -1;
    }
    return seed[actualSize++ % seed.length];
}
```

首先，我们检查是否达到了预期的大小。如果是，我们应该返回-1，这样流的消费者就知道停止读取。如果没有，我们应该从种子中返回一个字节。为了确定它应该是哪一个字节，我们使用[模操作符](/web/20221223190216/https://www.baeldung.com/modulo-java)来获得生成数据的实际大小除以种子长度的余数。

## 6.摘要

在本教程中，我们研究了如何在测试中处理`InputStreams`。我们学习了该类是如何构建的，以及我们可以在各种场景中使用哪些实现。最后，我们学习了如何编写自己的实现来动态生成数据。

和往常一样，代码示例可以在 GitHub 上找到。