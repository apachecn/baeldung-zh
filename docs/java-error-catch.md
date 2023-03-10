# Java 中的 Catch 块会捕捉到错误吗？

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-error-catch>

## 1.概观

在这篇短文中，我们将展示如何正确地捕捉 Java 错误，并解释何时这样做没有意义。

关于 Java 中`Throwable` s 的详细信息，请看我们关于 Java 中[异常处理的文章。](/web/20221127014058/https://www.baeldung.com/java-exceptions)

## 2.捕捉错误

由于 Java 中的 `java.lang.Error`类不是从`java.lang.Exception`继承的，我们必须在 catch 语句中声明`Error`基类——或者我们想要捕获的特定的`Error`子类——以便捕获它。

因此，如果我们运行下面的测试用例，它将通过:

```java
@Test(expected = AssertionError.class)
public void whenError_thenIsNotCaughtByCatchException() {
    try {
        throw new AssertionError();
    } catch (Exception e) {
        Assert.fail(); // errors are not caught by catch exception
    }
}
```

但是，下面的单元测试期望 catch 语句捕获错误:

```java
@Test
public void whenError_thenIsCaughtByCatchError() {
    try {
        throw new AssertionError();
    } catch (Error e) {
        // caught! -> test pass
    }
}
```

请注意，**Java 虚拟机抛出错误来表明它无法从**中恢复的严重问题，例如内存不足和堆栈溢出等等。

因此，我们必须有一个非常非常好的理由来捕捉错误！

## 3.结论

在本文中，我们看到了在 Java 中何时以及如何捕获`Error` s。代码示例可以在 GitHub 项目中找到。