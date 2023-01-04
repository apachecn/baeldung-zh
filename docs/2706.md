# 番石榴投掷物介绍

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/guava-throwables>

## 1.概观

在本文中，我们将快速浏览一下 Google Guava 的`Throwables`类。

该类包含一组静态实用程序方法，用于处理异常处理和:

*   传播
*   处理原因链

## 2.Maven 依赖性

```
<dependency>
    <groupId>com.google.guava</groupId>
    <artifactId>guava</artifactId>
    <version>31.0.1-jre</version>
</dependency>
```

## 3.传播

假设我们与一些抛出泛型`Throwable`的代码进行交互。

在大多数情况下，如果它是`Throwable`的直接子类，我们希望将其转换为`RuntimeException`。

然而，如果它是`Error, RuntimeException`或`Exception`的实例，我们可以调用`propagateIfPossible`来传播它:

```
try {
    methodThatMightThrowThrowable();
} catch (Throwable t) {
    Throwables.propagateIfPossible(t, Exception.class);
    throw new RuntimeException(t);
}
```

## 4.因果链

Guava 还提供了检查抛出的异常及其链的实用方法。

```
Throwable getRootCause(Throwable) 
```

**`getRootCause`方法允许我们获得最内部的异常**，这在我们想要找到最初的原因时很有用。

```
List<Throwable> getCausalChain(Throwable)
```

这个`getCausalChain`方法将返回层次结构中所有 throwables 的列表。如果我们想检查它是否包含某种类型的异常，这很方便。

```
String getStackTraceAsString(Throwable)
```

`getStackTraceAsString`方法将返回异常的递归堆栈跟踪。

## 5.结论

在本教程中，我们举例说明了一些使用 Guava 的`Throwables`类来简化异常处理的例子。

和往常一样，完整的源代码可以在 GitHub 上找到[。](https://web.archive.org/web/20220627075718/https://github.com/eugenp/tutorials/tree/master/guava-modules/guava-core)