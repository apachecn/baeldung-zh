# AssertJ 异常断言

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/assertj-exception-assertion>

## 1。概述

在这个快速教程中，我们将看看 [AssertJ 的](https://web.archive.org/web/20220830145527/https://joel-costigliola.github.io/assertj/)异常专用断言。

## 2.无资产 j

为了测试是否抛出了异常，我们需要捕捉异常，然后执行断言:

```java
try {
    // ...
} catch (Exception e) {
    // assertions
} 
```

但是，如果没有抛出异常呢？在这种情况下，测试将通过；这就是为什么手工测试用例失败是必要的。

## 3。带资产 J

使用 Java 8，通过利用 AssertJ 和 lambda 表达式，我们可以很容易地对异常进行断言。

### 3.1。使用`assertThatThrownBy()`

让我们检查索引列表中的越界项是否会引发`IndexOutOfBoundsException:`

```java
assertThatThrownBy(() -> {
    List<String> list = Arrays.asList("String one", "String two");
    list.get(2);
}).isInstanceOf(IndexOutOfBoundsException.class)
  .hasMessageContaining("Index: 2, Size: 2"); 
```

注意可能抛出异常的代码片段是如何作为 lambda 表达式传递的。

当然，我们可以利用各种标准的 AssertJ 断言，比如:

```java
.hasMessage("Index: %s, Size: %s", 2, 2)
.hasMessageStartingWith("Index: 2")
.hasMessageContaining("2")
.hasMessageEndingWith("Size: 2")
.hasMessageMatching("Index: \\d+, Size: \\d+")
.hasCauseInstanceOf(IOException.class)
.hasStackTraceContaining("java.io.IOException");
```

### 3.2。使用`assertThatExceptionOfType`

这个想法类似于上面的例子，但是我们可以在开始的时候指定异常类型:

```java
assertThatExceptionOfType(IndexOutOfBoundsException.class)
  .isThrownBy(() -> {
      // ...
}).hasMessageMatching("Index: \\d+, Size: \\d+"); 
```

### 3.3。使用`assertThatIOException`和其他常用类型

AssertJ 为常见的异常类型提供了包装器，比如:

```java
assertThatIOException().isThrownBy(() -> {
    // ...
}); 
```

类似地:

*   `assertThatIllegalArgumentException()`
*   `assertThatIllegalStateException()`
*   `assertThatIOException()`
*   `assertThatNullPointerException()`

### 3.4。将异常从断言中分离出来

编写单元测试的另一种方法是**将`when`和`then`逻辑写在不同的部分:**

```java
// when
Throwable thrown = catchThrowable(() -> {
    // ...
});

// then
assertThat(thrown)
  .isInstanceOf(ArithmeticException.class)
  .hasMessageContaining("/ by zero");
```

## 4。结论

我们到了。在这篇短文中，我们讨论了使用 AssertJ 对异常执行断言的不同方式。

和往常一样，与本文相关的代码可以从 Github 上的[处获得。](https://web.archive.org/web/20220830145527/https://github.com/eugenp/tutorials/tree/master/testing-modules/assertion-libraries)