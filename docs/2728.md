# 在 JUnit 中使用失败断言

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/junit-fail>

## 1.概观

在本教程中，我们将**探索如何将 JUnit `fail`断言用于常见的测试场景。**

我们还将看到[`fail()`](https://web.archive.org/web/20221024205248/http://junit.sourceforge.net/javadoc/org/junit/Assert.html#fail())[JUnit 4 和 JUnit 5](/web/20221024205248/https://www.baeldung.com/junit) 之间的方法差异。

## 2.使用`fail`断言

`fail`断言测试失败，无条件抛出`AssertionError`。

**当编写单元测试时，我们可以使用`fail`在期望的测试条件下显式地创建一个失败。**让我们来看一些有用的案例。

### 2.1.不完全测试

当测试不完整或尚未实现时，我们可能会失败:

```
@Test
public void incompleteTest() {
    fail("Not yet implemented");
}
```

### 2.2.预期异常

当我们认为会发生异常时，我们也可以这样做:

```
@Test
public void expectedException() {
    try {
        methodThrowsException();
        fail("Expected exception was not thrown");
    } catch (Exception e) {
        assertNotNull(e);
    }
}
```

### 2.3.意外异常

当不希望抛出异常时，测试失败是另一种选择:

```
@Test
public void unexpectedException() {
    try {
        safeMethod();
        // more testing code
    } catch (Exception e) {
        fail("Unexpected exception was thrown");
    }
}
```

### 2.4.测试条件

当一个结果不满足某些期望的条件时，我们可以调用`fail() `:

```
@Test
public void testingCondition() {
    int result = randomInteger();
    if(result > Integer.MAX_VALUE) {
        fail("Result cannot exceed integer max value");
    }
    // more testing code
}
```

### 2.5.之前返回

最后，当代码没有按预期返回/中断时，我们可能会测试失败:

```
@Test
public void returnBefore() {
    int value = randomInteger();
    for (int i = 0; i < 5; i++) {
        // returns when (value + i) is an even number
        if ((i + value) % 2 == 0) {
            return;
        }
    }
    fail("Should have returned before");
}
```

## 3.JUnit 5 对 JUnit 4

JUnit 4 中的所有断言都是`org.junit.Assert`类的一部分。对于 JUnit 5，这些被移到了`org.junit.jupiter.api.Assertions.`

当我们在 JUnit 5 中调用`fail`并得到一个异常时，我们收到的是`AssertionFailedError`而不是在` JUnit 4.`中找到的`AssertionError`

除了`fail()`和`fail(String message)`，JUnit 5 还包括一些有用的重载:

*   `fail(Throwable cause)`
*   `fail(String message, Throwable cause)`
*   `fail(Supplier<String> messageSupplier)`

**另外，JUnit 5 中所有形式的`fail`都声明为`public static <V> V fail()`。泛型返回类型`V,`允许这些方法在 lambda 表达式中作为单语句使用:**

```
Stream.of().map(entry -> fail("should not be called"));
```

## 4.结论

在本文中，我们介绍了 JUnit 中`fail`断言的一些实际用例。请参见 [JUnit 断言](/web/20221024205248/https://www.baeldung.com/junit-assertions "JUnit Assertions")，了解 [JUnit 4 和 JUnit 5](/web/20221024205248/https://www.baeldung.com/junit-5-migration) 中所有可用的断言。

我们还强调了 JUnit 4 和 JUnit 5 之间的主要区别，以及对`fail`方法的一些有用的增强。

和往常一样，GitHub 上的[提供了这篇文章的完整源代码。](https://web.archive.org/web/20221024205248/https://github.com/eugenp/tutorials/tree/master/testing-modules/junit-4)