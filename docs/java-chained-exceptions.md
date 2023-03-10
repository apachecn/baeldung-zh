# Java 中的链式异常

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-chained-exceptions>

## 1。概述

在本文中，我们将非常简要地看一下什么是`Exception`，并深入讨论 Java 中的链式异常。

简单来说，`exception`就是扰乱程序执行正常流程的事件。现在让我们来看看如何将异常链接起来以获得更好的语义。

## 2。连锁异常

链式`Exception`有助于识别应用程序中一个异常导致另一个`Exception`的情况。

**例如，考虑一个方法，由于试图被零除而抛出`ArithmeticException`** ，但是异常的实际原因是导致除数为零的 I/O 错误。该方法将把`ArithmeticException`扔给调用者。打电话的人不会知道`Exception`的真正原因。链式`Exception`用于这种情况。

这个概念是在 JDK 1.4 中引入的。

让我们看看 Java 是如何支持链式异常的。

## 3。`Throwable` 阶级

类有一些支持链式异常的构造函数和方法。首先，让我们看看构造函数。

*   **`Throwable(Throwable cause)`** `–` `Throwable` 有一个单一的参数，指定了一个`Exception`的实际原因。
*   **`Throwable(String desc, Throwable cause)`** `–` 这个构造函数接受一个`Exception`描述和一个`Exception`的实际原因。

接下来，让我们看看这个类提供的方法:

*   **`getCause()`方法**–该方法返回与当前`Exception`相关的实际原因。
*   **`initCause()`方法**——通过调用`Exception`来设置一个潜在的原因。

## 4。示例

现在，让我们看看这个例子，我们将设置自己的`Exception`描述并抛出一个链接的`Exception`:

```java
public class MyChainedException {

    public void main(String[] args) {
        try {
            throw new ArithmeticException("Top Level Exception.")
              .initCause(new IOException("IO cause."));
        } catch(ArithmeticException ae) {
            System.out.println("Caught : " + ae);
            System.out.println("Actual cause: "+ ae.getCause());
        }
    }    
}
```

正如猜测的那样，这将导致:

```java
Caught: java.lang.ArithmeticException: Top Level Exception.
Actual cause: java.io.IOException: IO cause.
```

## 5。为什么是链式异常？

我们需要将异常链接起来，以使日志可读。我们来写两个例子。第一种是不链接异常，第二种是链接异常。稍后，我们将比较日志在这两种情况下的行为。

首先，我们将创建一系列异常:

```java
class NoLeaveGrantedException extends Exception {

    public NoLeaveGrantedException(String message, Throwable cause) {
        super(message, cause);
    }

    public NoLeaveGrantedException(String message) {
        super(message);
    }
}

class TeamLeadUpsetException extends Exception {
    // Both Constructors
}
```

现在，让我们开始在代码示例中使用上述异常。

### 5.1。无链接

让我们编写一个不链接自定义异常的示例程序。

```java
public class MainClass {

    public void main(String[] args) throws Exception {
        getLeave();
    }

    void getLeave() throws NoLeaveGrantedException {
        try {
            howIsTeamLead();
        } catch (TeamLeadUpsetException e) {
            e.printStackTrace();
            throw new NoLeaveGrantedException("Leave not sanctioned.");
        }
    }

    void howIsTeamLead() throws TeamLeadUpsetException {
        throw new TeamLeadUpsetException("Team Lead Upset");
    }
}
```

在上面的示例中，日志如下所示:

```java
com.baeldung.chainedexception.exceptions.TeamLeadUpsetException: 
  Team lead Upset
    at com.baeldung.chainedexception.exceptions.MainClass
      .howIsTeamLead(MainClass.java:46)
    at com.baeldung.chainedexception.exceptions.MainClass
      .getLeave(MainClass.java:34)
    at com.baeldung.chainedexception.exceptions.MainClass
      .main(MainClass.java:29)
Exception in thread "main" com.baeldung.chainedexception.exceptions.
  NoLeaveGrantedException: Leave not sanctioned.
    at com.baeldung.chainedexception.exceptions.MainClass
      .getLeave(MainClass.java:37)
    at com.baeldung.chainedexception.exceptions.MainClass
      .main(MainClass.java:29)
```

### 5.2。带链条

接下来，让我们编写一个链接自定义异常的示例:

```java
public class MainClass {
    public void main(String[] args) throws Exception {
        getLeave();
    }

    public getLeave() throws NoLeaveGrantedException {
        try {
            howIsTeamLead();
        } catch (TeamLeadUpsetException e) {
             throw new NoLeaveGrantedException("Leave not sanctioned.", e);
        }
    }

    public void howIsTeamLead() throws TeamLeadUpsetException {
        throw new TeamLeadUpsetException("Team lead Upset.");
    }
}
```

最后，让我们来看看通过链式异常获得的日志:

```java
Exception in thread "main" com.baeldung.chainedexception.exceptions
  .NoLeaveGrantedException: Leave not sanctioned. 
    at com.baeldung.chainedexception.exceptions.MainClass
      .getLeave(MainClass.java:36) 
    at com.baeldung.chainedexception.exceptions.MainClass
      .main(MainClass.java:29) 
Caused by: com.baeldung.chainedexception.exceptions
  .TeamLeadUpsetException: Team lead Upset.
    at com.baeldung.chainedexception.exceptions.MainClass
  .howIsTeamLead(MainClass.java:44) 
    at com.baeldung.chainedexception.exceptions.MainClass
  .getLeave(MainClass.java:34) 
    ... 1 more
```

我们可以很容易地比较显示的日志，并得出结论，链式异常导致更干净的日志。

# 6。结论

在本文中，我们了解了链式异常的概念。

所有示例的实现都可以在 Github 项目中找到——这是一个基于 Maven 的项目，因此应该很容易导入和运行。