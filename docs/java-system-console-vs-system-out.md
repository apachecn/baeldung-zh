# System.console()与 System.out

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-system-console-vs-system-out>

## 1.介绍

在本教程中，我们将探索`System.console()`和`System.out`之间的区别。

## 2.`System.console()`

让我们首先创建一个程序来检索`Console`对象:

```java
void printConsoleObject() {
    Console console = System.console();
    console.writer().print(console);
}
```

从交互式终端运行这个程序将会输出类似于`[[email protected]](/web/20221128052127/https://www.baeldung.com/cdn-cgi/l/email-protection)` 的内容

然而，从其他媒介运行它将抛出`NullPointerException`，因为控制台对象将是`null`。

或者，如果我们运行如下程序:

```java
$ java ConsoleAndOut > test.txt
```

然后当我们重定向流时，程序也会抛出一个`NullPointerException` 。

`Console`类还提供了在不回显字符的情况下读取密码的方法。

让我们看看实际情况:

```java
void readPasswordFromConsole() {
    Console console = System.console();
    char[] password = console.readPassword("Enter password: ");
    console.printf(String.valueOf(password));
}
```

这将提示输入密码，当我们输入密码时，它不会回显字符。

## 3.`System.out`

现在让我们打印`System.out`的对象:

```java
System.out.println(System.out);
```

这将返回类似于`java.io.PrintStream.` 的内容

任何地方的输出都是一样的。

`System.out`用于将数据打印到输出流，没有读取数据的方法。输出流可以重定向到任何目的地，如文件，输出将保持不变。

我们可以按如下方式运行程序:

```java
$ java ConsoleAndOut > test.txt
```

这将把输出打印到`test.txt`文件。

## 4.差异

根据这些例子，我们可以找出一些不同之处:

*   从交互终端运行时，`System.console()`返回一个`java.io.Console`实例——另一方面，`System.out`将返回`java.io.PrintStream`对象，而不考虑调用媒介
*   如果我们没有重定向任何流， `System.out`和`System.console()`的行为是相似的；否则，`System.console()`返回`null`
*   当多个线程提示输入时，`Console`会很好地将这些提示排队——而在`System.out`的情况下，所有的提示会同时出现

## 5.结论

我们在这篇文章中了解了`System.console()`和`System.out`之间的区别。我们解释了当一个应用程序应该从一个交互式控制台运行时，`Console`是有用的，但是它有一些奇怪的地方需要注意和处理。

和往常一样，本文的完整代码可以在 GitHub 上的[处获得。](https://web.archive.org/web/20221128052127/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-console)