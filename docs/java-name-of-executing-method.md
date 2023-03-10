# 如何获得正在执行的方法的名称？

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-name-of-executing-method>

## 1。概述

有时我们需要知道当前正在执行的 Java 方法的名称。

这篇简短的文章介绍了几种在当前执行堆栈中获取方法名的简单方法。

## 2.Java 9:堆栈审核 API

Java 9 引入了[堆栈审核 API](/web/20221126221330/https://www.baeldung.com/java-9-stackwalking-api) ，以一种懒惰且高效的方式遍历 JVM 堆栈帧。为了找到这个 API 当前执行的方法，我们可以编写一个简单的测试:

```java
public void givenJava9_whenWalkingTheStack_thenFindMethod() {
    StackWalker walker = StackWalker.getInstance();
    Optional<String> methodName = walker.walk(frames -> frames
      .findFirst()
      .map(StackWalker.StackFrame::getMethodName));

    assertTrue(methodName.isPresent());
    assertEquals("givenJava9_whenWalkingTheStack_thenFindMethod", methodName.get());
}
```

首先，我们使用`[getInstance()](https://web.archive.org/web/20221126221330/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/StackWalker.html#getInstance()) `工厂方法获得一个`[StackWalker](https://web.archive.org/web/20221126221330/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/StackWalker.html) `实例。**然后我们使用`[walk()](https://web.archive.org/web/20221126221330/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/StackWalker.html#walk(java.util.function.Function)) `方法从上到下遍历堆栈帧:**

*   `walk() `方法可以将堆栈帧流— `Stream<[StackFrame](https://web.archive.org/web/20221126221330/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/StackWalker.StackFrame.html)> — `转换成任何东西
*   给定流中的第一个元素是堆栈的顶部帧
*   堆栈顶部的框架总是表示当前正在执行的方法

因此，如果我们从流中获取第一个元素，我们将知道当前执行的方法细节。更具体地说，**我们可以使用`[StackFrame.getMethodName()](https://web.archive.org/web/20221126221330/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/StackWalker.StackFrame.html#getMethodName()) `来查找方法名。**

### 2.1.优势

与其他方法(稍后将详细介绍)相比，堆栈审核 API 有几个优点:

*   无需创建虚拟匿名内部类实例— `new Object().getClass() {}`
*   无需创建虚拟异常— `new Throwable()`
*   不需要急切地捕获整个堆栈跟踪，这可能会很昂贵

相反，`StackWalker `只是以一种懒散的方式一个接一个地遍历堆栈。在这种情况下，它只获取顶部的帧，这与 stacktrace 方法相反，stack trace 方法急切地捕获所有的帧。

底线是，如果你在 Java 9+上，使用堆栈审核 API。

## 3。使用`getEnclosingMethod`

我们可以通过使用`getEnclosingMethod()` API 找到正在执行的方法的名称:

```java
public void givenObject_whenGetEnclosingMethod_thenFindMethod() {
    String methodName = new Object() {}
      .getClass()
      .getEnclosingMethod()
      .getName();

    assertEquals("givenObject_whenGetEnclosingMethod_thenFindMethod",
      methodName);
}
```

## 4。使用`Throwable`堆栈跟踪

使用一个`Throwable`堆栈跟踪给我们当前正在执行的方法的堆栈跟踪:

```java
public void givenThrowable_whenGetStacktrace_thenFindMethod() {
    StackTraceElement[] stackTrace = new Throwable().getStackTrace();

    assertEquals(
      "givenThrowable_whenGetStacktrace_thenFindMethod",
      stackTrace[0].getMethodName());
}
```

## 5。使用线程堆栈跟踪

此外，当前线程的堆栈跟踪(从 JDK 1.5 开始)通常包括正在执行的方法的名称:

```java
public void givenCurrentThread_whenGetStackTrace_thenFindMethod() {
    StackTraceElement[] stackTrace = Thread.currentThread()
      .getStackTrace();

    assertEquals(
      "givenCurrentThread_whenGetStackTrace_thenFindMethod",
      stackTrace[1].getMethodName()); 
}
```

然而，我们需要记住，这种解决方案有一个明显的缺点。一些虚拟机可能会跳过一个或多个堆栈帧。虽然这种情况并不常见，但我们应该意识到这种情况可能会发生。

## 6。结论

在本教程中，我们展示了一些如何获取当前执行的方法的名称的例子。示例基于堆栈跟踪和`getEnclosingMethod()`。

和往常一样，你可以在 GitHub 上查看本文[中提供的例子。另外，Java 9 的例子可以在我们的](https://web.archive.org/web/20221126221330/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-reflection) [Java 9 GitHub 模块](https://web.archive.org/web/20221126221330/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-9-new-features)中找到。