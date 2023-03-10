# Java 扫描器 hasNext()与 hasNextLine()

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-scanner-hasnext-vs-hasnextline>

## 1.概观

`[Scanner](https://web.archive.org/web/20221206031835/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/Scanner.html)`类是一个方便的工具，可以使用正则表达式解析原始类型和字符串，并被引入到 Java 5 的`java.util`包中。

在这个简短的教程中，我们将讨论它的`hasNext()` 和 `hasNextLine()` 方法。尽管这两种方法初看起来非常相似，但它们实际上做的是完全不同的检查。

您还可以在这里的快速指南中阅读更多关于多功能扫描仪类别[的信息。](/web/20221206031835/https://www.baeldung.com/java-scanner)

## 2.`hasNext()`

### 2.1.基本用法

`The [hasNext()](https://web.archive.org/web/20221206031835/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/Scanner.html#hasNext())`方法检查`Scanner`的输入中是否有另一个令牌。**一个`Scanner`使用一个定界符模式将它的输入分成几个记号，默认情况下这个定界符模式匹配[空格](https://web.archive.org/web/20221206031835/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/Character.html#isWhitespace(char))。**也就是说，`hasNext()` 检查输入并返回`true`，如果它有另一个非空白字符。

我们还应该注意一些关于默认分隔符的细节:

*   空白不仅包括空格字符，还包括制表符(`\t`)、换行符(`\n`)和[甚至更多的字符](https://web.archive.org/web/20221206031835/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/Character.html#isWhitespace(char))
*   连续的空白字符被视为单个分隔符
*   输入结束时的空行不被打印——也就是说，`hasNext()`返回`false`作为空行

让我们看一个例子，看看`hasNext()` 是如何使用默认分隔符的。首先，我们将准备一个输入字符串来帮助我们探索 S `canner`的解析结果:

```java
String INPUT = new StringBuilder()
    .append("magic\tproject\n")
    .append("     database: oracle\n")
    .append("dependencies:\n")
    .append("spring:foo:bar\n")
    .append("\n")  // Note that the input ends with a blank line
    .toString();
```

接下来，让我们解析输入并打印结果:

```java
Scanner scanner = new Scanner(INPUT);
while (scanner.hasNext()) {
    log.info(scanner.next());
}
log.info("--------OUTPUT--END---------") 
```

如果我们运行上面的代码，我们将看到控制台输出:

```java
[DEMO]magic
[DEMO]project
[DEMO]database:
[DEMO]oracle
[DEMO]dependencies:
[DEMO]spring:foo:bar
[DEMO]--------OUTPUT--END--------- 
```

### 2.2.带自定义分隔符

到目前为止，我们已经用默认分隔符查看了`hasNext()`。`Scanner`类**提供了一个 [`useDelimiter(String pattern)`](https://web.archive.org/web/20221206031835/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/Scanner.html#useDelimiter(java.lang.String)) 方法**，允许我们改变分隔符。一旦改变了定界符，`hasNext()`方法将使用新的定界符而不是默认的定界符进行检查。

让我们看看另一个例子，看看`hasNext() `和`next() `如何使用自定义分隔符。我们将重用上一个例子中的输入。

在扫描器解析匹配字符串“`dependencies:`”的令牌后，我们将把分隔符改为冒号(`: )` ，以便我们可以解析和提取依赖项的每个值:

```java
while (scanner.hasNext()) {
    String token = scanner.next();
    if ("dependencies:".equals(token)) {
        scanner.useDelimiter(":");
    }
    log.info(token);
}
log.info("--------OUTPUT--END---------");
```

让我们看看结果输出:

```java
[DEMO]magic
[DEMO]project
[DEMO]database:
[DEMO]oracle
[DEMO]dependencies:
[DEMO]
spring
[DEMO]foo
[DEMO]bar

[DEMO]--------OUTPUT--END---------
```

太好了！我们已经成功提取了“`dependencies`”中的值，但是有一些**意外的换行问题**。我们将在下一节中看到如何避免这些问题。

### 2.3.以`regex`为分隔符

让我们回顾一下上一部分的输出。首先，我们注意到在“`spring`”之前有一个换行符(`\n`)。在获取了`“dependencies:”` 令牌后，我们将分隔符更改为“`:`”。“`dependencies:`后的换行符现在成为下一个令牌的一部分。于是，`hasNext() `返回`true`，换行符打印出来。

同理，“`hibernate`”后的换行和最后一个空行成为最后一个 token 的一部分，所以两个空行和“【T1””一起打印出来。

如果我们能让冒号和空格都作为分隔符，那么“依赖”值将被正确解析，我们的问题将得到解决。为了实现这一点，让我们改变`useDelimiter(“:”)`调用:

```java
scanner.useDelimiter(":|\\s+"); 
```

这里的“`:|\\s+`”是匹配单个“:”或一个或多个空白字符的正则表达式。通过此修复，输出变成:

```java
[DEMO]magic
[DEMO]project
[DEMO]database:
[DEMO]oracle
[DEMO]dependencies:
[DEMO]spring
[DEMO]foo
[DEMO]bar
[DEMO]--------OUTPUT--END---------
```

## 3.`hasNextLine()`

**[`hasNextLine()`](https://web.archive.org/web/20221206031835/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/Scanner.html#hasNextLine())方法检查在`Scanner`对象的输入中是否还有另一行，不管这一行是否为空。**

让我们再来看看同样的输入。这一次，我们将使用`hasNextLine()`和`nextLine()`方法在输入的每一行前面添加行号:

```java
int i = 0;
while (scanner.hasNextLine()) {
    log.info(String.format("%d|%s", ++i, scanner.nextLine()));
}
log.info("--------OUTPUT--END---------");
```

现在，让我们看看我们的输出:

```java
[DEMO]1|magic	project
[DEMO]2|     database: oracle
[DEMO]3|dependencies:
[DEMO]4|spring:foo:bar
[DEMO]5|
[DEMO]--------OUTPUT--END---------
```

正如我们所料，行号被打印出来，最后一个空行也在那里。

## 4.结论

在本文中，我们了解到`Scanner`的`hasNextLine()` 方法检查输入中是否有另一行，不管这一行是否为空，而`hasNext()`使用分隔符来检查另一个标记。

与往常一样，GitHub 上的[提供了示例的完整源代码。](https://web.archive.org/web/20221206031835/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-io-apis)