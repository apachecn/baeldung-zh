# NoException 简介

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/no-exception>

## 1。概述

有时`try/catch`块会导致冗长甚至笨拙的代码结构。

在本文中，我们将重点关注 **[`NoException`](https://web.archive.org/web/20220820045038/https://noexception.machinezoo.com/) ，它提供了简洁方便的异常处理程序。**

## 2。Maven 依赖关系

让我们将`[NoException](https://web.archive.org/web/20220820045038/https://search.maven.org/classic/#search%7Cga%7C1%7Ca%3A%22noexception%22)`添加到`pom.xml`中:

```
<dependency>
    <groupId>com.machinezoo.noexception</groupId>
    <artifactId>noexception</artifactId>
    <version>1.1.0</version>
</dependency>
```

## 3。标准异常处理

先说一个常见的习语:

```
private static Logger logger = LoggerFactory.getLogger(NoExceptionUnitTest.class);

@Test
public void whenStdExceptionHandling_thenCatchAndLog() {
    try {
        logger.info("Result is " + Integer.parseInt("foobar"));
    } catch (Throwable exception) {
        logger.error("Caught exception:", exception);
    }
}
```

我们从分配一个`Logger`开始，然后进入一个`try`块。如果抛出了一个`Exception`,我们会记录下来:

```
09:29:28.140 [main] ERROR c.b.n.NoExceptionUnitTest 
  - Caught exception
j.l.NumberFormatException: For input string: "foobar"
at j.l.NumberFormatException.forInputString(NumberFormatException.java:65)
at j.l.Integer.parseInt(Integer.java:580)
...
```

## 4。用`NoException` 处理异常

### 4.1。默认日志处理程序

让我们用`NoException`的标准异常处理程序来代替它:

```
@Test
public void whenDefaultNoException_thenCatchAndLog() {
    Exceptions 
      .log()
      .run(() -> System.out.println("Result is " + Integer.parseInt("foobar")));
}
```

这段代码给出了与上面几乎相同的输出:

```
09:36:04.461 [main] ERROR c.m.n.Exceptions 
  - Caught exception
j.l.NumberFormatException: For input string: "foobar"
at j.l.NumberFormatException.forInputString(NumberFormatException.java:65)
at j.l.Integer.parseInt(Integer.java:580)
...
```

**在其最基本的形式中，`NoException`为我们提供了一种用单行代码替换`try/catch` /异常的方法。**它执行我们传递给`run()`的 lambda，如果一个`Exception`被抛出，它将被记录。

### 4.2。`Logger`添加自定义

如果我们仔细观察输出，我们会看到异常被记录为日志类，而不是我们的。

我们可以通过提供我们的记录器来解决这个问题:

```
@Test
public void whenDefaultNoException_thenCatchAndLogWithClassName() {
    Exceptions
      .log(logger)
      .run(() -> System.out.println("Result is " + Integer.parseInt("foobar")));
}
```

这给了我们这样的输出:

```
09:55:23.724 [main] ERROR c.b.n.NoExceptionUnitTest 
  - Caught exception
j.l.NumberFormatException: For input string: "foobar"
at j.l.NumberFormatException.forInputString(NumberFormatException.java:65)
at j.l.Integer.parseInt(Integer.java:580)
...
```

### 4.3。提供自定义日志消息

我们可能希望使用不同于默认的“捕获的异常”的消息**我们可以通过传递一个`Logger`作为第一个参数，传递一个`String`消息作为第二个`:`** 来实现

```
@Test
public void whenDefaultNoException_thenCatchAndLogWithMessage() {
    Exceptions
      .log(logger, "Something went wrong:")
      .run(() -> System.out.println("Result is " + Integer.parseInt("foobar")));
}
```

这给了我们这样的输出:

```
09:55:23.724 [main] ERROR c.b.n.NoExceptionUnitTest 
  - Something went wrong:
j.l.NumberFormatException: For input string: "foobar"
at j.l.NumberFormatException.forInputString(NumberFormatException.java:65)
at j.l.Integer.parseInt(Integer.java:580)
...
```

但是，如果我们想做的不仅仅是记录`Exceptions`，比如在`parseInt()`失败时插入一个回退值，该怎么办呢？

### 4.4。指定默认值

`Exceptions`可以返回一个包装在`Optional`中的结果。让我们移动一下，以便在目标失败时可以使用它来提供默认值:

```
@Test
public void
  givenDefaultValue_whenDefaultNoException_thenCatchAndLogPrintDefault() {
    System.out.println("Result is " + Exceptions
      .log(logger, "Something went wrong:")
      .get(() -> Integer.parseInt("foobar"))
      .orElse(-1));
}
```

我们仍然看到我们的`Exception`:

```
12:02:26.388 [main] ERROR c.b.n.NoExceptionUnitTest
  - Caught exception java.lang.NumberFormatException: For input string: "foobar"
at j.l.NumberFormatException.forInputString(NumberFormatException.java:65)
at j.l.Integer.parseInt(Integer.java:580)
...
```

但是我们也看到我们的消息被打印到控制台上:

```
Result is -1
```

## 5。创建自定义日志处理程序

到目前为止，我们有一个很好的方法来避免重复，并使代码在简单的`try/catch/log`场景中更具可读性。如果我们想要重用一个具有不同行为的处理程序呢？

让我们扩展`NoException`的`ExceptionHandler`类，并根据异常类型执行以下两种操作之一:

```
public class CustomExceptionHandler extends ExceptionHandler {

Logger logger = LoggerFactory.getLogger(CustomExceptionHandler.class);

    @Override
    public boolean handle(Throwable throwable) {
        if (throwable.getClass().isAssignableFrom(RuntimeException.class)
          || throwable.getClass().isAssignableFrom(Error.class)) {
            return false;
        } else {
            logger.error("Caught Exception", throwable);
            return true;
        }
    }
}
```

**当我们看到`Error`或`RuntimeException`时，通过返回`false`，我们告诉`ExceptionHandler` 再投一次。通过为其他所有事情返回`true`,我们表明异常已经被处理。**

首先，我们将运行这个标准异常:

```
@Test
public void givenCustomHandler_whenError_thenRethrowError() {
    CustomExceptionHandler customExceptionHandler = new CustomExceptionHandler();
    customExceptionHandler.run(() -> "foo".charAt(5));
}
```

我们将函数传递给从`ExceptionHandler:`继承的自定义处理程序中的`run()`方法

```
18:35:26.374 [main] ERROR c.b.n.CustomExceptionHandler 
  - Caught Exception 
j.l.StringIndexOutOfBoundsException: String index out of range: 5
at j.l.String.charAt(String.java:658)
at c.b.n.CustomExceptionHandling.throwSomething(CustomExceptionHandling.java:20)
at c.b.n.CustomExceptionHandling.lambda$main$0(CustomExceptionHandling.java:10)
at c.m.n.ExceptionHandler.run(ExceptionHandler.java:1474)
at c.b.n.CustomExceptionHandling.main(CustomExceptionHandling.java:10) 
```

此异常被记录。让我们用一个`Error`来试试:

```
@Test(expected = Error.class)
public void givenCustomHandler_whenException_thenCatchAndLog() {
    CustomExceptionHandler customExceptionHandler = new CustomExceptionHandler();
    customExceptionHandler.run(() -> throwError());
}

private static void throwError() {
    throw new Error("This is very bad.");
}
```

我们看到`Error`被重新抛出到`main()`中，而不是被记录:

```
Exception in thread "main" java.lang.Error: This is very bad.
at c.b.n.CustomExceptionHandling.throwSomething(CustomExceptionHandling.java:15)
at c.b.n.CustomExceptionHandling.lambda$main$0(CustomExceptionHandling.java:8)
at c.m.n.ExceptionHandler.run(ExceptionHandler.java:1474)
t c.b.n.CustomExceptionHandling.main(CustomExceptionHandling.java:8)
```

因此，我们有一个可重用的类，可以在整个项目中使用，以实现一致的异常处理。

## 6。结论

使用`NoException`,我们可以用一行代码逐例简化异常处理。

代码可以在[这个 GitHub 项目](https://web.archive.org/web/20220820045038/https://github.com/eugenp/tutorials/tree/master/libraries-4)中找到。