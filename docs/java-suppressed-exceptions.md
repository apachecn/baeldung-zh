# Java 隐藏异常

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-suppressed-exceptions>

## 1.介绍

在这个快速教程中，我们将学习 Java 中的隐藏异常。简而言之，被抑制的异常是被抛出但不知何故被忽略的异常。Java 中这种情况的常见场景是当`finally`块抛出异常时。任何最初在`try`块中抛出的异常都会被抑制。

从 Java 7 开始，我们现在可以在`[Throwable](https://web.archive.org/web/20221205223425/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/Throwable.html)`类上使用两种方法来处理被抑制的异常:`addSuppressed`和`getSuppressed`。我们应该注意到，Java 7 中也引入了`[try-with-resources](/web/20221205223425/https://www.baeldung.com/java-try-with-resources)`构造。我们将在示例中看到它们是如何关联的。

## 2.动作中隐藏的异常

### 2.1.隐藏的异常情况

让我们先快速看一个例子，其中原始异常被发生在`finally`块中的异常所抑制:

```java
public static void demoSuppressedException(String filePath) throws IOException {
    FileInputStream fileIn = null;
    try {
        fileIn = new FileInputStream(filePath);
    } catch (FileNotFoundException e) {
        throw new IOException(e);
    } finally {
        fileIn.close();
    }
}
```

只要我们提供一个现有文件的路径，就不会抛出任何异常，并且该方法将按预期工作。

但是，假设我们提供了一个不存在的文件:

```java
@Test(expected = NullPointerException.class)
public void givenNonExistentFileName_whenAttemptFileOpen_thenNullPointerException() throws IOException {
    demoSuppressedException("/non-existent-path/non-existent-file.txt");
}
```

在这种情况下，`try`块在试图打开不存在的文件时会抛出一个`FileNotFoundException`。因为`fileIn`对象从未被初始化，当我们试图在`finally`块中关闭它时，它会抛出一个`NullPointerException`。我们的调用方法将只得到`NullPointerException`，并且不容易发现原来的问题是什么:文件不存在。

### 2.2.添加隐藏的异常

现在让我们看看如何利用`Throwable.addSuppressed`方法来提供原始异常:

```java
public static void demoAddSuppressedException(String filePath) throws IOException {
    Throwable firstException = null;
    FileInputStream fileIn = null;
    try {
        fileIn = new FileInputStream(filePath);
    } catch (IOException e) {
        firstException = e;
    } finally {
        try {
            fileIn.close();
        } catch (NullPointerException npe) {
            if (firstException != null) {
                npe.addSuppressed(firstException);
            }
            throw npe;
        }
    }
}
```

让我们进入单元测试，看看`getSuppressed`在这种情况下是如何工作的:

```java
try {
    demoAddSuppressedException("/non-existent-path/non-existent-file.txt");
} catch (Exception e) {
    assertThat(e, instanceOf(NullPointerException.class));
    assertEquals(1, e.getSuppressed().length);
    assertThat(e.getSuppressed()[0], instanceOf(FileNotFoundException.class));
}
```

我们现在可以从提供的隐藏异常数组中访问原始异常。

### 2.3.使用`try-with-resources`

最后，让我们看一个使用`try-with-resources`的例子，其中`close`方法抛出一个异常。Java 7 引入了用于资源管理的`try-with-resources`结构和`AutoCloseable`接口。

首先，让我们创建一个实现`AutoCloseable`的资源:

```java
public class ExceptionalResource implements AutoCloseable {

    public void processSomething() {
        throw new IllegalArgumentException("Thrown from processSomething()");
    }

    @Override
    public void close() throws Exception {
        throw new NullPointerException("Thrown from close()");
    }
}
```

接下来，让我们在`try-with-resources`块中使用`ExceptionalResource`:

```java
public static void demoExceptionalResource() throws Exception {
    try (ExceptionalResource exceptionalResource = new ExceptionalResource()) {
        exceptionalResource.processSomething();
    }
}
```

最后，让我们回顾一下我们的单元测试，看看异常是如何出现的:

```java
try {
    demoExceptionalResource();
} catch (Exception e) {
    assertThat(e, instanceOf(IllegalArgumentException.class));
    assertEquals("Thrown from processSomething()", e.getMessage());
    assertEquals(1, e.getSuppressed().length);
    assertThat(e.getSuppressed()[0], instanceOf(NullPointerException.class));
    assertEquals("Thrown from close()", e.getSuppressed()[0].getMessage());
}
```

我们应该注意，当使用`AutoCloseable`、**时，被抑制的是`close`方法中抛出的异常**。引发原始异常。

## 3.结论

在这个简短的教程中，我们学习了什么是隐含异常以及它们是如何发生的。然后，我们看到了如何使用`addSuppressed`和`getSuppressed`方法来访问那些被抑制的异常。最后，我们看到了使用`try-with-resources` 块时被抑制的异常是如何工作的。

与往常一样，GitHub 上的示例代码[可用。](https://web.archive.org/web/20221205223425/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-exceptions-2)