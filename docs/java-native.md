# Java Native 关键字和方法

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-native>

## 1.概观

在这个快速教程中，我们将讨论 Java 中关键字`native`的概念，我们还将展示如何将`native`方法集成到 Java 代码中。

## 2.Java 中的关键字`native`

首先我们来讨论一下什么是 Java 中的一个 `native`关键字。

**简单来说，这是一个非访问修饰符，用于访问用 Java 以外的语言实现的方法，如 C/C++** 。

它表示一个方法或代码的平台相关实现，也作为 [JNI](/web/20221125202404/https://www.baeldung.com/jni) 和其他编程语言之间的接口。

## 3.`native`方法

**`native`方法是 Java 方法(实例方法或类方法)，其实现也是用另一种编程语言编写的，如 C/C++。**

此外，标记为`native`的方法不能有主体，应该以分号结束:

```java
[ public | protected | private] native [return_type] method ();
```

我们可以使用它们来:

*   用其他编程语言编写的系统调用或库实现一个接口
*   访问只能从其他语言访问的系统或硬件资源
*   将已经存在的用 C/C++编写的遗留代码集成到 Java 应用程序中
*   用 Java 中的任意代码调用编译后的动态加载库

## 4.例子

现在让我们演示如何将这些方法集成到我们的 Java 代码中。

### 4.1.在 Java 中访问本机代码

首先，让我们创建一个类`DateTimeUtils`，它需要访问一个名为`getSystemTime`的依赖于平台的`native`方法:

```java
public class DateTimeUtils {
    public native String getSystemTime();
    // ...
}
```

为了加载它，我们将使用`System.loadLibrary.`

让我们将加载这个库的调用放在一个`static `块中，以便它在我们的类中可用:

```java
public class DateTimeUtils {
    public native String getSystemTime();

    static {
        System.loadLibrary("nativedatetimeutils");
    }
}
```

我们已经创建了一个动态链接库`nativedatetimeutils`，它使用我们的[JNI 指南文章](/web/20221125202404/https://www.baeldung.com/jni)中的详细说明在 C++中实现了`getSystemTime`。

### 4.2.测试`native`方法

最后，让我们看看如何测试在`DateTimeUtils`类中定义的本地方法:

```java
public class DateTimeUtilsManualTest {

   @BeforeClass
    public static void setUpClass() {
        // .. load other dependent libraries  
        System.loadLibrary("nativedatetimeutils");
    }

    @Test
    public void givenNativeLibsLoaded_thenNativeMethodIsAccessible() {
        DateTimeUtils dateTimeUtils = new DateTimeUtils();
        LOG.info("System time is : " + dateTimeUtils.getSystemTime());
        assertNotNull(dateTimeUtils.getSystemTime());
    }
}
```

下面是记录器的输出:

```java
[main] INFO  c.b.n.DateTimeUtilsManualTest - System time is : Wed Dec 19 11:34:02 2018
```

正如我们所见，在关键字`native`的帮助下，我们能够成功地访问用另一种语言(在我们的例子中是 C++)编写的平台相关的实现。

## 5.结论

在这篇文章中，我们学习了`native`关键字和方法的基础知识。通过一个简单的例子，我们还学习了如何在 Java 中集成它们。

本文中使用的代码片段[可以从 Github](https://web.archive.org/web/20221125202404/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-lang-syntax-2) 上获得。