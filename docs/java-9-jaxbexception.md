# 在 Java 9 中处理 JAXBException 的 NoClassDefFoundError

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-9-jaxbexception>

## 1。简介

对于任何尝试升级到 Java 9 的人来说，在编译以前在 Java 早期版本中工作的代码时，他们可能经历过某种类型的`NoClassDefFoundError `。

在本文中，我们将看看一个常见的缺失类`JAXBException`，以及解决它的不同方法。这里提供的解决方案一般适用于升级到 Java 9 时可能缺失的任何类。

## 2。为什么 Java 9 找不到`JAXBException`？

Java 9 最受关注的特性之一是模块系统。Java 9 模块系统的目标是将核心 JVM 类和相关项目分解成独立的模块。这有助于我们通过只包含运行所需的最少类来创建占用空间更小的应用程序。

缺点是默认情况下，许多类在类路径中不再可用。在这种情况下，可以在一个名为`java.xml.bind`的新 Jakarta EE 模块中找到类`JAXBException`。因为核心 Java 运行时不需要这个模块，所以默认情况下它在类路径中不可用。

尝试运行使用`JAXBException`的应用程序将导致:

```java
NoClassDefFoundError: javax/xml/bind/JAXBException
```

为了绕过这个**，我们必须包括`java.xml.bind`模块**。正如我们将在下面看到的，有多种方法可以实现这一点。

## 3。短期解决方案

确保 JAXB API 类对应用程序可用的最快方法是使用`–add-modules `命令行参数添加**:**

```java
--add-modules java.xml.bind
```

然而，由于几个原因，这可能不是一个好的解决方案。

首先，`–add-modules `参数在 Java 9 中也是新的。对于需要在多个版本的 Java 上运行的应用程序来说，这带来了一些挑战。我们必须维护多组构建文件，应用程序运行的每个 Java 版本都有一组。

为了解决这个问题，我们也可以对旧的 Java 编译器使用`-XX:+IgnoreUnrecognizedVMOptions`命令行参数。

然而，这意味着任何打字错误或拼写错误的论点不会引起我们的注意。例如，如果我们试图设置一个最小或最大的堆大小，并且输入了错误的参数名，我们不会得到警告。我们的应用程序仍将启动，但它将以不同于我们预期的配置运行。

其次，`–add-modules `选项将在未来的 Java 版本中被弃用。这意味着在我们升级到新版本的 Java 后，我们将面临使用未知命令行参数的相同问题，并且必须再次解决这个问题。

## 4。长期解决方案

有一种更好的方法可以跨 Java 的不同版本工作，并且不会与未来的版本相冲突。

解决方案是**利用诸如 Maven** 这样的依赖管理工具。使用这种方法，我们可以像添加任何其他库一样，将 JAXB API 库作为一个依赖项来添加:

```java
<dependency>
    <groupId>javax.xml.bind</groupId>
    <artifactId>jaxb-api</artifactId>
    <version>2.3.0</version>
</dependency>
```

上面的库只包含 JAXB API 类，包括`JAXBException`。根据应用，我们可能需要包括其他模块。

还要记住， **Maven 工件名称可能不同于 Java 9 模块名称**，JAXB API 就是这种情况。可以在 [Maven Central](https://web.archive.org/web/20221206085047/https://search.maven.org/classic/#artifactdetails%7Cjavax.xml.bind%7Cjaxb-api-parent%7C2.3.0%7Cpom) 上找到。

## 5。结论

Java 9 模块系统提供了许多好处，比如减小应用程序的大小和更好的性能。

然而，它也引入了一些意想不到的后果。升级到 Java 9 时，了解应用程序真正需要哪些模块并采取措施确保它们在类路径中可用是很重要的。