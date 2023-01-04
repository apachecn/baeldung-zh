# 在 Java 中向字符串添加换行符

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-string-newline>

## 1.概观

字符串格式化和生成文本输出经常在编程过程中出现。在许多情况下，需要在字符串中添加新的一行来格式化输出。

让我们讨论如何使用换行符。

## 延伸阅读:

## [检查 Java 中的空字符串](/web/20221128040625/https://www.baeldung.com/java-blank-empty-strings)

Check out some simple ways in Java to test if a string is blank or empty.[Read more](/web/20221128040625/https://www.baeldung.com/java-blank-empty-strings) →

## [检查字符串是否包含子串](/web/20221128040625/https://www.baeldung.com/java-string-contains-substring)

Explore various ways to search for a substring in a String with performance benchmarks[Read more](/web/20221128040625/https://www.baeldung.com/java-string-contains-substring) →

## 2.在字符串中添加换行符

操作系统有特殊字符来表示新行的开始。例如，**在 Linux 中一个新的行用“`\n”`** `,` 表示，也叫做`Line Feed`。**在 Windows 中，新的一行用`\r\n”`** 来表示，有时称为`Carriage Return `和`Line Feed`，或`CRLF`。

**在 Java 中添加一个新行非常简单，只需在我们的字符串末尾包含“`\n”`、“`\r”,`或“`\` r `\n” `。**

### 2.1.使用 CRLF 换行符

对于这个例子，我们想用两行文本创建一个段落。具体来说，我们希望`line2`出现在`line1`之后的新行中。

对于基于 Unix/Linux/新 Mac 的操作系统，我们可以使用“`\n”:`

```java
String line1 = "Humpty Dumpty sat on a wall.";
String line2 = "Humpty Dumpty had a great fall.";
String rhyme = line1 + "\n" + line2;
```

如果我们在基于 Windows 的操作系统上，我们可以使用“`\r\n”:`

```java
rhyme = line1 + "\r\n" + line2;
```

对于旧的基于 Mac 的操作系统，我们可以使用“`\r”:`

```java
rhyme = line1 + "\r" + line2;
```

我们已经演示了添加新行的三种方法，但不幸的是，它们依赖于平台。

### 2.2.使用独立于平台的行分隔符

当我们希望代码独立于平台时，我们可以使用系统定义的常量。

**例如，使用`System.lineSeparator()`给出一个行分隔符:**

```java
rhyme = line1 + System.lineSeparator() + line2;
```

或者我们也可以用`System.getProperty(“line.separator”)` :

```java
rhyme = line1 + System.getProperty("line.separator") + line2;
```

### 2.3.使用独立于平台的换行符

尽管行分隔符提供了平台独立性，但它们迫使我们连接字符串。

如果我们使用类似于 [`System.out.printf`](/web/20221128040625/https://www.baeldung.com/java-printstream-printf) 或 [`String.format`](/web/20221128040625/https://www.baeldung.com/string/format) 的东西，那么**平台独立换行符`%n`可以直接在字符串**内使用:

```java
rhyme = "Humpty Dumpty sat on a wall.%nHumpty Dumpty had a great fall.";
```

这与在我们的字符串中包含`System.lineSeparator()`是一样的，但是我们不需要将字符串分成多个部分。

## 3.在 HTML 页面中添加换行符

假设我们正在创建一个字符串，它是 HTML 页面的一部分。在这种情况下，我们可以添加一个 HTML break 标签`<br>`。

**我们也可以使用 Unicode 字符`“& #13;” `(回车)和`“& #10;” `(换行)。**虽然这些字符可以工作，但它们并不像我们期望的那样可以跨所有平台工作。相反，最好使用`<br>`来换行。

此外，我们可以在一些 HTML 元素中使用`“\n”`来换行。

总的来说，这是 HTML 中的三种换行方法。我们可以根据使用的 HTML 标签来决定使用哪一个。

### 3.1.HTML 中断标签

我们可以使用 HTML break 标签`<br>`来换行:

```java
rhyme = line1 + "<br>" + line2;
```

用于换行的`<br>`标签可以在几乎所有的 HTML 元素中使用，比如`<body>`、`<p>`、`<pre>,`等等。但是，请注意，它在`<textarea>`标签中不起作用。

### 3.2.换行符

如果文本包含在`<pre>`或`<textarea>`标签中，我们可以使用`‘\n'`换行:

```java
rhyme = line1 + "\n" + line2;
```

### 3.3.Unicode 字符

最后，我们可以使用 Unicode 字符`“& #13;” `(回车)和`“& #10;” `(换行)来换行。例如，在`<textarea>`标签中，我们可以使用以下两者之一:

```java
rhyme = line1 + "
" + line2;
rhyme = line1 + "
" + line2; 
```

对于`<pre>`标签，下面的两行都可以使用:

```java
rhyme = line1 + "
" + line2;
rhyme = line1 + "

" + line2; 
```

## 4. `\n`和`\r`的区别

`\r` 和`\n`分别是用 ASCII 值 13 (CR)和 10 (LF)表示的字符。**它们** **都代表两条线**、**之间的中断，但是操作系统使用它们的方式不同。**

在 Windows 上，两个字符的序列用于开始新的一行，CR 紧接着 LF。相反，在类 Unix 系统上，只使用 LF。

编写 Java 应用程序时，我们必须注意我们使用的换行符，因为应用程序的行为会因运行的操作系统而异。

最安全和最具交叉兼容性的选择是这样使用`System.lineSeparator().` ，我们不必考虑操作系统。

## 5.结论

在本文中，我们讨论了如何在 Java 中给字符串添加换行符。

我们还看到了如何使用`System.lineSeparator()`和`System.getProperty(“line.separator”)`为一个新行编写平台无关的代码。

最后，我们总结了如何在生成 HTML 页面时添加新行。

这篇文章的完整实现可以在 GitHub 上找到[。](https://web.archive.org/web/20221128040625/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-string-operations)