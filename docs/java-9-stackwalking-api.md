# Java 9 堆栈审核 API 简介

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-9-stackwalking-api>

## 1。简介

在这篇简短的文章中，我们将看看 Java 9 的[堆栈审核 API](https://web.archive.org/web/20220820045610/https://openjdk.java.net/jeps/259) 。

**新功能提供了对`StackFrame` s** 的`Stream`的访问，使我们能够轻松地直接浏览堆栈，并充分利用 Java 8 中强大的 [`Stream` API。](/web/20220820045610/https://www.baeldung.com/java-8-streams)

## 2。`StackWalker` 【T2 的优势】

在 Java 8 中，`Throwable::getStackTrace`和`Thread::getStackTrace`返回一个由`StackTraceElement`组成的数组。如果没有大量的手动代码，就没有办法丢弃不需要的帧，只保留我们感兴趣的帧。

除此之外，`Thread::getStackTrace`可能会返回部分堆栈跟踪。这是因为规范允许 VM 实现为了性能而省略一些堆栈帧。

在 Java 9 中，**使用`StackWalker`的`walk()`方法，我们可以遍历我们感兴趣的几帧**或者完整的堆栈跟踪。

当然，新功能是线程安全的；这允许多个线程共享一个`StackWalker`实例来访问它们各自的堆栈。

正如在 [JEP-259](https://web.archive.org/web/20220820045610/https://openjdk.java.net/jeps/259) 中所描述的，JVM 将得到增强，以允许在需要时对额外的堆栈帧进行高效的惰性访问。

## 3。`StackWalker`在行动中

让我们首先创建一个包含一系列方法调用的类:

```java
public class StackWalkerDemo {

    public void methodOne() {
        this.methodTwo();
    }

    public void methodTwo() {
        this.methodThree();
    }

    public void methodThree() {
        // stack walking code
    }
}
```

### 3.1。捕获整个堆栈跟踪

让我们继续，添加一些堆栈审核代码:

```java
public void methodThree() {
    List<StackFrame> stackTrace = StackWalker.getInstance()
      .walk(this::walkExample);
} 
```

`StackWalker::walk`方法接受一个函数引用，为当前线程创建一个`StackFrame`的`Stream`，将函数应用于`Stream`，并关闭`Stream`。

现在让我们定义一下`StackWalkerDemo::walkExample`方法:

```java
public List<StackFrame> walkExample(Stream<StackFrame> stackFrameStream) {
    return stackFrameStream.collect(Collectors.toList());
}
```

这个方法只是收集`StackFrame`并将其作为`List<StackFrame>`返回。为了测试这个例子，请运行一个 JUnit 测试:

```java
@Test
public void giveStalkWalker_whenWalkingTheStack_thenShowStackFrames() {
    new StackWalkerDemo().methodOne();
}
```

将它作为 JUnit 测试运行的唯一原因是我们的堆栈中有更多的帧:

```java
class com.baeldung.java9.stackwalker.StackWalkerDemo#methodThree, Line 20
class com.baeldung.java9.stackwalker.StackWalkerDemo#methodTwo, Line 15
class com.baeldung.java9.stackwalker.StackWalkerDemo#methodOne, Line 11
class com.baeldung.java9.stackwalker
  .StackWalkerDemoTest#giveStalkWalker_whenWalkingTheStack_thenShowStackFrames, Line 9
class org.junit.runners.model.FrameworkMethod$1#runReflectiveCall, Line 50
class org.junit.internal.runners.model.ReflectiveCallable#run, Line 12
  ...more org.junit frames...
class org.junit.runners.ParentRunner#run, Line 363
class org.eclipse.jdt.internal.junit4.runner.JUnit4TestReference#run, Line 86
  ...more org.eclipse frames...
class org.eclipse.jdt.internal.junit.runner.RemoteTestRunner#main, Line 192
```

在整个堆栈跟踪中，我们只对前四帧感兴趣。来自 org.junit 的剩余的**帧和`org.eclipse`只不过是噪声帧**。

### 3.2。过滤`StackFrame`年代

让我们增强堆栈审核代码并消除干扰:

```java
public List<StackFrame> walkExample2(Stream<StackFrame> stackFrameStream) {
    return stackFrameStream
      .filter(f -> f.getClassName().contains("com.baeldung"))
      .collect(Collectors.toList());
}
```

利用`Stream` API 的强大功能，我们只保留我们感兴趣的帧。这将清除噪声，在堆栈日志中留下前四行:

```java
class com.baeldung.java9.stackwalker.StackWalkerDemo#methodThree, Line 27
class com.baeldung.java9.stackwalker.StackWalkerDemo#methodTwo, Line 15
class com.baeldung.java9.stackwalker.StackWalkerDemo#methodOne, Line 11
class com.baeldung.java9.stackwalker
  .StackWalkerDemoTest#giveStalkWalker_whenWalkingTheStack_thenShowStackFrames, Line 9
```

现在让我们来确定发起呼叫的 JUnit 测试:

```java
public String walkExample3(Stream<StackFrame> stackFrameStream) {
    return stackFrameStream
      .filter(frame -> frame.getClassName()
        .contains("com.baeldung") && frame.getClassName().endsWith("Test"))
      .findFirst()
      .map(f -> f.getClassName() + "#" + f.getMethodName() 
        + ", Line " + f.getLineNumber())
      .orElse("Unknown caller");
}
```

请注意，在这里，我们只对映射到一个`String`的单个`StackFrame,`感兴趣。输出将只是包含`StackWalkerDemoTest`类的那一行。

### 3.3。捕捉反射帧

为了捕捉默认隐藏的反射帧，`StackWalker`需要配置一个额外的选项`SHOW_REFLECT_FRAMES`:

```java
List<StackFrame> stackTrace = StackWalker
  .getInstance(StackWalker.Option.SHOW_REFLECT_FRAMES)
  .walk(this::walkExample);
```

使用此选项，包括`Method.invoke()`和`Constructor.newInstance()`在内的所有反射帧都将被捕获:

```java
com.baeldung.java9.stackwalker.StackWalkerDemo#methodThree, Line 40
com.baeldung.java9.stackwalker.StackWalkerDemo#methodTwo, Line 16
com.baeldung.java9.stackwalker.StackWalkerDemo#methodOne, Line 12
com.baeldung.java9.stackwalker
  .StackWalkerDemoTest#giveStalkWalker_whenWalkingTheStack_thenShowStackFrames, Line 9
jdk.internal.reflect.NativeMethodAccessorImpl#invoke0, Line -2
jdk.internal.reflect.NativeMethodAccessorImpl#invoke, Line 62
jdk.internal.reflect.DelegatingMethodAccessorImpl#invoke, Line 43
java.lang.reflect.Method#invoke, Line 547
org.junit.runners.model.FrameworkMethod$1#runReflectiveCall, Line 50
  ...eclipse and junit frames...
org.eclipse.jdt.internal.junit.runner.RemoteTestRunner#main, Line 192
```

正如我们所见，`jdk.internal`帧是由`SHOW_REFLECT_FRAMES`选项捕获的新帧。

### 3.4。捕捉隐藏帧

除了反射帧之外，JVM 实现可能会选择隐藏特定于实现的帧。

然而，这些帧并没有对`StackWalker`隐藏:

```java
Runnable r = () -> {
    List<StackFrame> stackTrace2 = StackWalker
      .getInstance(StackWalker.Option.SHOW_HIDDEN_FRAMES)
      .walk(this::walkExample);
    printStackTrace(stackTrace2);
};
r.run();
```

注意，在这个例子中，我们将一个 lambda 引用赋给了一个`Runnable`。唯一的原因是 JVM 会为 lambda 表达式创建一些隐藏框架。

这在堆栈跟踪中清晰可见:

```java
com.baeldung.java9.stackwalker.StackWalkerDemo#lambda$0, Line 47
com.baeldung.java9.stackwalker.StackWalkerDemo$Lambda$39/924477420#run, Line -1
com.baeldung.java9.stackwalker.StackWalkerDemo#methodThree, Line 50
com.baeldung.java9.stackwalker.StackWalkerDemo#methodTwo, Line 16
com.baeldung.java9.stackwalker.StackWalkerDemo#methodOne, Line 12
com.baeldung.java9.stackwalker
  .StackWalkerDemoTest#giveStalkWalker_whenWalkingTheStack_thenShowStackFrames, Line 9
jdk.internal.reflect.NativeMethodAccessorImpl#invoke0, Line -2
jdk.internal.reflect.NativeMethodAccessorImpl#invoke, Line 62
jdk.internal.reflect.DelegatingMethodAccessorImpl#invoke, Line 43
java.lang.reflect.Method#invoke, Line 547
org.junit.runners.model.FrameworkMethod$1#runReflectiveCall, Line 50
  ...junit and eclipse frames...
org.eclipse.jdt.internal.junit.runner.RemoteTestRunner#main, Line 192
```

最上面的两个框架是 lambda 代理框架，由 JVM 内部创建。值得注意的是，我们在前面的例子中捕获的反射帧仍然保留在`SHOW_HIDDEN_FRAMES`选项中。这是因为 **`SHOW_HIDDEN_FRAMES`是`SHOW_REFLECT_FRAMES`** 的超集。

### 3.5。识别调用类

选项`RETAIN_CLASS_REFERENCE`在`StackWalker`走过的所有`StackFrame`中零售`Class`的对象。这允许我们调用方法`StackWalker::getCallerClass`和`StackFrame::getDeclaringClass`。

让我们使用`StackWalker::getCallerClass`方法来识别调用类:

```java
public void findCaller() {
    Class<?> caller = StackWalker
      .getInstance(StackWalker.Option.RETAIN_CLASS_REFERENCE)
      .getCallerClass();
    System.out.println(caller.getCanonicalName());
}
```

这一次，我们将从一个单独的 JUnit 测试中直接调用这个方法:

```java
@Test
public void giveStalkWalker_whenInvokingFindCaller_thenFindCallingClass() {
    new StackWalkerDemo().findCaller();
}
```

`caller.getCanonicalName(),`的输出将是:

```java
com.baeldung.java9.stackwalker.StackWalkerDemoTest
```

请注意，不应该从堆栈底部的方法调用`StackWalker::getCallerClass`。因为这会导致`IllegalCallerException`被抛出。

## 4。结论

通过这篇文章，我们看到了使用结合了`StackWalker`和`Stream` API 的`StackFrame`的能力来处理`StackFrame`是多么容易。

当然，我们可以探索各种其他功能——例如跳过、丢弃和限制`StackFrame`s。[官方文档](https://web.archive.org/web/20220820045610/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/StackWalker.html)包含一些其他用例的可靠示例。

和往常一样，您可以在 GitHub 上获得本文[的完整源代码。](https://web.archive.org/web/20220820045610/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-9-new-features)