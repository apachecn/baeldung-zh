# 在 Java 中创建自定义异常

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-new-custom-exception>

## 1。概述

在本教程中，我们将讲述如何在 Java 中创建一个自定义异常。

我们将展示用户定义的异常是如何实现的，以及如何用于检查和未检查的异常。

## 延伸阅读:

## [Java 中的异常处理](/web/20221101183545/https://www.baeldung.com/java-exceptions)

Learn the basics of exception handling in Java as well as some best and worst practices.[Read more](/web/20221101183545/https://www.baeldung.com/java-exceptions) →

## [Java 中已检查和未检查的异常](/web/20221101183545/https://www.baeldung.com/java-checked-unchecked-exceptions)

Learn the differences between Java's checked and unchecked exception with some examples[Read more](/web/20221101183545/https://www.baeldung.com/java-checked-unchecked-exceptions) →

## [常见的 Java 异常](/web/20221101183545/https://www.baeldung.com/java-common-exceptions)

A quick overview to the common Java Exceptions.[Read more](/web/20221101183545/https://www.baeldung.com/java-common-exceptions) →

## 2。需要自定义例外

Java 异常几乎涵盖了编程中必然会发生的所有一般异常。

然而，我们有时需要用自己的标准异常来补充这些标准异常。

这些是引入自定义异常的主要原因:

*   业务逻辑异常–特定于业务逻辑和工作流的异常。这些有助于应用程序用户或开发人员理解确切的问题是什么。
*   捕捉现有 Java 异常的子集并提供特定的处理

Java 异常可以检查和不检查。在下一节中，我们将讨论这两种情况。

## 3。自定义检查异常

检查异常是需要显式处理的异常。

让我们考虑一段返回文件第一行的代码:

```java
try (Scanner file = new Scanner(new File(fileName))) {
    if (file.hasNextLine()) return file.nextLine();
} catch(FileNotFoundException e) {
    // Logging, etc 
} 
```

上面的代码是处理 Java 检查异常的经典方式。虽然代码抛出了`FileNotFoundException`，但不清楚确切的原因是什么——是文件不存在还是文件名无效。

**要创建自定义异常，我们必须扩展`java.lang.Exception`类。**

让我们通过创建一个名为`IncorrectFileNameException`的自定义检查异常来看看这个例子:

```java
public class IncorrectFileNameException extends Exception { 
    public IncorrectFileNameException(String errorMessage) {
        super(errorMessage);
    }
} 
```

注意，我们还必须提供一个构造函数，它将一个`String`作为错误消息，并调用父类构造函数。

这就是我们定义自定义异常所需要做的一切。

接下来，让我们看看如何在示例中使用自定义异常:

```java
try (Scanner file = new Scanner(new File(fileName))) {
    if (file.hasNextLine())
        return file.nextLine();
} catch (FileNotFoundException e) {
    if (!isCorrectFileName(fileName)) {
        throw new IncorrectFileNameException("Incorrect filename : " + fileName );
    }
    //...
} 
```

我们已经创建并使用了一个自定义异常，所以用户现在可以知道确切的异常是什么。

这样够了吗？因此，我们失去了异常的根本原因。

为了解决这个问题，我们还可以向构造函数添加一个`java.lang.Throwable`参数。这样，我们可以将根异常传递给方法调用:

```java
public IncorrectFileNameException(String errorMessage, Throwable err) {
    super(errorMessage, err);
} 
```

现在`IncorrectFileNameException` 与异常的根本原因一起使用:

```java
try (Scanner file = new Scanner(new File(fileName))) {
    if (file.hasNextLine()) {
        return file.nextLine();
    }
} catch (FileNotFoundException err) {
    if (!isCorrectFileName(fileName)) {
        throw new IncorrectFileNameException(
          "Incorrect filename : " + fileName , err);
    }
    // ...
} 
```

这就是我们如何使用定制异常**而不丢失它们发生的根本原因。**

## 4。自定义未检查异常

在同一个例子中，假设如果文件名不包含任何扩展名，我们需要一个自定义异常。

在这种情况下，我们需要一个类似于上一个的自定义未检查异常，因为这个错误只会在运行时被检测到。

**要创建一个定制的未检查异常，我们需要扩展`java.lang.RuntimeException`类**:

```java
public class IncorrectFileExtensionException 
  extends RuntimeException {
    public IncorrectFileExtensionException(String errorMessage, Throwable err) {
        super(errorMessage, err);
    }
} 
```

这样，我们可以在示例中使用这个自定义的未检查异常:

```java
try (Scanner file = new Scanner(new File(fileName))) {
    if (file.hasNextLine()) {
        return file.nextLine();
    } else {
        throw new IllegalArgumentException("Non readable file");
    }
} catch (FileNotFoundException err) {
    if (!isCorrectFileName(fileName)) {
        throw new IncorrectFileNameException(
          "Incorrect filename : " + fileName , err);
    }

    //...
} catch(IllegalArgumentException err) {
    if(!containsExtension(fileName)) {
        throw new IncorrectFileExtensionException(
          "Filename does not contain extension : " + fileName, err);
    }

    //...
} 
```

## 5。结论

当我们需要处理与业务逻辑相关的特定异常时，自定义异常非常有用。如果使用得当，它们可以作为更好的异常处理和日志记录的实用工具。

本文中使用的示例代码可以从 GitHub 上的[处获得。](https://web.archive.org/web/20221101183545/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-exceptions)