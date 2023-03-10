# Java“Hello World”示例

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-hello-world>

## 1.概观

Java 是一种通用编程语言，它专注于 WORA(一次编写，随处运行)原则。

它运行在 JVM ( [Java 虚拟机](/web/20221126220351/https://www.baeldung.com/jvm-vs-jre-vs-jdk))上，JVM 负责抽象底层操作系统，允许 Java 程序在几乎任何地方运行，从应用服务器到手机。

学习一门新语言时，“Hello World”往往是我们写的第一个程序。

在本教程中，我们将**学习一些基本的 Java 语法，并编写一个简单的“Hello World”程序**。

## 2.编写 Hello World 程序

让我们打开任何 IDE 或文本编辑器，创建一个名为`HelloWorld.java`的简单文件:

```java
public class HelloWorld {
    public static void main(String[] args) {
        System.out.println("Hello World!");
    }
}
```

在我们的例子中，**我们已经创建了一个名为`HelloWorld`的 Java `class` ，它包含一个将一些文本写入控制台的`main`方法**。

当我们执行程序时，Java 将运行`main`方法，打印出“Hello World！”在[控制台的](/web/20221126220351/https://www.baeldung.com/java-console-input-output)上。

现在，让我们看看如何编译和执行我们的程序。

## 3.编译和执行程序

为了编译一个 Java 程序，我们需要从命令行调用 [Java 编译器](/web/20221126220351/https://www.baeldung.com/javac):

```java
$ javac HelloWorld.java
```

编译器生成`HelloWorld.class`文件，这是我们代码的编译后的字节码版本。

让我们通过调用来运行它:

```java
$ java HelloWorld
```

看看结果:

```java
Hello World!
```

## 4.结论

在这个简单的例子中，我们用默认的`main`方法创建了一个 Java 类，在[系统控制台](/web/20221126220351/https://www.baeldung.com/java-lang-system)上打印出一个字符串。

我们看到了如何创建、编译和执行 Java 程序，并熟悉了一些基本语法。我们在这里看到的 Java 代码和命令在每个支持 Java 的操作系统上都是一样的。