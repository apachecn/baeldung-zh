# Java“找不到或加载主类”错误

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-could-not-find-load-main-class>

## 1.概观

偶尔，当我们运行 Java 程序时，我们可能会看到“找不到或加载主类”原因很容易猜到:JVM 找不到主类，给出了这个错误。但是为什么不能呢？

## 延伸阅读:

## [如何修复 Java . lang . unsupportedclassversionerror](/web/20220803080356/https://www.baeldung.com/java-lang-unsupportedclassversion)

Learn what causes the "java.lang.UnsupportedClassVersionError: Unsupported major.minor version error" message, and how to fix it.[Read more](/web/20220803080356/https://www.baeldung.com/java-lang-unsupportedclassversion) →

## [Java main()方法讲解](/web/20220803080356/https://www.baeldung.com/java-main-method)

Learn about the standard Java main() method along with some uncommon, but still supported, ways of writing it.[Read more](/web/20220803080356/https://www.baeldung.com/java-main-method) →

在本教程中，我们将讨论找不到主类的可能原因。我们还将看到如何修复它们。

## 2.抽样程序

我们将从一个`HelloWorld`程序开始:

```java
public class HelloWorld {
    public static void main(String[] args) {
        System.out.println("Hello world..!!!");
    }
}
```

现在让我们编译它:

```java
$ javac HelloWorld.java
```

这里，编译器将为我们的程序生成一个`.class`文件。这个`.class`文件将在同一个目录中生成。**`.class`文件将与 Java 程序中给出的类名同名。**这个`.class`文件是可执行的。

在接下来的部分中，我们将运行这个`.class`文件，并尝试理解错误“无法找到或加载主类”的可能原因

## 3.错误的类名

要运行 Java 编译器生成的`.class`文件，我们可以使用这个命令:

```java
java <.class filename>
```

现在让我们运行我们的程序:

```java
$ java helloworld
Error: Could not find or load main class helloworld
```

它失败了，并显示错误“无法找到或加载主类 helloworld”

如前所述，**编译器将生成`.class`文件，其名称与程序中 Java 类的名称完全相同。**所以在我们的例子中，主类的名字是`HelloWorld`，而不是`helloworld`。

让我们用正确的大小写再试一次:

```java
$ java HelloWorld
Hello world..!!!
```

这一次它运行成功。

### 3.1.文件扩展名

要编译一个 Java 程序，我们必须提供带有扩展名(.`java`):

```java
$ javac HelloWorld.java
```

但是要运行一个.`class`文件，我们需要提供类名，而不是文件名。所以不需要提供`.class`扩展:

```java
$ java HelloWorld.class
Error: Could not find or load main class HelloWorld.class
```

同样，让我们使用正确的类名运行我们的程序:

```java
$ java HelloWorld 
Hello world..!!!
```

## 4.Java 包名

在 Java 中，我们将相似的类放在一起，我们称之为`package`。

让我们将`HelloWorld`类移到`com.baeldung`包中:

```java
package com.baeldung;

public class HelloWorld {
    public static void main(String[] args) {
        System.out.println("Hello world..!!!");
    }
}
```

现在让我们像以前一样编译并运行更新后的`HelloWorld`程序:

```java
$ java HelloWorld
Error: Could not find or load main class HelloWorld
```

但是，我们再次得到错误“无法找到或加载主类 HelloWorld。”

让我们试着理解我们在这里错过了什么。

**要运行包中的 Java 类，我们必须提供它的完全限定名。**所以在我们这里，`HelloWorld`的全限定名是`com.baeldung.HelloWorld`。

现在，当我们创建`com.baeldung`包时，我们实际上创建了这个文件夹结构:

```java
com/baeldung/HelloWorld.java
```

首先，让我们试着从`com/baeldung`目录运行我们的程序:

```java
$ java com.baeldung.HelloWorld
Error: Could not find or load main class com.baeldung.HelloWorld
```

尽管如此，我们还是不能运行我们的程序。

这里，当我们指定完全限定类名`com.baeldung.HelloWorld`时，Java 试图在运行程序的目录下的`com/baeldung`中找到 HelloWorld.class 文件。

由于我们已经在`com/baeldung`中，Java 无法找到并运行`HelloWorld`程序。

现在让我们回到父文件夹并运行它:

```java
$ java com.baeldung.HelloWorld
Hello world..!!!
```

我们又可以向世界问好了。

## 5.无效的类路径

在继续之前，让我们首先了解什么是类路径。它是我们当前运行的 JVM 可用的一组类。

我们使用 classpath 变量来告诉 JVM 在文件系统的哪里可以找到`.class`文件。

运行程序时，我们可以使用`-classpath`选项提供类路径:

```java
java -classpath /my_programs/compiled_classes HelloWorld
```

在这里，Java 将在`/my_programs/compiled_classes`文件夹中查找`HelloWorld.class`文件，这个文件夹的名称是我们刚刚虚构的。默认情况下，**class path 变量被设置为"."，表示当前目录。**

在上面的部分中，我们更改了目录来运行我们的程序。但是如果我们想从其他文件夹运行它呢？这就是类路径变量帮助我们的时候了。

为了从目录`com/baeldung`运行我们的程序，我们可以简单地声明我们的类路径在两个目录之上——每个包部分一个目录:

```java
$ java -claspath ../../ com.baeldung.HelloWorld
Hello world..!!!
```

这里，“。”表示父目录。在我们的情况下"../../”代表我们的包层次结构的顶端。

## 6.结论

在本文中，我们了解了错误“无法找到或加载主类”的可能原因

然后，当然，我们也学会了如何解决这个错误。