# Scanner nextLine()方法

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-scanner-nextline>

## 1.概观

在这个快速教程中，我们将简要地看一下 [`java.util.Scanner`](/web/20220929211315/https://www.baeldung.com/java-scanner) 类的`nextLine()`方法，当然重点是学习如何在实践中使用它。

## 2.`Scanner.nextLine()`

`java.util.Scanner`类的`nextLine()`方法从当前位置开始扫描，直到找到一个行分隔符。该方法返回从当前位置到行尾的`String`。

因此，在操作之后，扫描器的位置被设置为分隔符之后的下一行的开始。

该方法将在输入数据中搜索换行符。如果没有行分隔符，它可以扫描所有输入数据，搜索要跳过的行。

`nextLine()`方法的签名是:

```
public String nextLine()
```

该方法不带参数。它返回当前行，不包括末尾的任何行分隔符。

让我们看看它的用法:

```
try (Scanner scanner = new Scanner("Scanner\nTest\n")) {
    assertEquals("Scanner", scanner.nextLine());
    assertEquals("Test", scanner.nextLine());
}
```

正如我们所看到的，该方法从当前扫描器位置返回输入，直到找到行分隔符:

```
try (Scanner scanner = new Scanner("Scanner\n")) {
    scanner.useDelimiter("");
    scanner.next();
    assertEquals("canner", scanner.nextLine());
}
```

在上面的例子中，对`next()`的调用返回`‘S'`,并将扫描仪位置推进到指向`‘c'`。

因此，当我们调用`nextLine()`方法时，它从当前扫描仪位置返回输入，直到找到一个行分隔符。

`nextLine()`方法抛出两种类型的检查异常。

首先，当没有找到行分隔符时，抛出`NoSuchElementException`:

```
@Test(expected = NoSuchElementException.class)
public void whenReadingLines_thenThrowNoSuchElementException() {
    try (Scanner scanner = new Scanner("")) {
        scanner.nextLine();
    }
}
```

其次，如果扫描仪关闭，它抛出`IllegalStateException` :

```
@Test(expected = IllegalStateException.class)
public void whenReadingLines_thenThrowIllegalStateException() {
    Scanner scanner = new Scanner("");
    scanner.close();
    scanner.nextLine();
}
```

## 3.结论

在这篇中肯的文章中，我们看了 Java 的`Scanner`类的`nextLine()`方法。

此外，我们还查看了它在一个简单的 Java 程序中的用法。最后，我们看了由方法抛出的异常和说明它的示例代码。

和往常一样，GitHub 上的[提供了工作示例的完整源代码。](https://web.archive.org/web/20220929211315/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-io-apis)