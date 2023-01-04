# Java 11 单文件源代码

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-single-file-source-code>

## 1。简介

[JDK 11](https://web.archive.org/web/20220625073334/https://openjdk.java.net/projects/jdk/11/) ，这是 Java SE 11 的实现，2018 年 9 月发布。

在本教程中，我们将介绍启动单文件源代码程序的 Java 11 新特性。

## 2。Java 11 之前

单文件程序是指程序适合放在单个源文件中的程序。

在 Java 11 之前，即使对于单文件程序，我们也必须遵循两步过程来运行程序。

例如，如果一个名为`HelloWorld.java`的文件包含一个名为`HelloWorld`的带有`main()`方法的类，**我们必须首先编译它:**

```
$ javac HelloWorld.java
```

这将生成一个类文件，我们必须使用命令运行该文件

```
$ java HelloWorld
Hello Java 11!
```

注意，因为我们已经通过编译创建了`.class`文件，所以`java`命令运行它。作为证明，我们可以改变我们在原始文件中打印的内容，但是如果我们下次不编译它，再次运行相同的`java`命令仍然会打印“Hello world”。

这类程序在学习 Java 的初级阶段或编写小型实用程序时是标准的。在这种情况下，在运行程序之前必须编译它是有点礼仪性的。

但是，一步到位的流程不是很好吗？Java 11 试图解决这个问题，它允许我们直接从源代码运行这样的程序。

## 3。启动单文件源代码程序

首先，让我们指出，在 Java 11 中，我们仍然可以像以前使用早期 Java 版本一样，编译和运行我们的 Java 程序。

此外，从 Java 11 开始，我们可以使用下面的命令来执行一个单文件程序:

```
$ java HelloWorld.java
Hello Java 11!
```

**注意我们是如何将 Java 源代码文件名而不是 Java 类传递给`java`命令的。**

**JVM 将源文件编译到内存中，然后运行它找到的第一个公共`main()`方法。**

如果源文件包含错误，我们会得到编译错误，否则，它会像已经编译过一样运行。

让我们也注意一下**这个命令在文件名和类名兼容性方面更加宽松**。

例如，如果我们重命名文件`WrongName.java`而不改变其内容，我们可以运行它:

```
java WrongName.java
```

这将起作用，并将预期的结果打印到控制台。但是，如果我们试图用' javac '命令编译 WrongName.java，我们会得到一个错误消息，因为文件中定义的类名与文件名不一致。

也就是说，不遵循几乎通用的命名约定仍然是不鼓励的。相应地重命名我们的文件或类应该是正确的做法。

## 4。命令行选项

Java launcher 引入了一个新的`source-file mode`来支持这个特性。如果满足以下两个条件之一，则启用源文件模式:

1.  命令行上的第一项，后跟 JVM 选项，是一个扩展名为`.java`的文件名
2.  命令行包含`–source`版本选项

**如果文件不遵循 Java 源文件的标准命名约定，我们需要使用`–source`选项。**我们将在下一节中更多地讨论这样的文件。

**在原始命令行中，任何放在源文件名**后面的参数都会在编译后的类执行时传递给它。

例如，我们有一个名为`Addition.java`的文件，其中包含一个`Addition`类。这个类包含一个计算其参数总和的`main()`方法:

```
$ java Addition.java 1 2 3
```

此外，我们可以在文件名前传递类似于`–class-path`的选项:

```
$ java --class-path=/some-path Addition.java 1 2 3
```

现在，**如果应用程序类路径中有一个类与我们正在执行的类**同名，我们将得到一个错误。

例如，假设在开发过程中的某个时刻，我们使用`javac`编译了当前工作目录中的文件:

```
$ javac HelloWorld.java
```

我们现在在当前工作目录中有两个`HelloWorld.java and HelloWorld.class`:

```
$ ls
HelloWorld.class  HelloWorld.java
```

但是，如果我们尝试使用源文件模式，我们会得到一个错误:

```
$ java HelloWorld.java                                            
error: class found on application class path: HelloWorld
```

## 5。社邦档案

在 Unix 派生的系统中，如 macOS 和 Linux，使用“#”是很常见的指令来运行可执行脚本文件。

例如，shell 脚本通常以下列开头:

```
#!/bin/sh
```

然后，我们可以执行脚本:

```
$ ./some_script
```

这种档案被称为“社邦档案”。

我们现在可以使用同样的机制来执行 Java 单文件程序。

如果我们将以下内容添加到文件的开头:

```
#!/path/to/java --source version
```

例如，让我们在名为`add`的文件中添加以下代码:

```
#!/usr/local/bin/java --source 11

import java.util.Arrays;

public class Addition
{
    public static void main(String[] args) {
        Integer sum = Arrays.stream(args)
          .mapToInt(Integer::parseInt)
          .sum();

        System.out.println(sum);
    }
}
```

并将文件标记为可执行文件:

```
$ chmod +x add
```

然后，我们可以像执行脚本一样执行该文件:

```
$ ./add 1 2 3
6
```

我们还可以显式地使用启动器来调用 shebang 文件:

```
$ java --source 11 add 1 2 3
6
```

**`–source`选项是必需的，即使它已经存在于文件中。**文件中的 shebang 被忽略，被视为不带`.java`扩展名的普通 java 文件。

然而**，我们不能将`.java`文件视为 shebang 文件，即使它包含有效的 shebang。**因此下面将导致一个错误:

```
$ ./Addition.java
./Addition.java:1: error: illegal character: '#'
#!/usr/local/bin/java --source 11
^
```

关于 shebang 文件需要注意的最后一点是，该指令使得文件依赖于平台。该文件无法在 Windows 等平台上使用，因为 Windows 本身不支持它。

## 6。结论

在本文中，我们看到了 Java 11 中引入的新的单文件源代码特性。

像往常一样，代码片段可以在 GitHub 上找到[。](https://web.archive.org/web/20220625073334/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-11)**