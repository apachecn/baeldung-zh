# Gradle:源兼容性与目标兼容性

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/gradle-sourcecompatiblity-vs-targetcompatibility>

## 1.概观

在本文中，我们将看看**`sourceCompatbility`和`targetCompatibility` Java 配置**的区别以及它们在 Gradle 中的用法。

你可以查看我们的[Gradle](/web/20220524070609/https://www.baeldung.com/gradle)介绍文章，了解更多基础知识。

## 2.在 Java 中处理版本

当我们使用`javac`编译一个 Java 程序时，我们可以为版本处理提供编译选项。有两个选项可用:

*   `-source`使用与 Java 版本相匹配的值，**直到我们用于编译的 JDK**(例如，JDK8 的 1.8)。我们提供的版本值**将把我们可以在源代码中使用的语言特性**限制在各自的 Java 版本中。
*   `-target`类似，但控制生成的类文件的版本。这意味着我们提供的版本值将是我们的程序可以在上运行的最低 Java 版本。

例如:

```java
javac HelloWorld.java -source 1.6 -target 1.8
```

这样会生成一个**需要 Java 8 或以上版本才能运行**的类文件。此外，源代码**不能包含 lambda 表达式或任何 Java 6** 中没有的特性。

## 3.用 Gradle 处理版本

Gradle 和 Java 插件让我们用`java`任务的`sourceCompatibility`和`targetCompatibility`配置来设置`source `和`target`选项。类似地，**我们使用与** `**javac**.`相同的值

让我们设置`build.gradle`文件:

```java
plugins {
    id 'java'
}

group 'com.baeldung'

java {
    sourceCompatibility = "1.6"
    targetCompatibility = "1.8"
} 
```

## 4.`HelloWorldApp`示例编译

我们可以创造一个你好的世界！控制台应用程序，并通过使用上面的脚本构建它来演示功能。

让我们创建一个非常简单的类:

```java
public class HelloWorldApp {
    public static void main(String[] args) {
        System.out.println("Hello World!");
    }
} 
```

当我们使用`gradle build`命令构建它时，Gradle 将生成一个名为`HelloWorldApp.class`的类文件。

我们可以使用与 Java 打包在一起的`javap `命令行工具来检查这个类文件生成的字节码版本:

```java
javap -verbose HelloWorldApp.class
```

这打印了很多信息，但是在前几行中，我们可以看到:

```java
public class com.baeldung.helloworld.HelloWorldApp
  minor version: 0
  major version: 52
  flags: ACC_PUBLIC, ACC_SUPER
```

`major version`字段的值为 52，这是 Java 8 类文件的版本号。这意味着我们的`HelloWorldApp.class ` **只能使用 Java 8 及以上的**运行。

为了测试`sourceCompatibility`配置，我们可以修改源代码并引入一个 Java 6 中没有的特性。

让我们用一个λ表达式:

```java
public class HelloWorldApp {

    public static void main(String[] args) {
        Runnable helloLambda = () -> {
            System.out.println("Hello World!");
        }
        helloLambda.run();
    }

}
```

如果我们试图用 Gradle 构建我们的代码，我们会看到一个编译错误:

```java
error: lambda expressions are not supported in -source 1.6
```

`-source`选项是`sourceCompatibility` Gradle 配置的 Java 等价物，它阻止我们的代码编译。基本上，如果我们不想引入更高版本的特性，它**会防止我们错误地使用它们**——例如，我们可能希望我们的应用程序也能够在 Java 6 运行时上运行。

## 5.结论

在本文中，我们解释了如何使用`-source` 和`-target`编译选项来处理 Java 源代码和目标运行时的版本。此外，我们了解了这些选项如何映射到 Gradle 的`sourceCompatbility`和`targetCompatibility`配置以及 Java 插件，并在实践中演示了它们的功能。

和往常一样，这篇文章的源代码可以在 GitHub 的[上找到。](https://web.archive.org/web/20220524070609/https://github.com/eugenp/tutorials/tree/master/gradle/gradle-source-vs-target-compatibility)