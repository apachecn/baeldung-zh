# 用 Java 获取当前堆栈跟踪

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-get-current-stack-trace>

## 1.概观

作为一名 Java 开发人员，在处理[异常](/web/20221026031411/https://www.baeldung.com/java-checked-unchecked-exceptions)时，经常会遇到堆栈跟踪的概念。

在本教程中，**我们将了解什么是堆栈跟踪，以及如何在编程/调试时使用它。**此外，我们还将经历`StackTraceElement` 课。最后，我们将学习如何使用`Thread`和`Throwable` 类来获取它。

## 2.什么是堆栈跟踪？

堆栈跟踪，也称为回溯，是堆栈帧的列表。简单来说，这些帧代表了程序执行过程中的某个时刻。

堆栈帧**包含关于代码已经调用**的方法的信息。这是一个帧列表，从当前方法开始，一直延续到程序启动时。

为了更好地理解这一点，让我们来看一个简单的例子，在这个例子中，我们在一个异常之后转储了当前的堆栈跟踪:

```java
public class DumpStackTraceDemo 
{ 
    public static void main(String[] args) {
        methodA(); 
    } 

    public static void methodA() {
        try {
            int num1 = 5/0; // java.lang.ArithmeticException: divide by zero
        }
        catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

在上面的例子中，`methodA()`抛出`ArithmeticException,` ,后者转储当前堆栈跟踪到`catch` 块中:

```java
java.lang.ArithmeticException: / by zero
at main.java.com.baeldung.tutorials.DumpStackTraceDemo.methodA(DumpStackTraceDemo.java:11)
at main.java.com.baeldung.tutorials.DumpStackTraceDemo.main(DumpStackTraceDemo.java:6)
```

## 3。`StackTraceElement`班

**堆栈跟踪由`StackTraceElement`类的元素组成。**我们可以用下面的方法分别得到类名和方法名:

*   `getClassName`–返回包含当前执行点的类的完全限定名。
*   `getMethodName`–返回包含由该堆栈跟踪元素表示的执行点的方法的名称。

我们可以在 Java API 文档的[中的`StackTraceElement`类中看到方法及其细节的完整列表。](https://web.archive.org/web/20221026031411/https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/lang/StackTraceElement.html)

## 4.使用`Thread`类获取堆栈跟踪

我们可以通过调用`Thread`实例上的`getStackTrace()`方法从线程中获得堆栈跟踪。它返回一个`StackTraceElement`数组，从中可以找到线程堆栈帧的详细信息。

让我们看一个例子:

```java
public class StackTraceUsingThreadDemo {

    public static void main(String[] args) {
        methodA();
    }

    public static StackTraceElement[] methodA() {
        return methodB();
    }

    public static StackTraceElement[] methodB() {
        Thread thread = Thread.currentThread();
        return thread.getStackTrace();
    }
}
```

在上面的类中，方法调用以下面的方式发生—`main() -> methodA() -> methodB() -> getStackTrace().`

让我们用下面的测试用例来验证它，其中测试用例方法正在调用`methodA()`:

```java
@Test
public void whenElementIsFetchedUsingThread_thenCorrectMethodAndClassIsReturned() {
    StackTraceElement[] stackTrace = new StackTraceUsingThreadDemo().methodA();

    StackTraceElement elementZero = stackTrace[0];
    assertEquals("java.lang.Thread", elementZero.getClassName());
    assertEquals("getStackTrace", elementZero.getMethodName());

    StackTraceElement elementOne = stackTrace[1];
    assertEquals("com.baeldung.tutorials.StackTraceUsingThreadDemo", elementOne.getClassName());
    assertEquals("methodB", elementOne.getMethodName());

    StackTraceElement elementTwo = stackTrace[2];
    assertEquals("com.baeldung.tutorials.StackTraceUsingThreadDemo", elementTwo.getClassName());
    assertEquals("methodA", elementTwo.getMethodName());

    StackTraceElement elementThree = stackTrace[3];
    assertEquals("test.java.com.baeldung.tutorials.CurrentStacktraceDemoUnitTest", elementThree.getClassName());
    assertEquals("whenElementIsFetchedUsingThread_thenCorrectMethodAndClassIsReturned", elementThree.getMethodName());
}
```

在上面的测试案例中，我们使用 `methodB() of StackTraceUsingThreadDemo`类获取了一个`StackTraceElement` 数组。然后，使用`StackTraceElement`类的`getClassName()`和`getMethodName() `方法验证堆栈跟踪中的方法和类名。

## 5.使用`Throwable`类获取堆栈跟踪

当任何 Java 程序抛出一个`Throwable`对象时，我们可以通过调用`getStackTrace()`方法获得一个`StackTraceElement`对象的数组，而不是简单地将它打印在控制台上或记录下来。

让我们看一个例子:

```java
public class StackTraceUsingThrowableDemo {

    public static void main(String[] args) {
        methodA(); 
    } 

    public static StackTraceElement[] methodA() {
        try {
            methodB();
        } catch (Throwable t) {
            return t.getStackTrace();
        }
        return null;
    }

    public static void methodB() throws Throwable {
        throw new Throwable("A test exception");
    }
}
```

这里，方法调用以下面的方式发生—`main() -> methodA() -> methodB() -> getStackTrace().`

让我们用一个测试来验证它:

```java
@Test
public void whenElementIsFecthedUsingThrowable_thenCorrectMethodAndClassIsReturned() {
    StackTraceElement[] stackTrace = new StackTraceUsingThrowableDemo().methodA();

    StackTraceElement elementZero = stackTrace[0];
    assertEquals("com.baeldung.tutorials.StackTraceUsingThrowableDemo", elementZero.getClassName());
    assertEquals("methodB", elementZero.getMethodName());

    StackTraceElement elementOne = stackTrace[1];
    assertEquals("com.baeldung.tutorials.StackTraceUsingThrowableDemo", elementOne.getClassName());
    assertEquals("methodA", elementOne.getMethodName());

    StackTraceElement elementThree = stackTrace[2];
    assertEquals("test.java.com.baeldung.tutorials.CurrentStacktraceDemoUnitTest", elementThree.getClassName());
    assertEquals("whenElementIsFecthedUsingThrowable_thenCorrectMethodAndClassIsReturned", elementThree.getMethodName());
}
```

在上面的测试案例中，我们使用 `methodB() of StackTraceUsingThrowableDemo`类获取了一个`StackTraceElement` 数组。然后，验证方法和类名以理解`StackTraceElement`类数组中元素的顺序。

## 6.结论

在本文中，我们学习了 Java 堆栈跟踪以及如何使用`printStackTrace()` 方法打印它，在出现异常`.` 的情况下，我们还学习了如何使用`Thread`和`Throwable`类获取当前堆栈跟踪。

和往常一样，本文的完整代码示例可以在 GitHub 的[中找到。](https://web.archive.org/web/20221026031411/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-exceptions-4)