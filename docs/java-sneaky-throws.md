# Java 中的“鬼祟抛出”

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-sneaky-throws>

## 1。概述

在 Java 中，sn `eaky throw`概念允许我们抛出任何检查过的异常，而无需在方法签名中明确定义它。这允许省略`throws`声明，有效地模仿运行时异常的特征。

在本文中，我们将通过查看一些代码示例来了解这在实践中是如何实现的。

## 2。关于鬼鬼祟祟的投掷

检查异常是 Java 的一部分，而不是 JVM 的一部分。在字节码中，我们可以从任何地方抛出任何异常，没有任何限制。

Java 8 引入了一个新的类型推断规则，规定只要允许，就将一个`throws T`推断为`RuntimeException`。这提供了在没有 helper 方法的情况下实现卑鄙抛出的能力。

`sneaky throws` 的一个问题是，您可能最终想要捕获异常，但是 Java 编译器不允许您使用特定异常类型的异常处理程序来捕获偷偷抛出的检查过的异常。

## 3。鬼祟的投掷动作

正如我们已经提到的，编译器和 Jave 运行时可以看到不同的东西:

```java
public static <E extends Throwable> void sneakyThrow(Throwable e) throws E {
    throw (E) e;
}

private static void throwSneakyIOException() {
    sneakyThrow(new IOException("sneaky"));
}
```

**编译器将签名的`throws T`推断为 `RuntimeException`类型**，因此它允许未检查的异常传播。Java 运行时在抛出中看不到任何类型，因为所有抛出都是一样的一个简单的`throw e`T3

这个快速测试演示了这个场景:

```java
@Test
public void throwSneakyIOException_IOExceptionShouldBeThrown() {
    assertThatThrownBy(() -> throwSneakyIOException())
      .isInstanceOf(IOException.class)
      .hasMessage("sneaky")
      .hasStackTraceContaining("SneakyThrowsExamples.throwSneakyIOException");
}
```

此外，可以使用字节码操作或`Thread.stop(Throwable)`抛出一个检查过的异常，但是这很麻烦，不推荐使用。

## 4。使用 Lombok 注释

来自 [Lombok](https://web.archive.org/web/20220920191012/https://projectlombok.org/) 的 `@SneakyThrows` 注释允许你在不使用`throws` 声明的情况下抛出检查过的异常。当你需要在像`Runnable.`这样非常严格的接口中从一个方法中引发一个异常时，这就很方便了

假设我们从一个`Runnable`中抛出一个异常；它只会被传递给`Thread'` `s`未处理的异常处理程序。

这段代码将抛出`Exception` 实例，因此您不需要将它包装在`RuntimeException:`中

```java
@SneakyThrows
public static void throwSneakyIOExceptionUsingLombok() {
    throw new IOException("lombok sneaky");
}
```

这段代码的一个缺点是，您无法捕获未声明的已检查异常。例如，**如果我们试图捕捉上面方法偷偷抛出的`IOException`，我们将得到一个编译错误。**

现在，让我们调用`throwSneakyIOExceptionUsingLombok `并期待 Lombok 抛出 IOException:

```java
@Test
public void throwSneakyIOExceptionUsingLombok_IOExceptionShouldBeThrown() {
    assertThatThrownBy(() -> throwSneakyIOExceptionUsingLombok())
      .isInstanceOf(IOException.class)
      .hasMessage("lombok sneaky")
      .hasStackTraceContaining("SneakyThrowsExamples.throwSneakyIOExceptionUsingLombok");
}
```

## 5。结论

正如我们在本文中看到的，我们可以欺骗 Java 编译器将检查异常视为未检查异常。

和往常一样，代码可以在 GitHub 的[上获得。](https://web.archive.org/web/20220920191012/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-exceptions-4)