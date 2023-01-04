# 用换行符分割 Java 字符串

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-string-split-by-newline>

## 1.概观

在本教程中，我们将看看用换行符分割 Java 字符串的不同方法。由于换行符在不同的操作系统中是不同的，我们将研究一下适用于 Unix、Linux、Mac OS 9 以及更早版本的 Mac OS 和 Windows OS 的方法。

## 2.由新行拆分`String`

### 2.1.使用`System#lineSeparator`方法通过换行符拆分`String`

考虑到换行符在不同的操作系统中是不同的，当我们希望我们的代码独立于平台时，我们可以使用系统定义的常量或方法。

[`System#lineSeparator`](https://web.archive.org/web/20220924071314/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/System.html#lineSeparator()) 方法返回底层操作系统的行分隔符字符串。它返回系统属性`line.separator`的值。

因此，我们可以使用由`System#lineSeparator`方法和`String#split`方法返回的行分隔符字符串，通过换行符来拆分 Java `String`:

```
String[] lines = "Line1\r\nLine2\r\nLine3".split(System.lineSeparator());
```

生成的行将是:

```
["Line1", "Line2", "Line3"]
```

### 2.2.使用正则表达式通过换行符拆分`String`

接下来，让我们从不同操作系统中用来分隔行的不同字符开始。

在 Unix、Linux 和 macOS 中，“`\n`”字符分隔行。另一方面，在 Windows 环境中，“`\r\n`”字符分隔行。最后，在 Mac OS 9 和更早的版本中，“`\r`”字符分隔行。

因此，在使用正则表达式通过换行符拆分字符串时，我们需要注意所有可能的换行符。

最后，让我们看看正则表达式模式，它将涵盖所有不同操作系统的换行符。也就是说，我们需要寻找“\n”、“\r\n”和“\r”模式。这可以通过在 Java 中使用[正则表达式轻松完成。](/web/20220924071314/https://www.baeldung.com/regular-expressions-java)

涵盖所有不同换行符的正则表达式模式将是:

```
"\\r?\\n|\\r"
```

分解一下，我们看到:

*   `\\n` = Unix、Linux 和 macOS 模式
*   `\\r\\n` = Windows 环境模式
*   `\\r` = MacOS 9 及更早的模式

接下来，我们用`String` # `split`的方法来拆分 Java `String`。让我们看几个例子:

```
String[] lines = "Line1\nLine2\nLine3".split("\\r?\\n|\\r");
String[] lines = "Line1\rLine2\rLine3".split("\\r?\\n|\\r");
String[] lines = "Line1\r\nLine2\r\nLine3".split("\\r?\\n|\\r");
```

所有示例的结果行将是:

```
["Line1", "Line2", "Line3"]
```

### 2.3.在 Java 8 中用换行符拆分

Java 8 提供了一个`“\R”`模式，可以匹配任何 Unicode 换行符序列，并覆盖不同操作系统的所有换行符。因此，我们可以在 Java 8 或更高版本中使用`“\R”`模式来代替`“\\r?\\n|\\r”`。

让我们看几个例子:

```
String[] lines = "Line1\nLine2\nLine3".split("\\R");
String[] lines = "Line1\rLine2\rLine3".split("\\R");
String[] lines = "Line1\r\nLine2\r\nLine3".split("\\R");
```

同样，所有示例的结果输出行将是:

```
["Line1", "Line2", "Line3"]
```

### 2.4.使用`Pattern`类通过换行符拆分`String`

在 Java 8 中，`Pattern`类附带了一个方便的 [`splitAsStream`](https://web.archive.org/web/20220924071314/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/regex/Pattern.html#splitAsStream(java.lang.CharSequence)) 方法。

在我们的例子中，我们可以利用`“\R”`模式，但是当然，这个方法也可以用于通过任何更复杂的[正则表达式](/web/20220924071314/https://www.baeldung.com/regular-expressions-java)来拆分`String`。

让我们来看看它的实际应用:

```
Pattern pattern = Pattern.compile("\\R");
Stream<String> lines = pattern.splitAsStream("Line1\nLine2\nLine3");
Stream<String> lines = pattern.splitAsStream("Line1\rLine2\rLine3");
Stream<String> lines = pattern.splitAsStream("Line1\r\nLine2\r\nLine3");
```

正如我们所看到的，这一次，我们得到了一个由`String`组成的`Stream`，而不是一个数组，我们可以很容易地进一步处理它。

### 2.5.在 Java 11 中用换行符拆分

Java 11 让换行拆分变得非常容易:

```
Stream<String> lines = "Line1\nLine2\rLine3\r\nLine4".lines();
```

因为`lines()`在遮光罩下使用了一个`“\R”`图案，所以它可以与各种行分隔符一起工作。

正如我们所看到的，很难找到一种更简单的方法来通过换行符分割一个`String`!

## 3.结论

在这篇简短的文章中，我们研究了在不同的操作系统中可能会遇到的不同的换行符。此外，我们看到了如何使用我们自己的正则表达式模式通过换行符分割 Java 字符串，以及使用从 Java 8 开始可用的`“\R”`模式。

和往常一样，所有这些代码示例都可以在 GitHub 上的[处获得。](https://web.archive.org/web/20220924071314/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-string-operations-3)