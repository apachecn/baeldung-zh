# 在 Java 中将堆栈跟踪转换为字符串

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-stacktrace-to-string>

## 1。简介

当在 Java 中处理异常时，我们经常记录或简单地显示堆栈跟踪。然而，有时我们不希望仅仅打印堆栈跟踪，我们可能需要将堆栈跟踪写入文件、数据库，甚至通过网络传输。

出于这些目的，将堆栈跟踪作为一个*字符串*将非常有用。不幸的是，Java 没有提供一个非常方便的方法来直接做到这一点。

## 2。用核心 Java 进行转换

让我们从核心库开始。

*异常*类的函数 *printStackTrace()* 可以带一个参数，可以是 *PrintStream* 或 *PrintWriter* 。因此，可以使用 *StringWriter* 将堆栈跟踪打印到*字符串*中:

```
StringWriter sw = new StringWriter();
PrintWriter pw = new PrintWriter(sw);
e.printStackTrace(pw); 
```

然后，调用 *sw.toString()* 将堆栈跟踪作为*字符串*返回。

## 3。与 Commons-Lang 的转换

虽然前一种方法是使用 core Java 将堆栈跟踪转换成字符串的最简单的方法，但它仍然有点麻烦。幸运的是， [Apache Commons-Lang](https://web.archive.org/web/20221208143830/https://search.maven.org/classic/#search%7Cga%7C1%7Ca%3A%22commons-lang3%22) 提供了一个函数来完成这项工作。

Apache Commons-Lang 是一个非常有用的库，它提供了 Java API 的核心类所缺少的许多特性，包括可以用来处理异常的类。

首先，我们从项目配置开始。当使用 Maven 时，我们只需向 *pom.xml* 添加以下依赖关系:

```
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-lang3</artifactId>
    <version>3.12.0</version>
</dependency> 
```

然后，在我们的例子中，最有趣的类是 *ExceptionUtils* ，它提供了操作异常的函数。使用这个类，从一个*异常*中获取堆栈跟踪作为一个*字符串*非常简单:

```
String stacktrace = ExceptionUtils.getStackTrace(e); 
```

## 4。结论

将异常的堆栈跟踪作为一个*字符串*并不困难，但是这并不直观。本文给出了两种方法，要么使用核心 Java，要么使用 Apache Commons-Lang。

请记住，Java 9 将带来一个新的 [StackWalking API](/web/20221208143830/https://www.baeldung.com/java-9-stackwalking-api) ，这将使事情变得更容易。

和往常一样，代码样本可以在 GitHub 上找到[。](https://web.archive.org/web/20221208143830/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-string-conversions)