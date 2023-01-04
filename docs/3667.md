# Java finally 关键字指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-finally-keyword>

## 1.概观

在本教程中，我们将探索 Java 中的关键字`finally`。我们将看到如何在错误处理中与`try/catch`块一起使用它。尽管`finally`旨在保证代码的执行，我们将讨论 JVM 不执行它的异常情况。

我们还将讨论一些常见的陷阱，其中一个`finally`块可能会产生意想不到的结果。

## 2.什么是`finally?`

`finally`定义了我们与`try` 关键字一起使用的代码块。它定义了在方法完成之前，总是在`try`和任何`catch`块之后运行的代码。

**`finally`块执行，不管是否抛出异常或捕获异常**。

### 2.1.一个简单的例子

让我们看看`try-catch-finally`块中的`finally `:

```
try {
    System.out.println("The count is " + Integer.parseInt(count));
} catch (NumberFormatException e) {
    System.out.println("No count");
} finally {
    System.out.println("In finally");
} 
```

在这个例子中，不管参数`count`的值是多少，JVM 都会执行`finally`块并打印`“In finally”`。

### 2.2.使用没有`catch`块的`finally`

同样，我们可以将**块与`try`块一起使用`finally` 块，而不管`catch`块是否存在**:

```
try {
    System.out.println("Inside try");
} finally {
    System.out.println("Inside finally");
}
```

我们会得到输出:

```
Inside try
Inside finally
```

### 2.3.为什么`finally`有用

我们通常使用`finally`块来执行清理代码，如关闭连接、关闭文件或释放线程，因为它执行时不考虑异常。

**注意:** [try-with-resources](/web/20221028145408/https://www.baeldung.com/java-try-with-resources) 也可以用来关闭资源而不是`finally`块。

## 3.当执行`finally`时

让我们看看 JVM 执行`finally`块时的所有排列，这样我们可以更好地理解它。

### 3.1.不会引发任何异常

当`try`块完成时，执行`finally `块，即使没有异常:

```
try {
    System.out.println("Inside try");
} finally {
    System.out.println("Inside finally");
}
```

在这个例子中，我们没有从`try`块抛出异常。因此，JVM 执行`try`和`finally`块中的所有代码。

这将输出:

```
Inside try
Inside finally
```

### 3.2.异常被抛出且未被处理

如果有一个异常并且没有被捕获，那么`finally`块仍然被执行:

```
try {
    System.out.println("Inside try");
    throw new Exception();
} finally {
    System.out.println("Inside finally");
}
```

**即使在未处理异常的情况下，JVM 也会执行 `finally`块。**

输出将是:

```
Inside try
Inside finally
Exception in thread "main" java.lang.Exception
```

### 3.3.异常被抛出并得到处理

如果出现异常并且被`catch`块捕获，那么`finally`块仍然被执行:

```
try {
    System.out.println("Inside try");
    throw new Exception();
} catch (Exception e) {
    System.out.println("Inside catch");
} finally {
    System.out.println("Inside finally");
}
```

在这种情况下，`catch`块处理抛出的异常，然后 JVM 执行`finally`块并产生输出:

```
Inside try
Inside catch
Inside finally
```

### 3.4.方法从`try`块返回

即使从该方法返回也不会阻止`finally`块运行:

```
try {
    System.out.println("Inside try");
    return "from try";
} finally {
    System.out.println("Inside finally");
}
```

这里，即使方法有一个`return`语句，JVM 在将控制权移交给调用方法之前还是会执行`finally`块。

我们将得到输出:

```
Inside try
Inside finally
```

### 3.5.方法从`catch`块返回

当`catch`块包含一个`return`语句时，`finally`块仍然被调用:

```
try {
    System.out.println("Inside try");
    throw new Exception();
} catch (Exception e) {
    System.out.println("Inside catch");
    return "from catch";
} finally {
    System.out.println("Inside finally");
}
```

当我们从`try`块抛出一个异常时，`catch`块处理这个异常。虽然在`catch`块中有一个 return 语句，但是 JVM 在将控制权移交给调用方法之前执行了`finally`块，它输出:

```
Inside try
Inside catch
Inside finally
```

## 4.当`finally`未执行时

尽管我们总是希望 JVM 执行`finally`块中的语句，但是在某些情况下，JVM 不会执行`finally`块。

我们可能已经预料到，如果操作系统停止了我们的程序，那么程序就没有机会执行它所有的代码。我们也可以采取一些措施来阻止一个挂起的`finally`块的执行。

### 4.1.调用`System.exit`

在这种情况下，我们通过调用 [`System.exit`](/web/20221028145408/https://www.baeldung.com/java-system-exit) 来终止 JVM，因此 JVM 不会执行我们的`finally`块:

```
try {
    System.out.println("Inside try");
    System.exit(1);
} finally {
    System.out.println("Inside finally");
}
```

这将输出:

```
Inside try
```

### 4.2.调用`halt`

与`System.exit`类似，对 [`Runtime.halt`](/web/20221028145408/https://www.baeldung.com/java-runtime-halt-vs-system-exit) 的调用也停止执行，JVM 不执行任何`finally`块:

```
try {
    System.out.println("Inside try");
    Runtime.getRuntime().halt(1);
} finally {
    System.out.println("Inside finally");
}
```

因此，输出将是:

```
Inside try
```

### 4.3.守护线程

如果[守护线程](/web/20221028145408/https://www.baeldung.com/java-daemon-thread)进入`try/finally`块的执行，并且所有其他非守护线程在守护线程执行`finally`块之前退出，JVM 不会等待守护线程完成`finally`块的执行:

```
Runnable runnable = () -> {
    try {
        System.out.println("Inside try");
    } finally {
        try {
            Thread.sleep(1000);
            System.out.println("Inside finally");
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
};
Thread regular = new Thread(runnable);
Thread daemon = new Thread(runnable);
daemon.setDaemon(true);
regular.start();
Thread.sleep(300);
daemon.start();
```

在本例中，`runnable`一进入方法就打印`“Inside try”`，并在打印`“Inside finally”`前等待 1 秒钟。

这里，我们启动`regular` 线程和`daemon` 线程有一个小的延迟。当`regular` 线程执行`finally`块时，`daemon` 线程仍在`try`块内等待。当`regular` 线程完成执行并退出时，JVM 也退出，不等待`daemon` 线程完成`finally`块。

以下是输出结果:

```
Inside try
Inside try
Inside finally
```

### 4.4.JVM 到达一个无限循环

这里有一个包含无限`while`循环的`try`块:

```
try {
    System.out.println("Inside try");
    while (true) {
    }
} finally {
    System.out.println("Inside finally");
}
```

尽管这不是针对`finally`的，但值得一提的是，如果`try`或`catch`块包含一个无限循环，JVM 将永远不会到达该循环之外的任何块。

## 5.常见陷阱

使用`finally`块时，我们必须避免一些常见的陷阱。

虽然这完全合法，**但是从`finally`块、**中使用`return`语句或抛出异常被认为是不好的做法，我们应该不惜一切代价避免。

### 5.1.忽略例外

`finally`块中的`return`语句忽略一个未捕获的异常:

```
try {
    System.out.println("Inside try");
    throw new RuntimeException();
} finally {
    System.out.println("Inside finally");
    return "from finally";
}
```

在这种情况下，该方法忽略抛出的`RuntimeException`并返回值`“from finally”`。

### 5.2.忽略其他`return`语句

`finally`块中的`return`语句忽略`try`或`catch`块中的任何其他返回语句。只有`finally`块中的`return`语句执行:

```
try {
    System.out.println("Inside try");
    return "from try";
} finally {
    System.out.println("Inside finally");
    return "from finally";
}
```

在这个例子中，该方法总是返回`“from finally”`并完全忽略`try`块中的`return`语句。这可能是一个很难发现的错误，这就是为什么我们应该避免在`finally`块中使用`return`。

### 5.3.更改抛出或返回的内容

此外，在从`finally`块抛出异常的情况下，该方法忽略抛出的异常或`try`和`catch`块中的`return`语句:

```
try {
    System.out.println("Inside try");
    return "from try";
} finally {
    throw new RuntimeException();
}
```

这个方法从不返回值，总是抛出一个`RuntimeException`。

虽然我们可能不会像本例中那样故意从`finally`块抛出异常，但我们仍然可能会遇到这个问题。当我们在`finally`块中使用的清理方法抛出异常时，就会发生这种情况。

## 6.结论

在本文中，我们讨论了`finally`块在 Java 中的作用以及如何使用它们。然后，我们看了 JVM 执行它们的不同情况，以及一些不执行的情况。

最后，我们看了一些与使用`finally`块相关的常见陷阱。

和往常一样，本教程中使用的源代码可以从 GitHub 上的[处获得。](https://web.archive.org/web/20221028145408/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-lang-2)