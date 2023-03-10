# 在 Java 8 中抛出异常

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-optional-throw-exception>

## 1.介绍

在本教程中，我们将展示如何在一个`Optional` i `s`为空时抛出一个自定义异常。

如果你想更深入地了解`Optional, `，看看我们的完整[指南](/web/20221205215609/https://www.baeldung.com/java-optional)，在这里。

## 2.`Optional.orElseThrow`

简单地说，如果这个值存在，那么`isPresent()`将返回`true`，调用`get() `将返回这个值。否则抛出`NoSuchElementException`。

还有一个方法`**orElseThrow(Supplier<? extends X> exceptionSupplier)**` 允许我们提供一个定制的异常实例。这个方法只有在存在的情况下才会返回值。否则，它将抛出一个由提供的供应商创建的异常。

## 3.在活动

假设**我们有一个返回可空结果的方法:**

```java
public String findNameById(String id) {
    return id == null ? null : "example-name";
}
```

现在我们将调用我们的`findNameById(String id)`方法两次，并通过使用 `ofNullable(T value)`方法用一个`Optional`包装结果。

**`Optional`为创建新实例** `. `提供了一个静态工厂方法，这个方法叫做`ofNullable(T value)`。然后我们可以调用`orElseThrow.`

我们可以通过运行以下测试来验证该行为:

```java
@Test
public void whenIdIsNull_thenExceptionIsThrown() {
    assertThrows(InvalidArgumentException.class, () -> Optional
      .ofNullable(personRepository.findNameById(null))
      .orElseThrow(InvalidArgumentException::new));
}
```

根据我们的实现，`findNameById `将返回`null`。所以新的`InvalidArgumentException `将从` orElseThrow `方法中抛出。

我们可以用一个非空参数调用这个方法。那么，我们不会得到一个`InvalidArgumentException:`

```java
@Test
public void whenIdIsNonNull_thenNoExceptionIsThrown() {
    assertAll(() -> Optional
      .ofNullable(personRepository.findNameById("id"))
      .orElseThrow(RuntimeException::new));
} 
```

## 4.结论

在这篇简短的文章中，我们讨论了如何从 Java 8 `Optional. `抛出异常

像往常一样，我们将源代码[放在我们的 GitHub](https://web.archive.org/web/20221205215609/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-optional) 上。