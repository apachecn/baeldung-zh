# Java IOException“打开的文件太多”

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-too-many-open-files>

## 1.介绍

在 Java 中处理文件时，一个常见的缺陷是可能用完可用的文件描述符。

在本教程中，我们将看看这种情况，并提供两种方法来避免这个问题。

## 2.JVM 如何处理文件

尽管 JVM 很好地将我们与操作系统隔离开来，但它将文件管理等底层操作委托给了操作系统。

这意味着，对于我们在 Java 应用程序中打开的每个文件，操作系统将分配一个文件描述符来将文件与我们的 Java 进程相关联。一旦 JVM 处理完文件，它就释放描述符。

现在，让我们深入了解如何触发异常。

## 3.泄漏文件描述符

回想一下，对于 Java 应用程序中的每个文件引用，我们在操作系统中都有一个对应的文件描述符。只有当文件引用实例被释放时，描述符才会被关闭。这将在垃圾收集阶段发生。

但是，如果引用保持活动状态，并且越来越多的文件被打开，那么最终操作系统将用完要分配的文件描述符。此时，它会将这种情况转发给 JVM，这将导致抛出一个`IOException`。

我们可以通过一个简短的单元测试来重现这种情况:

```java
@Test
public void whenNotClosingResoures_thenIOExceptionShouldBeThrown() {
    try {
        for (int x = 0; x < 1000000; x++) {
            FileInputStream leakyHandle = new FileInputStream(tempFile);
        }
        fail("Method Should Have Failed");
    } catch (IOException e) {
        assertTrue(e.getMessage().containsIgnoreCase("too many open files"));
    } catch (Exception e) {
        fail("Unexpected exception");
    }
} 
```

在大多数操作系统上，JVM 进程在完成循环之前会用完文件描述符，从而触发`IOException`。

让我们看看如何通过适当的资源处理来避免这种情况。

## 4.处理资源

正如我们之前所说，文件描述符是由 JVM 进程在[垃圾收集](/web/20221013193919/https://www.baeldung.com/jvm-garbage-collectors)期间释放的。

但是，如果我们没有正确地关闭我们的文件引用，收集器可能会选择不销毁引用，让描述符保持打开，并限制我们可以打开的文件数量。

然而，我们可以很容易地解决这个问题，如果我们打开一个文件，我们确保当我们不再需要它时关闭它。

### 4.1.手动释放引用

在 JDK 8 之前，手动释放引用是确保适当资源管理的一种常见方式。

不仅**我们必须显式关闭我们打开的任何文件**，而且还要确保即使我们的代码失败并抛出异常，我们也会这样做。这意味着使用 [`finally`](/web/20221013193919/https://www.baeldung.com/java-finally-keyword) 关键字:

```java
@Test
public void whenClosingResoures_thenIOExceptionShouldNotBeThrown() {
    try {
        for (int x = 0; x < 1000000; x++) {
            FileInputStream nonLeakyHandle = null;
            try {
                nonLeakyHandle = new FileInputStream(tempFile);
            } finally {
                if (nonLeakyHandle != null) {
                    nonLeakyHandle.close();
                }
            }
        }
    } catch (IOException e) {
        assertFalse(e.getMessage().toLowerCase().contains("too many open files"));
        fail("Method Should Not Have Failed");
    } catch (Exception e) {
        fail("Unexpected exception");
    }
} 
```

由于`finally`块总是被执行，这给了我们正确关闭引用的机会，从而限制了打开描述符的数量。

### 4.2.使用`try-with-resources`

JDK 7 为我们带来了一种更清洁的资源处理方式。它通常被称为 [`try-with-resources`](/web/20221013193919/https://www.baeldung.com/java-try-with-resources) ，允许我们通过将资源包含在`try` 定义中来委托资源的处置:

```java
@Test
public void whenUsingTryWithResoures_thenIOExceptionShouldNotBeThrown() {
    try {
        for (int x = 0; x < 1000000; x++) {
            try (FileInputStream nonLeakyHandle = new FileInputStream(tempFile)) {
                // do something with the file
            }
        }
    } catch (IOException e) {
        assertFalse(e.getMessage().toLowerCase().contains("too many open files"));
        fail("Method Should Not Have Failed");
    } catch (Exception e) {
        fail("Unexpected exception");
    }
}
```

这里，我们在`try` 语句中声明了`nonLeakyHandle`。因此，Java 会为我们关闭资源，而不是我们需要使用`finally.`

## 5.结论

正如我们所看到的，未能正确关闭打开的文件会导致一个复杂的异常，并对整个程序产生影响。通过适当的资源处理，我们可以确保这个问题永远不会出现。

这篇文章的完整源代码可以在 GitHub 的[上找到。](https://web.archive.org/web/20221013193919/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-exceptions-2)