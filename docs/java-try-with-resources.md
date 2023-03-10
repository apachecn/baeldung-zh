# Java——尝试使用资源

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-try-with-resources>

## 1。概述

Java 7 中引入的对`try-with-resources`的支持允许我们声明要在`try`块中使用的资源，并保证资源将在该块执行后关闭。

声明的资源需要实现`AutoCloseable`接口。

## 延伸阅读:

## 捕捉可投掷物体是一种不好的做法吗？

Find out if it is a bad practice to catch Throwable.[Read more](/web/20220829103433/https://www.baeldung.com/java-catch-throwable-bad-practice) →

## [Java 全局异常处理程序](/web/20220829103433/https://www.baeldung.com/java-global-exception-handler)

Learn how to globally handle all uncaught exceptions in your Java application[Read more](/web/20220829103433/https://www.baeldung.com/java-global-exception-handler) →

## [Java 中已检查和未检查的异常](/web/20220829103433/https://www.baeldung.com/java-checked-unchecked-exceptions)

Learn the differences between Java's checked and unchecked exception with some examples[Read more](/web/20220829103433/https://www.baeldung.com/java-checked-unchecked-exceptions) →

## 2。使用`try-with-resources`

简单地说，要自动关闭，资源必须在`try`中声明和初始化:

```java
try (PrintWriter writer = new PrintWriter(new File("test.txt"))) {
    writer.println("Hello World");
} 
```

## 3。将`try`–`catch-finally`替换为`try-with-resources`

使用新的`try-with-resources`功能的简单而明显的方法是替换传统而冗长的`try-catch-finally`块。

让我们比较下面的代码示例。

第一个是典型的`try-catch-finally` 块:

```java
Scanner scanner = null;
try {
    scanner = new Scanner(new File("test.txt"));
    while (scanner.hasNext()) {
        System.out.println(scanner.nextLine());
    }
} catch (FileNotFoundException e) {
    e.printStackTrace();
} finally {
    if (scanner != null) {
        scanner.close();
    }
}
```

这里是使用`try-with-resources`的新的超级简洁的解决方案:

```java
try (Scanner scanner = new Scanner(new File("test.txt"))) {
    while (scanner.hasNext()) {
        System.out.println(scanner.nextLine());
    }
} catch (FileNotFoundException fnfe) {
    fnfe.printStackTrace();
}
```

这里是进一步探索[`Scanner`级](/web/20220829103433/https://www.baeldung.com/java-scanner)的地方。

## 4。`try-with-resources`拥有多种资源

我们可以在一个`try-with-resources`块中声明多个资源，用分号分隔它们:

```java
try (Scanner scanner = new Scanner(new File("testRead.txt"));
    PrintWriter writer = new PrintWriter(new File("testWrite.txt"))) {
    while (scanner.hasNext()) {
	writer.print(scanner.nextLine());
    }
}
```

## 5。带有的自定义资源`**AutoCloseable**`

为了构建一个将由`try-with-resources`块正确处理的自定义资源，该类应该实现`Closeable`或`AutoCloseable`接口并覆盖`close`方法:

```java
public class MyResource implements AutoCloseable {
    @Override
    public void close() throws Exception {
        System.out.println("Closed MyResource");
    }
}
```

## 6。资源关闭订单

首先定义/获取的资源将最后关闭。让我们来看一个这种行为的例子:

**资源 1:**

```java
public class AutoCloseableResourcesFirst implements AutoCloseable {

    public AutoCloseableResourcesFirst() {
        System.out.println("Constructor -> AutoCloseableResources_First");
    }

    public void doSomething() {
        System.out.println("Something -> AutoCloseableResources_First");
    }

    @Override
    public void close() throws Exception {
        System.out.println("Closed AutoCloseableResources_First");
    }
} 
```

**资源 2:**

```java
public class AutoCloseableResourcesSecond implements AutoCloseable {

    public AutoCloseableResourcesSecond() {
        System.out.println("Constructor -> AutoCloseableResources_Second");
    }

    public void doSomething() {
        System.out.println("Something -> AutoCloseableResources_Second");
    }

    @Override
    public void close() throws Exception {
        System.out.println("Closed AutoCloseableResources_Second");
    }
}
```

**代码:**

```java
private void orderOfClosingResources() throws Exception {
    try (AutoCloseableResourcesFirst af = new AutoCloseableResourcesFirst();
        AutoCloseableResourcesSecond as = new AutoCloseableResourcesSecond()) {

        af.doSomething();
        as.doSomething();
    }
} 
```

**输出:**

`Constructor -> AutoCloseableResources_First`
`Constructor -> AutoCloseableResources_Second`
`Something -> AutoCloseableResources_First`
`Something -> AutoCloseableResources_Second`
`Closed AutoCloseableResources_Second`

## 7。`catch`和`finally`

一个`try-with-resources`块**仍然可以有`catch`和`finally`块**，它们将以与传统`try`块相同的方式工作。

## 8.Java 9–有效的最终变量

在 Java 9 之前，我们只能在`try-with-resources `块中使用新变量:

```java
try (Scanner scanner = new Scanner(new File("testRead.txt")); 
    PrintWriter writer = new PrintWriter(new File("testWrite.txt"))) { 
    // omitted
}
```

如上所示，当声明多个资源时，这尤其冗长。从 Java 9 开始，作为 [JEP 213](https://web.archive.org/web/20220829103433/https://openjdk.java.net/jeps/213) 、**的一部分，我们现在可以在`try-with-resources` 块**中使用 [`final `甚至有效的 final](/web/20220829103433/https://www.baeldung.com/java-effectively-final) 变量:

```java
final Scanner scanner = new Scanner(new File("testRead.txt"));
PrintWriter writer = new PrintWriter(new File("testWrite.txt"))
try (scanner;writer) { 
    // omitted
}
```

简单地说，如果一个变量在第一次赋值后没有改变，那么它实际上就是最终变量，即使它没有被显式地标记为`final`。

如上所示，`scanner `变量被显式声明为`final `，所以我们可以在`try-with-resources `块中使用它。虽然`writer `变量没有显式地`final, `，但它在第一次赋值后不会改变。所以，我们也可以使用`writer `变量。

## 9。结论

在本文中，我们讨论了如何使用 try-with-resources 以及如何用 try-with-resources 替换`try`、`catch`和`finally`。

我们还看了用`AutoCloseable`构建定制资源以及关闭资源的顺序。

在[GitHub 项目](https://web.archive.org/web/20220829103433/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-exceptions-2)中可以获得该示例的完整**源代码**。