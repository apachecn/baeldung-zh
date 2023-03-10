# Java 源和目标选项指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-source-target-options>

## 1。概述

在本教程中，我们将探索 Java 提供的–`source`和–`target`选项。此外，我们将了解这些选项在 Java 8 中是如何工作的，以及它们是如何从 Java 9 发展而来的。

## 2。向后兼容旧的 Java 版本

由于 Java 发布和更新频繁，应用程序可能无法每次都迁移到较新的版本。**有时应用程序需要确保它们的代码向后兼容旧版本的 Java。在`javac`中的`target`和`source`选项可以很容易地实现这一点。**

为了详细理解这一点，首先，让我们创建一个示例类，并使用 Java 9 中添加的、Java 8 中没有的`List.of()`方法:

```java
public class TestForSourceAndTarget {
    public static void main(String[] args) {
        System.out.println(List.of("Hello", "Baeldung"));
    }
}
```

让我们假设我们使用 Java 9 来编译代码，并希望与 Java 8 兼容。
我们可以使用`-source`和`-target`来实现这一点:

```java
/jdk9path/bin/javac TestForSourceAndTarget.java -source 8 -target 8
```

现在，我们得到一个关于编译的警告，但是编译是成功的:

```java
warning: [options] bootstrap class path not set in conjunction with -source 8
1 warning
```

让我们用 Java 8 运行代码，我们可以看到错误:

```java
$ /jdk8path/bin/java TestForSourceAndTarget
Exception in thread "main" java.lang.NoSuchMethodError: ↩
  java.util.List.of(Ljava/lang/Object;Ljava/lang/Object;)Ljava/util/List;
  at com.baeldung.TestForSourceAndTarget.main(TestForSourceAndTarget.java:7)
```

在 Java 8 中，`List.of()`是不存在的。理想情况下，Java 应该在编译时抛出这个错误。然而，在编译期间，我们只得到一个警告。

让我们来看看在编译过程中得到的警告。`javac`通知我们引导类不与–`source`8 结合使用。结果是，**我们必须提供引导类文件路径，以便`javac`可以选择正确的文件进行交叉编译。**在我们的例子中，我们希望兼容 Java 8，但是默认选择了 Java 9 引导类。

为此，**我们必须使用–`Xbootclasspath`指向需要交叉编译的 Java 版本的路径**:

```java
/jdk9path/bin/javac TestForSourceAndTarget.java -source 8 -target 8 -Xbootclasspath ${jdk8path}/jre/lib/rt.jar
```

现在，让我们编译它，我们可以看到编译时的错误:

```java
TestForSourceAndTarget.java:7: error: cannot find symbol
        System.out.println(List.of("Hello", "Baeldung"));
                               ^
  symbol:   method of(String, 
String)
  location: interface List
1 error
```

## 3。信号源选项

**–`source`选项指定编译器接受的 Java 源代码版本:**

```java
/jdk9path/bin/javac TestForSourceAndTarget.java -source 8 -target 8
```

如果没有`-source`选项，编译器将根据正在使用的 Java 版本编译源代码。

In our example, If `-source` 8 is not provided, the compiler will compile source code in accordance with Java 9 specifications.

`-source`值 8 也意味着我们不能使用任何特定于 Java 9 的 API。为了使用 Java 9 中引入的任何 API，比如 *List.of()* ，我们必须将 source 选项的值设置为 9。

## 4。目标选项

**目标选项指定要生成的类文件的 Java 版本。目标版本必须等于或高于来源选项:**

```java
/jdk9path/bin/javac TestForSourceAndTarget.java -source 8 -target 8
```

这里的**–`target`值为 8 意味着这将生成一个** **需要 Java 8 或更高版本才能运行**的类文件。如果我们在 Java 7 中运行上面的类文件，我们会得到一个错误。

## 5.Java 8 和更早版本中的源和目标

**从我们的例子中可以看出，要在 Java 8 之前正确地进行交叉编译，我们需要提供三个选项，即——`source, -target,`和`-Xbootclasspath.`** 例如，如果我们需要用 Java 9 构建代码，但它需要与 Java 8 兼容:

```java
/jdk9path/bin/javac TestForSourceAndTarget.java -source 8 -target 8 -Xbootclasspath ${jdk8path}/jre/lib/rt.jar
```

从 JDK 8 开始，不推荐使用 1.5 或更低版本的源或目标，而在 JDK 9 中，完全删除了对 1.5 或更低版本的源或目标的支持。

## 6.Java 9 和更高版本中的源和目标

尽管交叉编译在 Java 8 中运行良好，但三个命令行选项是必需的。当我们有三个选项时，很难让它们都保持最新。

**作为 Java 9 的一部分，[–`release`选项](/web/20220905180146/https://www.baeldung.com/java-compiler-release-option)被引入以简化交叉编译过程。**使用–`release`选项，我们可以完成与前面选项相同的交叉编译。

让我们使用`–release`选项来编译我们之前的示例类:

```java
/jdk9path/bin/javac TestForSourceAndTarget.java —release 8
```

```java
TestForSourceAndTarget.java:7: error: cannot find symbol
        System.out.println(List.of("Hello", "Baeldung"));
                               ^
  symbol:   method of(String,String)
  location: interface List
1 error
```

很明显，在编译期间只需要一个选项`-release`，该错误表明`javac`已经在内部为`-source, -target,`和`-Xbootclasspath.`分配了正确的值

## 7 .**。结论**

在本文中，我们了解了`javac`的–`source`和–`target`选项，以及它们与交叉编译的关系。此外，我们还发现了它们在 Java 8 及更高版本中的用法。此外，我们还学习了 Java 9 中引入的`-release`选项。