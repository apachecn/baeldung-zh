# Java 14 中有用的 NullPointerExceptions

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-14-nullpointerexception>

## 1.概观

在本教程中，我们将继续我们关于 Java 14 的[系列，来看看有用的`NullPointerException` s，这是这个版本的 JDK 中引入的新特性。](/web/20220928153048/https://www.baeldung.com/tag/java-14/)

## 2.传统的

在实践中，我们经常看到或编写链接 Java 中的方法的代码。但是当这段代码抛出一个`NullPointerException`时，就很难知道异常是从哪里产生的。

假设我们想找出一个雇员的电子邮件地址:

```java
String emailAddress = employee.getPersonalDetails().getEmailAddress().toLowerCase();
```

如果`employee`对象、`getPersonalDetails()`或`getEmailAddress()` 是 `null,`，JVM 抛出一个`NullPointerException`:

```java
Exception in thread "main" java.lang.NullPointerException
  at com.baeldung.java14.npe.HelpfulNullPointerException.main(HelpfulNullPointerException.java:10)
```

异常的根本原因是什么？不使用调试器很难确定哪个变量是`null`。**此外，JVM 将只打印出导致异常**的方法、文件名和行号。

在下一节中，我们将通过 JEP 358 来看看 Java 14 将如何解决这个问题。

## 3.有益的

SAP 在 2006 年为他们的商业 JVM 实现了有用的。它是在 2019 年 2 月作为 OpenJDK 社区的增强版提出的，此后不久，它就成为了 JEP。**因此，该功能于 2019 年 10 月完成并在 JDK 14 版本**中推送。

**本质上， [JEP 358](https://web.archive.org/web/20220928153048/https://openjdk.java.net/jeps/358) 旨在通过描述哪个变量是`null`** 来提高 JVM 生成的`NullPointerException` s 的可读性。

JEP 358 通过描述变量`null`以及方法、文件名和行号带来了详细的`NullPointerException`消息。它通过分析程序的字节码指令来工作。因此，它能够精确地确定哪个变量或表达式是`null`。

最重要的是，**详细的异常消息在 JDK 14** 中被默认关闭。要启用它，我们需要使用命令行选项:

```java
-XX:+ShowCodeDetailsInExceptionMessages
```

### 3.1.详细的异常消息

让我们考虑在激活了`ShowCodeDetailsInExceptionMessages` 标志的情况下再次运行代码:

```java
Exception in thread "main" java.lang.NullPointerException: 
  Cannot invoke "String.toLowerCase()" because the return value of 
"com.baeldung.java14.npe.HelpfulNullPointerException$PersonalDetails.getEmailAddress()" is null
  at com.baeldung.java14.npe.HelpfulNullPointerException.main(HelpfulNullPointerException.java:10)
```

这一次，从附加信息中，我们知道员工个人详细信息的电子邮件地址缺失导致了我们的异常。从这种增强中获得的知识可以节省我们调试的时间。

JVM 由两部分组成详细的异常消息。**第一部分代表失败的操作，引用为`null`的结果，而第二部分识别`null`引用**的原因:

```java
Cannot invoke "String.toLowerCase()" because the return value of "getEmailAddress()" is null
```

为了构建异常消息，JEP 358 重新创建将`null`引用推到操作数堆栈上的源代码部分。

### 3.2.技术方面

现在我们已经很好地理解了如何使用有用的`NullPointerException`来识别`null`引用，让我们来看看它的一些技术方面。

首先，**只有当 JVM 本身抛出一个`NullPointerException`** **`—`时，才会进行详细的消息计算。如果我们在 Java 代码中显式地抛出异常**，就不会进行计算。这背后的原因是，在这些情况下，我们很可能已经在异常构造函数中传递了一个有意义的消息。

其次， **JEP 358 延迟计算消息，这意味着只有当我们打印异常消息时，而不是当异常发生时**。因此，对于通常的 JVM 流来说，不应该有任何性能影响，在 JVM 流中，我们捕获并重新抛出异常，因为我们并不总是打印异常消息。

最后，**详细的异常消息可能包括来自我们源代码**的局部变量名。因此，我们可以认为这是一个潜在的安全风险。然而，这仅发生在我们运行用激活的`-g`标志编译的代码时，它生成调试信息并将其添加到我们的类文件中。

考虑一个简单的例子，我们已经编译了这个例子来包含这个额外的调试信息:

```java
Employee employee = null;
employee.getName();
```

当我们运行这段代码时，异常消息打印出局部变量名:

```java
Cannot invoke 
  "com.baeldung.java14.npe.HelpfulNullPointerException$Employee.getName()" 
because "employee" is null
```

相反，如果没有额外的调试信息，JVM 只在详细消息中提供它所知道的变量信息:

```java
Cannot invoke 
  "com.baeldung.java14.npe.HelpfulNullPointerException$Employee.getName()" 
because "<local1>" is null
```

**JVM 打印由编译器**分配的变量索引，而不是本地变量名(`employee`)。

## 4.结论

在这个快速教程中，我们学习了 Java 14 中有用的`NullPointerException`。如上所示，改进的消息有助于我们更快地调试代码，因为异常消息中有源代码细节。

和往常一样，GitHub 上的[提供了这篇文章的完整源代码。](https://web.archive.org/web/20220928153048/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-14)