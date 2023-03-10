# Java 全局异常处理程序

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-global-exception-handler>

## 1。概述

在本教程中，我们将关注 Java 中的全局异常处理程序。我们将首先讨论异常和异常处理的基础知识。然后，我们将全面了解全局异常处理程序。

要了解关于异常的更多信息，请看一下 Java 中的[异常处理。](/web/20220720104415/https://www.baeldung.com/java-exceptions)

## 2。什么是例外？

异常是运行时或编译时代码序列中出现的异常情况。当程序违反 Java 编程语言的语义约束时，就会出现这种异常情况。

**编译期间发生的异常是`checked exceptions`。**这些异常是`Exception`类的直接子类，有必要在代码中处理这些异常。

**另一种类型的异常是`unchecked exceptions`。编译器在编译时不会检查这些异常。**这些异常是`RuntimeException`类的直接子类，它扩展了`Exception`类。

此外，没有必要在代码中处理运行时异常。

## 3。异常处理程序

Java 是一种健壮的编程语言。使其健壮的核心特性之一是异常处理框架。这意味着程序可以在出错时优雅地退出，而不是崩溃。

**每当异常发生时，一个`E` `xception`对象被构造**，要么由 JVM 要么由执行代码的方法来构造。该对象包含关于异常的信息。**异常处理是处理这个`Exception`对象的一种方式。**

### 3.1。`try-catch`街区

在下面的例子中，`try`块包含了可以抛出异常的代码。`catch`块包含处理该异常的逻辑。

`catch`块捕获`try`块中的代码引发的`Exception`对象:

```java
String string = "01, , 2010";
DateFormat format = new SimpleDateFormat("MM, dd, yyyy");
Date date;
try {
    date = format.parse(string);
} catch (ParseException e) {
    System.out.println("ParseException caught!");
}
```

### 3.2。`throw`和`throws`关键词

或者，该方法也可以选择引发异常，而不是处理它。这意味着处理`Exception`对象的逻辑是在别处编写的。

通常，调用方法在以下情况下处理异常:

```java
public class ExceptionHandler {

    public static void main(String[] args) {

        String strDate = "01, , 2010";
        String dateFormat = "MM, dd, yyyy";

        try {
            Date date = new DateParser().getParsedDate(strDate, dateFormat);
        } catch (ParseException e) {
            System.out.println("The calling method caught ParseException!");
        }
    }
}

class DateParser {

    public Date getParsedDate(String strDate, String dateFormat) throws ParseException {
        DateFormat format = new SimpleDateFormat(dateFormat);

        try {
            return format.parse(strDate);
        } catch (ParseException parseException) {
            throw parseException;
        }		
    }

}
```

接下来，我们将考虑全局异常处理程序，作为处理异常的通用方法。

## 4。全局异常处理程序

*RuntimeException* 的实例是可选处理的。因此，它仍然为在运行时获取长堆栈跟踪打开了一个窗口。为了处理这些， **Java 提供了`UncaughtExceptionHandler `接口**。`Thread`类包含这个作为内部类。

除了这个接口， **Java 1.5 版本还在`Thread`类**中引入了一个静态方法*setDefaultUncaughtExceptionHandler()*。这个方法的参数是一个实现`UncaughtExceptionHandler`接口的处理程序类。

此外，该接口声明了方法`uncaughtException(Thread t, Throwable e)`。当给定的线程`t`由于给定的未捕获异常`e`而终止时，它将被调用。实现类实现该方法，并定义处理这些未捕获异常的逻辑。

让我们考虑下面这个在运行时抛出`ArithmeticException`的例子。我们定义了实现接口`UncaughtExceptionHandler`的类`Handler`。

这个类实现了方法`uncaughtException()`，并定义了处理其中未捕获异常的逻辑:

```java
public class GlobalExceptionHandler {

    public static void main(String[] args) {

        Handler globalExceptionHandler = new Handler();
        Thread.setDefaultUncaughtExceptionHandler(globalExceptionHandler);
        new GlobalExceptionHandler().performArithmeticOperation(10, 0);
    }

    public int performArithmeticOperation(int num1, int num2) {
        return num1/num2;
    }
}

class Handler implements Thread.UncaughtExceptionHandler {

    private static Logger LOGGER = LoggerFactory.getLogger(Handler.class);

    public void uncaughtException(Thread t, Throwable e) {
        LOGGER.info("Unhandled exception caught!");
    }
}
```

这里，当前执行的线程是主线程。因此，它的实例连同引发的异常一起被传递给方法`uncaughtException()`。然后类`Handler`处理这个异常。

这同样适用于未处理的已检查异常。让我们看一个简单的例子:

```java
public static void main(String[] args) throws Exception {
    Handler globalExceptionHandler = new Handler();
    Thread.setDefaultUncaughtExceptionHandler(globalExceptionHandler);
    Path file = Paths.get("");
    Files.delete(file);
}
```

这里，`Files.delete()`方法抛出一个被检查的`IOException,`，这个被检查的`IOException,`由`main()`方法签名进一步抛出。`Handler`也会捕捉这个异常。

这样，`UncaughtExceptionHandler`有助于在运行时管理未处理的异常。然而，它**打破了在靠近起源点**的地方捕捉和处理异常的想法。

## 5。结论

在本文中，我们花时间了解了异常是什么，以及处理它们的基本方法是什么。此外，我们发现全局异常处理程序是`Thread`类的一部分，它处理未捕获的运行时异常。

然后，我们看到了一个示例程序，它抛出一个运行时异常，并使用一个全局异常处理程序来处理它。

本文的示例代码可以在 GitHub 上找到[。](https://web.archive.org/web/20220720104415/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-exceptions-2)