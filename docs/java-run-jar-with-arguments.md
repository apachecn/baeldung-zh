# 从命令行运行 Java 应用程序

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-run-jar-with-arguments>

## 1.概观

通常，每个有意义的应用程序都包含一个或多个 JAR 文件作为依赖项。但是有时 JAR 文件本身代表一个独立的应用程序或一个 web 应用程序。

这里我们将重点关注独立应用程序场景。从现在开始，我们将把它称为 JAR 应用程序。

在本教程中，我们将首先学习如何创建一个 JAR 应用程序。稍后，我们将学习如何使用或不使用命令行参数运行 JAR 应用程序。

## 2.创建一个 **JAR** 应用

一个 [JAR 文件](/web/20221103233850/https://www.baeldung.com/java-create-jar)可以包含一个或多个主类。**每个主类都是一个应用程序的入口点。**因此，一个 JAR 文件理论上可以包含多个应用程序，但是它必须包含至少一个主类才能运行。

一个 JAR 文件可以在它的[清单文件](/web/20221103233850/https://www.baeldung.com/java-jar-executable-manifest-main-class)中设置一个入口点。在这种情况下，JAR 文件是一个[可执行 JAR](/web/20221103233850/https://www.baeldung.com/executable-jar-with-maven) 。主类必须包含在 JAR 文件中。

首先，让我们看一个简单的例子，看看如何编译我们的类并创建一个带有清单文件的可执行 JAR:

```
$ javac com/baeldung/jarArguments/*.java
$ jar cfm JarExample.jar ../resources/example_manifest.txt com/baeldung/jarArguments/*.class
```

不可执行的 JAR 只是一个在清单文件中没有定义`Main-Class`的 JAR 文件。正如我们将在后面看到的，我们仍然可以运行包含在 JAR 文件本身中的主类。

下面是我们如何创建一个没有清单文件的不可执行 JAR:

```
$ jar cf JarExample2.jar com/baeldung/jarArguments/*.class
```

## 3.Java 命令行参数

就像任何应用程序一样，JAR 应用程序接受任意数量的参数，包括零个参数。这完全取决于应用程序的需求。

这允许用户在应用程序启动时**指定配置信息。**

因此，应用程序可以避免硬编码的值，并且仍然可以处理许多不同的用例。

参数可以包含任何字母数字字符、unicode 字符以及 shell 允许的一些特殊字符，例如@。

**参数由一个或多个空格分隔。如果一个参数需要包含空格，空格必须用引号括起来。单引号或双引号都可以。**

通常，对于典型的 Java 应用程序，当调用应用程序时，用户在类名后输入命令行参数。

然而，对于 JAR 应用程序来说，情况并非总是如此。

正如我们所讨论的，Java 主类的入口点是[主方法](/web/20221103233850/https://www.baeldung.com/java-main-method)。**参数都是`String`的**，并作为`String`数组传递给主方法。

也就是说，在应用程序内部，我们可以将`String`数组的任何元素转换为其他数据类型，例如`char`、`int`、`double`，它们的[、**包装类**、](/web/20221103233850/https://www.baeldung.com/java-wrapper-classes)或其他适当的类型。

## 4.运行带有参数的可执行文件 **JAR**

让我们看看运行带参数的可执行 JAR 文件的基本语法:

`**java -jar jar-file-name [args …]**`

前面创建的可执行 JAR 是一个简单的应用程序，它只是打印出传入的参数。我们可以用任意数量的参数来运行它。

下面是一个有两个参数的例子:

```
$ java -jar JarExample.jar "arg 1" [[email protected]](/web/20221103233850/https://www.baeldung.com/cdn-cgi/l/email-protection) 
```

我们将在控制台中看到以下输出:

```
Hello Baeldung Reader in JarExample!
There are 2 argument(s)!
Argument(1):arg 1
Argument(2):[[email protected]](/web/20221103233850/https://www.baeldung.com/cdn-cgi/l/email-protection) 
```

所以，**当调用一个可执行的 JAR 时，我们不需要在命令行上指定主类名。**我们只需在 JAR 文件名后添加我们的参数。如果我们在可执行 JAR 文件名后提供了一个类名，那么它将成为实际主类的第一个参数。

大多数情况下，JAR 应用程序是一个可执行的 JAR。一个可执行 JAR 最多可以有一个在[清单文件](/web/20221103233850/https://www.baeldung.com/java-jar-executable-manifest-main-class)中定义的主类。

因此，同一个可执行 JAR 文件中的其他应用程序不能在 manifest 文件中设置，但是我们仍然可以从命令行运行它们，就像我们对不可执行 JAR 那样。我们将在下一节中看到具体的方法。

## 5.运行带有参数的不可执行的 **JAR**

要在不可执行的 JAR 文件中运行应用程序，我们必须使用`-cp`选项，而不是`-jar`。

我们将使用 `-cp`选项(classpath 的缩写)来指定包含我们想要执行的类文件的 JAR 文件:

`**java -cp jar-file-name main-class-name [args …]**`

正如我们所看到的，**在这种情况下，我们必须在命令行中包含主类名，后跟参数。**

前面创建的不可执行 JAR 包含相同的简单应用程序。我们可以用任何参数(包括零)来运行它。

下面是一个有两个参数的例子:

```
$ java -cp JarExample2.jar com.baeldung.jarArguments.JarExample "arg 1" [[email protected]](/web/20221103233850/https://www.baeldung.com/cdn-cgi/l/email-protection)
```

就像我们在上面看到的一样，我们将看到以下输出:

```
Hello Baeldung Reader in JarExample!
There are 2 argument(s)!
Argument(1):arg 1
Argument(2):[[email protected]](/web/20221103233850/https://www.baeldung.com/cdn-cgi/l/email-protection)
```

## 6.结论

在本文中，我们学习了在命令行上运行 JAR 应用程序的两种方法，可以带参数也可以不带参数。

我们还演示了参数可以包含空格和特殊字符(如果 shell 允许的话)。

和往常一样，这些例子的代码可以在 GitHub 的[上找到。](https://web.archive.org/web/20221103233850/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-8-2)