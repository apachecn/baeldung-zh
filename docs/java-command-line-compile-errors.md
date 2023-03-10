# Java 中常见的命令行编译错误

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-command-line-compile-errors>

## 1.概观

在命令行上编译 Java 程序时，预期命令行选项或参数的任何不匹配都会导致错误。

在本教程中，我们将首先调查`“Class Names Are Only Accepted if Annotation Processing Is Explicitly Requested”`错误。然后，我们将看看其他一些常见的编译错误。

## 2.错误示例

假设我们有下面的类`DemoClass`:

```java
package com.baeldung;

public class DemoClass {
    // fields and methods
}
```

现在，让我们尝试使用`[javac](/web/20220524050953/https://www.baeldung.com/javac)`命令编译`DemoClass`:

```java
javac DemoClass
```

上面的命令会给出一个错误:

```java
error: Class names, 'DemoClass', are only accepted if annotation processing is explicitly requested
1 error
```

该错误似乎与注释处理有关，并且有点隐晦，因为`DemoClass`没有与[注释处理](/web/20220524050953/https://www.baeldung.com/java-annotation-processing-builder)相关的代码。这个错误的实际原因是 **`DemoClass` 不是注释处理源文件**。

注释处理源文件是一种在编译时生成额外源代码的**便利技术**。与标准 Java 源文件不同，编译这些源文件时，不需要提供文件扩展名`.java`

## 3.解决问题

让我们用正确的文件扩展名`.java`再次编译`DemoClass`:

```java
javac DemoClass.java
```

不出所料，我们会把源文件编译成`DemoClass.class`文件。

## 4。其他提示和技巧

虽然当我们知道正确的编译方法时，这是一个简单的解决方法，但在编译或运行应用程序时，我们仍可能会遇到类似的困难。

### 4.1.使用不正确的文件扩展名

现在让我们试着用下面的命令来编译源文件，这个命令有一个错别字——"`.JAVA”`全是大写的:

```java
javac DemoClass.JAVA
```

这样做会产生与上面看到的相同的错误消息:

```java
error: Class names, 'DemoClass.JAVA', are only accepted if annotation processing is explicitly requested
1 error
```

### 4.2.主类错误

假设我们有一个包含`main`方法的`DemoApplication`类:

```java
public class DemoApplication {

    public static void main(String[] args) {
        System.out.println("This is a DemoApplication");
    }
}
```

现在让我们使用`java`命令来执行应用程序:

```java
java DemoApplication.class
```

结果是一个`ClassNotFoundException`:

```java
Error: Could not find or load main class DemoApplication.Class
Caused by: java.lang.ClassNotFoundException: DemoApplication.Class
```

现在，让我们尝试运行没有任何文件扩展名的应用程序——甚至没有`.class`或`.java`:

```java
java DemoApplication 
```

我们应该在控制台上看到输出:

```java
This is a DemoApplication
```

## 5.结论

在本文中，我们已经了解了在从命令行编译类时，`.java`文件扩展名的不正确使用或省略是如何导致错误的。此外，我们还看到了一些其他错误，这些错误与编译和运行独立应用程序时命令行参数的不正确使用有关。