# 如何在 Java 中处理 InterruptedException

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-interrupted-exception>

## 1.介绍

在本教程中，我们将探索 Java 的`InterruptedException`。首先，我们将通过一个例子快速浏览一个线程的生命周期。接下来，我们将看到在多线程应用中工作如何潜在地导致`InterruptedException`。最后，我们将看到如何处理这个异常。

## 2.多线程基础

在讨论中断之前，我们先来回顾一下多线程。多线程是同时执行两个或更多线程的过程。一个 Java 应用程序从一个与`main()`方法相关联的被称为主线程的线程开始。然后，这个主线程可以启动其他线程。

线程是轻量级的，这意味着它们运行在相同的内存空间中。因此，他们可以很容易地相互交流。让我们来看看线程的[生命周期:](/web/20221208143832/https://www.baeldung.com/java-thread-lifecycle)

[![Threadlifecycle](img/d66fb80423b2bbe7c3c726453946e926.png)](/web/20221208143832/https://www.baeldung.com/wp-content/uploads/2021/04/Threadlifecycle.jpg)

一旦我们创建了一个新线程，它就处于`NEW`状态。它一直保持这种状态，直到程序使用`start()`方法启动线程。

在线程上调用`start()`方法会将其置于`RUNNABLE`状态。处于这种状态的线程要么正在运行，要么准备运行。

当一个线程正在等待一个监视器锁，并试图访问被其他线程锁定的代码时，它进入`BLOCKED`状态。

一个线程可以被各种事件置于*等待*状态，比如调用`wait()`方法。在这种状态下，一个线程正在等待来自另一个线程的信号。

当一个线程完成执行或者异常终止时，它将进入`TERMINATED`状态。**线程可以被中断，当一个线程被中断时会抛出`InterruptedException`。**

在接下来的章节中，我们将详细了解`InterruptedException`，并学习如何对其做出回应。

## 3.什么是中断异常？

一个 [`InterruptedException`](https://web.archive.org/web/20221208143832/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/InterruptedException.html) 当一个线程在等待、睡眠或被占用时被中断，就会抛出。换句话说，一些代码在我们的线程上调用了`interrupt()`方法。是一个[检查过的异常](/web/20221208143832/https://www.baeldung.com/java-checked-unchecked-exceptions)，Java 中很多阻塞操作都可以抛出。

### 3.1.中断

中断系统的目的是提供一个定义良好的框架，允许线程中断其他线程中的任务(可能是耗时的任务)。考虑中断的一个好方法是，它实际上并不中断正在运行的线程——它只是请求线程在下一个方便的时机中断自己。

### 3.2.阻塞和可中断方法

线程阻塞的原因有几个:等待从`Thread.sleep` `(),` 中醒来，等待获取锁，等待 I/O 完成，或者等待另一个线程的计算结果，等等。

**`InterruptedException`通常由所有阻塞方法抛出，以便对其进行处理并执行纠正措施。**Java 中有几种抛出 [`InterruptedException`](https://web.archive.org/web/20221208143832/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/class-use/InterruptedException.html) 的方法。这些方法包括`Thread.sleep()`、`Thread.join()`、`Object`类的`wait()`方法、`BlockingQueue`类的`put()`和`take()`方法等等。

### 3.3.线程中的中断方法

让我们快速看一下`Thread`类处理中断的一些关键方法:

```java
public void interrupt() { ... }
public boolean isInterrupted() { ... }
public static boolean interrupted() { ... }
```

**`Thread`提供了`interrupt()`中断线程的方法，要查询线程是否被中断，我们可以使用`isInterrupted()`方法**。有时，我们可能希望测试当前线程是否被中断，如果是，立即抛出这个异常。在这里，我们可以使用`interrupted()`方法:

```java
if (Thread.interrupted()) {
    throw new InterruptedException();
}
```

### 3.4.中断状态标志

中断机制是使用称为中断状态的标志来实现的。**每个线程都有一个代表其中断状态的`boolean`属性。调用`Thread.interrupt()`设置该标志。** **当线程通过调用`static`方法`Thread.interrupted()`检查中断时，中断状态被清除。**

为了响应中断请求，我们必须处理`InterruptedException.`我们将在下一节看到如何处理。

## 4.如何处理一个`InterruptedException`

值得注意的是，线程调度是依赖于 JVM 的。这很自然，因为 JVM 是一个虚拟机，需要本地操作系统资源来支持多线程。因此，我们不能保证我们的线程永远不会被中断。

中断是对一个线程的指示，它应该停止正在做的事情，去做别的事情。更具体地说，如果我们正在编写一些将由`[Executor](https://web.archive.org/web/20221208143832/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/concurrent/Executor.html)` 或其他线程管理机制执行的代码，我们需要确保我们的代码对中断做出迅速响应。否则，我们的应用程序可能会导致[死锁](/web/20221208143832/https://www.baeldung.com/java-deadlock-livelock)。

处理`InterruptedException`的实用策略很少。让我们来看看它们。

### 4.1.传播`InterruptedException`

我们可以允许`InterruptedException`在调用栈中向上传播，例如，**通过依次给每个方法添加一个`throws`子句，并让调用者决定如何处理中断**。这可能涉及到我们没有捕获异常，或者捕获并重新抛出异常。让我们通过一个例子来实现这一点:

```java
public static void propagateException() throws InterruptedException {
    Thread.sleep(1000);
    Thread.currentThread().interrupt();
    if (Thread.interrupted()) {
        throw new InterruptedException();
    }
}
```

这里，我们检查线程是否被中断，如果是，我们抛出一个 *InterruptedException* 。现在，让我们调用`propagateException()`方法:

```java
public static void main(String... args) throws InterruptedException {
    propagateException();
}
```

当我们试图运行这段代码时，我们将收到一个带有堆栈跟踪的`InterruptedException`:

```java
Exception in thread "main" java.lang.InterruptedException
    at com.baeldung.concurrent.interrupt.InterruptExample.propagateException(InterruptExample.java:16)
    at com.baeldung.concurrent.interrupt.InterruptExample.main(InterruptExample.java:7)
```

**尽管这是响应异常的最明智的方式，但有时我们不能抛出它**——例如，当我们的代码是`Runnable`的一部分时。在这种情况下，我们必须抓住它，恢复状态。我们将在下一节看到如何处理这种情况。

### 4.2.恢复中断

有些情况下我们不能传播`InterruptedException.`例如，假设我们的任务是由一个`Runnable`定义的，或者覆盖一个不抛出任何检查异常的方法。在这种情况下，我们可以保留中断。做到这一点的标准方法是恢复中断的状态。

**我们可以再次调用`interrupt()`方法(它会将标志设置回`true`)，这样调用栈中更高层的代码可以看到一个中断被发出。**例如，让我们中断一个线程，并尝试访问其中断状态:

```java
public class InterruptExample extends Thread {
    public static Boolean restoreTheState() {
        InterruptExample thread1 = new InterruptExample();
        thread1.start();
        thread1.interrupt();
        return thread1.isInterrupted();
    }
}
```

下面是处理这个中断并恢复中断状态的`run()`方法:

```java
public void run() {
    try {
        Thread.sleep(1000);
    } catch (InterruptedException e) {
        Thread.currentThread().interrupt();  //set the flag back to <code>true
 } }
```

最后，让我们测试状态:

```java
assertTrue(InterruptExample.restoreTheState());
```

尽管 Java 异常涵盖了所有的异常情况和条件，但我们可能希望抛出一个特定的定制异常，该异常是代码和业务逻辑所独有的。在这里，我们可以创建自定义异常来处理中断。我们将在下一节看到它。

### 4.3.自定义异常处理

定制异常提供了添加不属于标准 Java 异常的属性和方法的灵活性。因此，**根据具体情况，以自定义方式处理中断是完全有效的**。

我们可以完成额外的工作，让应用程序优雅地处理中断请求。例如，当一个线程正在休眠或等待一个 I/O 操作，并且它接收到中断时，我们可以在终止线程之前关闭任何资源。

让我们创建一个名为`CustomInterruptedException`的自定义检查异常:

```java
public class CustomInterruptedException extends Exception {
    CustomInterruptedException(String message) {
        super(message);
    }
}
```

**线程中断时我们可以抛出我们的`CustomInterruptedException`**:

```java
public static void throwCustomException() throws Exception {
    Thread.sleep(1000);
    Thread.currentThread().interrupt();
    if (Thread.interrupted()) {
        throw new CustomInterruptedException("This thread was interrupted");
    }
}
```

让我们看看如何检查异常是否抛出了正确的消息:

```java
@Test
 public void whenThrowCustomException_thenContainsExpectedMessage() {
    Exception exception = assertThrows(CustomInterruptedException.class, () -> InterruptExample.throwCustomException());
    String expectedMessage = "This thread was interrupted";
    String actualMessage = exception.getMessage();

    assertTrue(actualMessage.contains(expectedMessage));
}
```

**同样，我们可以处理异常并恢复中断状态**:

```java
public static Boolean handleWithCustomException() throws CustomInterruptedException{
    try {
        Thread.sleep(1000);
        Thread.currentThread().interrupt();
    } catch (InterruptedException e) {
        Thread.currentThread().interrupt();
        throw new CustomInterruptedException("This thread was interrupted...");
    }
    return Thread.currentThread().isInterrupted();
}
```

我们可以通过检查中断状态来测试代码，以确保它返回`true`:

```java
assertTrue(InterruptExample.handleWithCustomException());
```

## 5.结论

在本教程中，我们看到了处理`InterruptedException`的不同方法。如果处理得当，我们可以平衡应用程序的响应性和健壮性。和往常一样，这里使用的代码片段可以从 GitHub 的[上获得。](https://web.archive.org/web/20221208143832/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-concurrency-basic-3)