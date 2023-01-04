# Java 中包装与重新抛出异常

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-wrapping-vs-rethrowing-exceptions>

## 1.概观

Java 中的 `throw`关键字用于显式抛出[一个定制的异常或内置的异常。](/web/20221207032034/https://www.baeldung.com/java-exceptions)但是有时候在`catch` 块中，我们需要再次抛出同样的异常。这导致重新抛出异常。

在本教程中，我们将讨论重新抛出异常的两种最常见的方式。

## 2.重新引发异常

有时，在将异常传播到更高层之前，我们可能希望执行一些活动。例如，我们可能希望回滚数据库事务、记录异常或发送电子邮件。

我们可以在 catch 块中执行这样的活动，然后再次抛出异常。通过这种方式，更高级别得到通知，系统中发生了异常。

让我们用一个例子来理解我们的案例。

下面，我们再次抛出同样的异常。而且，在抛出之前，我们会记录一条错误消息:

```
String name = null;

try {
    return name.equals("Joe"); // causes NullPointerException
} catch (Exception e) {
    // log
    throw e;
}
```

控制台将显示以下消息:

```
Exception in thread "main" java.lang.NullPointerException
  at com.baeldung.exceptions.RethrowSameExceptionDemo.main(RethrowSameExceptionDemo.java:16)
```

正如我们所看到的，我们的代码只是重新抛出它捕捉到的任何异常。正因为如此，我们得到了没有任何变化的**原始堆栈跟踪**。

## 3.包装异常

现在，让我们来看看一种不同的方法。

在这种情况下，我们将在不同异常的构造函数中传递相同的异常作为引用:

```
String name = null;

try {
    return name.equals("Joe"); // causes NullPointerException
} catch (Exception e) {
    // log
    throw new IllegalArgumentException(e);
}
```

控制台将显示:

```
Exception in thread "main" java.lang.IllegalArgumentException: java.lang.NullPointerException
  at com.baeldung.exceptions.RethrowDifferentExceptionDemo.main(RethrowDifferentExceptionDemo.java:24)
Caused by: java.lang.NullPointerException
  at com.baeldung.exceptions.RethrowDifferentExceptionDemo.main(RethrowDifferentExceptionDemo.java:18) 
```

这一次，我们看到了原始异常和包装异常。这样，**我们的`IllegalArgumentException` 实例将原来的`NullPointerException` 包装成一个原因**。因此，我们可以显示更具体的异常，而不是显示一般的异常。

## 4.结论

在这篇短文中，我们介绍了重新抛出原始异常与首先包装它之间的主要区别。**两种方式** **显示异常消息**的方式互不相同。

根据我们的需求，我们可以通过使用第二种方法来重新抛出同一个异常，或者用某个特定的异常来包装它。****第二种方法看起来更简洁，也更容易回溯异常**。**

 **和往常一样，这个项目可以在 GitHub 上获得[。](https://web.archive.org/web/20221207032034/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-exceptions-2)**