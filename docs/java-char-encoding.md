# 字符编码指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-char-encoding>

## 1。概述

在本教程中，我们将讨论字符编码的基础知识以及如何在 Java 中处理它。

## 2.字符编码的重要性

我们经常不得不处理属于多种语言的文本，这些语言具有不同的书写脚本，如拉丁语或阿拉伯语。每种语言中的每个字符都需要以某种方式映射到一组 1 和 0。真的，计算机能正确处理我们所有的语言是一个奇迹。

为了正确地做到这一点，我们需要考虑字符编码。不这样做往往会导致数据丢失甚至安全漏洞。

为了更好地理解这一点，让我们定义一个用 Java 解码文本的方法:

```java
String decodeText(String input, String encoding) throws IOException {
    return 
      new BufferedReader(
        new InputStreamReader(
          new ByteArrayInputStream(input.getBytes()), 
          Charset.forName(encoding)))
        .readLine();
}
```

注意，我们在这里输入的文本使用默认的平台编码。

**如果我们用`input`作为“外观模式是一个软件设计模式”来运行这个方法`encoding`为“US-ASCII”**，它将输出:

```java
The fa��ade pattern is a software design pattern.
```

嗯，和我们预想的不太一样。

哪里出了问题？我们将在本教程的剩余部分尝试理解并纠正这一点。

## 3。基础知识

不过，在深入探讨之前，让我们快速回顾三个术语:`encoding`、`charsets`和`code point`。

### 3.1。编码

计算机只能理解像`1`和`0`这样的二进制表示。处理其他任何东西都需要某种从现实世界的文本到其二进制表示的映射。**这种映射就是我们所知的`character encoding`，或者简称为`encoding`** 。

比如我们消息中的第一个字母“T”，在 US-ASCII 中`encodes `为“01010100”。

### 3.2。字符集

根据字符所包含的字符，字符到其二进制表示的映射可以有很大的不同。在实际使用中，映射中包含的字符数量可以从几个到所有字符不等。**映射定义中包含的字符集正式称为`charset`** 。

例如， [ASCII 有 128 个字符的字符集](https://web.archive.org/web/20220628161825/http://ee.hawaii.edu/~tep/EE160/Book/chap4/subsection2.1.1.1.html)。

### 3.3。代码点

代码点是将字符与其实际编码分开的抽象概念。 **A `code point`是对特定字符的整数引用。**

我们可以用普通的十进制或者十六进制或者八进制来表示整数本身。为了便于引用大量数据，我们使用了交替基数。

例如，我们的消息中的第一个字母 T 在 Unicode 中有一个代码点“U+0054”(或者十进制的 84)。

## 4。理解编码方案

根据编码的字符数量，字符编码可以采用多种形式。

编码的字符数与每个表示的长度直接相关，通常以字节数来衡量。有更多的字符要编码意味着需要更长的二进制表示。

今天让我们来看看实践中一些流行的编码方案。

### 4.1。单字节编码

最早的编码方案之一，称为 [ASCII](/web/20220628161825/https://www.baeldung.com/cs/ascii-code) (美国信息交换标准代码)使用单字节编码方案。这实质上意味着**ASCII 中的每个字符都用 7 位二进制数表示。**这仍然会在每个字节中留出一位空间！

ASCII 的 128 个字符集包括小写和大写的英文字母、数字以及一些特殊字符和控制字符。

让我们在 Java 中定义一个简单的方法来显示特定编码方案下字符的二进制表示:

```java
String convertToBinary(String input, String encoding) 
      throws UnsupportedEncodingException {
    byte[] encoded_input = Charset.forName(encoding)
      .encode(input)
      .array();  
    return IntStream.range(0, encoded_input.length)
        .map(i -> encoded_input[i])
        .mapToObj(e -> Integer.toBinaryString(e ^ 255))
        .map(e -> String.format("%1$" + Byte.SIZE + "s", e).replace(" ", "0"))
        .collect(Collectors.joining(" "));
}
```

现在，字符' T '在 US-ASCII 中的码位是 84(ASCII 在 Java 中被称为 US-ASCII)。

如果我们使用我们的效用方法，我们可以看到它的二进制表示:

```java
assertEquals(convertToBinary("T", "US-ASCII"), "01010100");
```

正如我们所料，这是字符“T”的 7 位二进制表示。

最初的 ASCII 码没有使用每个字节的最高有效位。与此同时，ASCII 留下了相当多的没有表示的字符，尤其是非英语语言。

这导致了利用未使用位并包括额外的 128 个字符的努力。

随着时间的推移，ASCII 编码方案出现了多种变体并被采用。这些被笼统地称为“ASCII 扩展”。

许多 ASCII 扩展取得了不同程度的成功，但是很明显，这对于广泛采用来说还不够好，因为许多字符仍然没有被表示。

更流行的 ASCII 扩展之一是 ISO-8859-1 ，也被称为“ISO Latin 1”。

### 4.2。多字节编码

随着容纳越来越多字符的需求的增长，像 ASCII 这样的单字节编码方案是不可持续的。

这产生了多字节编码方案，其具有更好的容量，尽管代价是增加了空间需求。

BIG5 和 SHIFT-JIS 是**多字节字符编码方案的例子，它开始使用一个和两个字节来表示更宽的字符集**。其中大部分是为了表示汉字和类似文字的需要而创建的，这些文字有大量的字符。

现在让我们用`input`作为'来调用方法`convertToBinary`語，汉字，与`encoding`同为“Big5”:

```java
assertEquals(convertToBinary("語", "Big5"), "10111011 01111001");
```

上面的输出显示 Big5 编码使用两个字节来表示字符'語。

国际号码管理局维护着一个字符编码及其别名的综合列表。

## 5。Unicode

不难理解，虽然编码很重要，但解码对理解表示也同样重要。只有广泛使用一致或兼容的编码方案，这在实践中才有可能。

孤立地开发并在当地实践的不同编码方案开始变得具有挑战性。

这一挑战催生了称为 Unicode 的单一编码标准，它能够容纳世界上所有可能的字符。这包括正在使用的字符，甚至那些已经死亡！

嗯，那一定需要几个字节来存储每个字符？老实说是的，但是 Unicode 有一个巧妙的解决方案。

作为一种标准，Unicode 为世界上所有可能的字符定义了代码点。Unicode 中字符“T”的码位是十进制的 84。在 Unicode 中，我们通常称之为“U+0054 ”,也就是 U+后跟十六进制数。

我们使用十六进制作为 Unicode 码位的基础，因为有 1，114，112 个点，这是一个非常大的数字，便于用十进制进行交流！

如何将这些代码点编码成比特，这取决于 Unicode 中的特定编码方案。我们将在下面的小节中介绍一些编码方案。

### 5.1。UTF-32

UTF-32 是 Unicode 的一种编码方案，它使用四个字节来表示 Unicode 定义的每个代码点。显然，对每个字符使用四个字节是空间低效的。

让我们看看像' T '这样的简单字符是如何用 UTF-32 表示的。我们将使用之前介绍的方法`convertToBinary`:

```java
assertEquals(convertToBinary("T", "UTF-32"), "00000000 00000000 00000000 01010100");
```

上面的输出显示了使用四个字节来表示字符“T ”,其中前三个字节只是浪费空间。

### 5.2。UTF-8

UTF-8 是另一种 Unicode 编码方案，它采用可变长度的字节来编码。虽然它通常使用单个字节来编码字符，但如果需要，它可以使用更多的字节，从而节省空间。

让我们再次调用方法`convertToBinary`，输入为‘T ’,编码为‘UTF-8’:

```java
assertEquals(convertToBinary("T", "UTF-8"), "01010100");
```

输出完全类似于仅使用一个字节的 ASCII。事实上，UTF-8 完全向后兼容 ASCII。

让我们再次调用输入为'的方法`convertToBinary`語编码为“UTF-8”:

```java
assertEquals(convertToBinary("語", "UTF-8"), "11101000 10101010 10011110");
```

正如我们在这里看到的，UTF-8 使用三个字节来表示字符語。**这就是所谓的`variable-width encoding`。**

由于其空间效率，UTF-8 是网络上最常用的编码。

## 6。Java 中的编码支持

Java 支持多种编码及其相互转换。类`Charset`定义了一组[标准编码](https://web.archive.org/web/20220628161825/https://docs.oracle.com/en/java/javase/15/intl/supported-encodings.html#GUID-187BA718-195F-4C39-B0D5-F3FDF02C7205)，Java 平台的每个实现都必须支持这些编码。

这包括 US-ASCII、ISO-8859-1、UTF-8 和 UTF-16 等等。Java 的特定实现可能可选地支持附加编码。

Java 选择字符集的方式有一些微妙之处。让我们更详细地看一下它们。

### 6.1。默认字符集

Java 平台很大程度上依赖于一个名为`the default charset`的属性。**Java 虚拟机(JVM)在启动时确定默认字符集**。

这取决于运行 JVM 的底层操作系统的语言环境和字符集。例如，在 MacOS 上，默认字符集是 UTF-8。

让我们看看如何确定默认字符集:

```java
Charset.defaultCharset().displayName();
```

如果我们在 Windows 机器上运行这个代码片段，我们得到的输出是:

```java
windows-1252
```

现在，“windows-1252”是英语中 windows 平台的默认字符集，在这种情况下，它确定了在 Windows 上运行的 JVM 的默认字符集。

### 6.2。谁使用默认字符集？

许多 Java APIs 使用 JVM 确定的默认字符集。仅举几个例子:

*   `InputStreamReader`和`FileReader`
*   `OutputStreamWriter`和`FileWriter`
*   `Formatter`和`Scanner`
*   [`URLEncoder`和`URLDecoder`](/web/20220628161825/https://www.baeldung.com/java-url-encoding-decoding)

因此，这意味着如果我们在没有指定字符集的情况下运行我们的示例:

```java
new BufferedReader(new InputStreamReader(new ByteArrayInputStream(input.getBytes()))).readLine();
```

然后它会使用默认的字符集对其进行解码。

默认情况下，有几个 API 会做出同样的选择。

因此，默认字符集的重要性不容忽视。

### 6.3。默认字符集的问题

正如我们已经看到的，Java 中的默认字符集是在 JVM 启动时动态确定的。这使得该平台在跨不同操作系统使用时不太可靠或容易出错。

例如，如果我们跑

```java
new BufferedReader(new InputStreamReader(new ByteArrayInputStream(input.getBytes()))).readLine();
```

在 macOS 上，它将使用 UTF-8。

如果我们在 Windows 上尝试相同的代码片段，它将使用 Windows-1252 来解码相同的文本。

或者，想象一下在 macOS 上写一个文件，然后在 Windows 上读取同一个文件。

不难理解，由于不同的编码方案，这可能会导致数据丢失或损坏。

### 6.4。我们能覆盖默认字符集吗？

Java 中默认字符集的确定导致了两个系统属性:

*   `file.encoding`:该系统属性的值是默认字符集的名称
*   `sun.jnu.encoding`:该系统属性的值是编码/解码文件路径时使用的字符集的名称

现在，通过命令行参数覆盖这些系统属性是很直观的:

```java
-Dfile.encoding="UTF-8"
-Dsun.jnu.encoding="UTF-8"
```

但是，需要注意的是，这些属性在 Java 中是只读的。文件中没有上述用法。覆盖这些系统属性可能不会产生预期或可预测的行为。

因此，**我们应该避免覆盖 Java** 中的默认字符集。

### 6.5。为什么 Java 没有解决这个问题？

有一个 [Java 增强提案(JEP)规定使用“UTF-8”作为 Java](https://web.archive.org/web/20220628161825/https://openjdk.java.net/jeps/8187041) 的默认字符集，而不是基于语言环境和操作系统字符集。

这个 JEP 现在处于草稿状态，当它(希望如此！)将解决我们之前讨论的大部分问题。

注意，像`java.nio.file.Files`中的新 API 不使用默认字符集。这些 API 中的方法使用 UTF-8 字符集而不是默认字符集来读写字符流。

### 6.6。在我们的程序中解决这个问题

我们通常应该**选择在处理文本时指定一个字符集，而不是依赖默认设置**。我们可以在处理字符到字节转换的类中显式声明我们想要使用的编码。

幸运的是，我们的例子已经指定了字符集。我们只需要选择正确的，剩下的就交给 Java 吧。

到目前为止，我们应该认识到，像‘c’这样的重音字符在编码模式 ASCII 中是不存在的，因此我们需要一个包含它们的编码。也许，UTF-8？

让我们尝试一下，我们现在将运行方法 *decodeText* ，输入相同，但编码为“UTF-8”:

```java
The façade pattern is a software-design pattern.
```

答对了。我们现在可以看到我们希望看到的输出。

这里我们在`InputStreamReader`的构造函数中设置了我们认为最适合我们需要的编码。这通常是在 Java 中处理字符和字节转换的最安全的方法。

类似地，`OutputStreamWriter`和许多其他 API 支持通过它们的构造函数设置编码方案。

### 6.7.`MalformedInputException`

当我们解码一个字节序列时，存在给定的`Charset`不合法的情况，或者它不是合法的 16 位 Unicode。换句话说，给定的字节序列在指定的`Charset`中没有映射。

当输入序列有畸形输入时，有三种预定义的策略(或`CodingErrorAction`):

*   `IGNORE` 将忽略畸形字符并恢复编码操作
*   `REPLACE`将替换输出缓冲区中的畸形字符，并恢复编码操作
*   `REPORT`会抛出一个`MalformedInputException`

`InputStreamReader`中默认解码器的`CharsetDecoder is REPORT,`和默认`malformedInputAction` 的默认`malformedInputAction`为`REPLACE.`

让我们定义一个解码函数，它接收一个指定的`Charset`、一个`CodingErrorAction` 类型和一个要解码的字符串:

```java
String decodeText(String input, Charset charset, 
  CodingErrorAction codingErrorAction) throws IOException {
    CharsetDecoder charsetDecoder = charset.newDecoder();
    charsetDecoder.onMalformedInput(codingErrorAction);
    return new BufferedReader(
      new InputStreamReader(
        new ByteArrayInputStream(input.getBytes()), charsetDecoder)).readLine();
}
```

因此，如果我们解码“外观模式是一种软件设计模式。”使用`US_ASCII`，每个策略的输出将会不同。首先，我们使用跳过非法字符的`CodingErrorAction.IGNORE`:

```java
Assertions.assertEquals(
  "The faade pattern is a software design pattern.",
  CharacterEncodingExamples.decodeText(
    "The façade pattern is a software design pattern.",
    StandardCharsets.US_ASCII,
    CodingErrorAction.IGNORE));
```

对于第二个测试，我们使用`CodingErrorAction.REPLACE`来代替非法字符:

```java
Assertions.assertEquals(
  "The fa��ade pattern is a software design pattern.",
  CharacterEncodingExamples.decodeText(
    "The façade pattern is a software design pattern.",
    StandardCharsets.US_ASCII,
    CodingErrorAction.REPLACE));
```

对于第三个测试，我们使用`CodingErrorAction.REPORT`导致抛出`MalformedInputException:`

```java
Assertions.assertThrows(
  MalformedInputException.class,
    () -> CharacterEncodingExamples.decodeText(
      "The façade pattern is a software design pattern.",
      StandardCharsets.US_ASCII,
      CodingErrorAction.REPORT));
```

## 7。编码很重要的其他地方

我们在编程时不仅仅需要考虑字符编码。在许多其他地方，文本可能最终出错。

在这些情况下，问题的最常见原因是文本从一种编码方案转换到另一种编码方案，从而可能导致数据丢失。

让我们快速浏览几个在编码或解码文本时可能会遇到问题的地方。

### 7.1。文本编辑器

在大多数情况下，文本编辑器是文本产生的地方。有许多流行的文本编辑器，包括 vi、记事本和 MS Word。大多数文本编辑器允许我们选择编码方案。因此，我们应该始终确保它们适合我们正在处理的文本。

### 7.2。文件系统

在编辑器中创建文本后，我们需要将它们存储在某个文件系统中。文件系统依赖于运行它的操作系统。大多数操作系统都固有地支持多种编码方案。但是，仍可能存在编码转换导致数据丢失的情况。

### 7.3。网络

使用文件传输协议(FTP)之类的协议通过网络传输文本时，也会涉及字符编码之间的转换。对于用 Unicode 编码的任何东西，最安全的方法是以二进制形式传输，以最大限度地降低转换过程中丢失的风险。然而，通过网络传输文本是数据损坏的一个不太常见的原因。

### 7.4。数据库

大多数流行的数据库，如 Oracle 和 MySQL，都支持在安装或创建数据库时选择字符编码方案。我们必须根据我们希望存储在数据库中的文本来选择。这是由于编码转换而导致文本数据损坏的最常见的地方之一。

### 7.5。浏览器

最后，在大多数 web 应用程序中，我们创建文本并通过不同的层传递它们，目的是在用户界面(如浏览器)中查看它们。在这里，我们必须选择正确的字符编码来正确显示字符。大多数流行的浏览器，如 Chrome、Edge，都允许通过它们的设置来选择字符编码。

## 8。结论

在本文中，我们讨论了编码在编程时如何成为一个问题。

我们进一步讨论了基础知识，包括编码和字符集。此外，我们经历了不同的编码方案及其用途。

我们还挑选了一个 Java 中不正确的字符编码用法的例子，并看到了如何纠正它。最后，我们讨论了其他一些与字符编码相关的常见错误场景。

和往常一样，这些例子的代码可以在 GitHub 的[上找到。](https://web.archive.org/web/20220628161825/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-string-operations-2)