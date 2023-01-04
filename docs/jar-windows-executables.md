# 从 Java 创建 Jar 可执行文件和 Windows 可执行文件的指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/jar-windows-executables>

## 1.概观

在本教程中，我们将从学习如何将一个 Java 程序打包成一个可执行的 Java 归档(JAR)文件开始。然后，我们将看到如何使用这个可执行 JAR 生成一个 Microsoft Windows 支持的可执行文件。

**我们将使用 Java 自带的`jar`命令行工具来创建 JAR 文件。然后我们将学习使用`jpackage` 工具，在 Java 16 和更高版本中可以作为`jdk.jpackage,`来生成一个可执行文件。**

## 2.`jar`和`jpackage`命令的基础

一个 [JAR 文件](/web/20220821150146/https://www.baeldung.com/java-create-jar)是编译后的 Java 类文件和其他资源的容器。它基于流行的 ZIP 文件格式。

可执行 JAR 文件也是一个 JAR 文件，但是也包含一个主类。main 类在一个 manifest 文件中被引用，我们稍后将讨论这个文件。

为了运行以 JAR 格式交付的应用程序，我们必须有一个 Java 运行时环境(JRE)。

**与 JAR 文件不同，特定于平台的可执行文件可以在为**构建的平台上本地运行。例如，该平台可以是微软的 Windows、Linux 或苹果的 macOS。

**为了获得良好的最终用户体验，最好为客户端提供特定于平台的可执行文件**。

### 2.1.`jar`命令

创建 JAR 文件的一般语法是:

```
jar cf jar-file input-file(s)
```

让我们来看看在使用`jar`命令创建新的归档文件时可以使用的一些选项:

*   `c`指定我们想要创建一个 JAR 文件
*   指定我们希望输出到一个文件
*   `m`用于包含现有清单文件中的清单信息
*   `jar-file`是我们希望得到的 JAR 文件的名称。JAR 文件通常有一个`.jar`扩展名，但这不是必需的。
*   `input-file(s)`是一个以空格分隔的文件名列表，我们希望将它包含在我们的 JAR 文件中。通配符`*`也可以在这里使用。

一旦我们创建了一个 JAR 文件，我们将经常检查它的内容。要查看 JAR 文件包含的内容，我们使用以下语法:

```
jar tf jar-file 
```

这里，`t`表示我们想要列出 JAR 文件的内容。`f`选项表示我们想要检查的 JAR 文件是在命令行上指定的。

### 2.2.`jpackage`命令

**[`jpackage`](/web/20220821150146/https://www.baeldung.com/java14-jpackage)命令行工具帮助我们为模块化和非模块化的 Java 应用程序生成可安装包**。

它使用 [`jlink`](/web/20220821150146/https://www.baeldung.com/jlink) 命令为我们的应用程序生成一个 Java 运行时映像。因此，我们获得了一个用于特定平台的自包含应用程序包。

由于应用程序包是为目标平台构建的，因此该系统必须包含以下内容:

*   应用程序本身
*   a JDK
*   打包工具需要的软件。**对于 Windows，`jpackage`需要 WiX 3.0 或更高版本**。

下面是 [`jpackage`](/web/20220821150146/https://www.baeldung.com/java14-jpackage) 命令的常用形式:

`jpackage --input . --main-jar MyAppn.jar`

## 3.创建可执行文件

现在让我们来创建一个可执行的 JAR 文件。一旦准备好了，我们将开始生成一个 Windows 可执行文件。

### 3.1.创建可执行的 JAR 文件

创建一个可执行的 JAR 相当简单。我们首先需要一个 Java 项目，其中至少有一个使用`main()`方法的类。在我们的例子中，我们创建了一个名为`MySampleGUIAppn`的 Java 类。

第二步是创建一个清单文件。让我们将清单文件创建为`MySampleGUIAppn.mf`:

```
Manifest-Version: 1.0
Main-Class: MySampleGUIAppn 
```

我们必须确保在这个清单文件的末尾有一个换行符，这样它才能正常工作。

一旦清单文件准备就绪，我们将创建一个可执行的 JAR:

```
jar cmf MySampleGUIAppn.mf MySampleGUIAppn.jar MySampleGUIAppn.class MySampleGUIAppn.java
```

让我们来查看我们创建的 JAR 的内容:

```
jar tf MySampleGUIAppn.jar
```

下面是一个输出示例:

```
META-INF/
META-INF/MANIFEST.MF
MySampleGUIAppn.class
MySampleGUIAppn.java
```

接下来，我们可以通过 CLI 或在 GUI 中运行 JAR 可执行文件。

让我们在命令行上运行它:

```
java -jar MySampleGUIAppn.jar
```

在 GUI 中，我们可以简单地双击相关的 JAR 文件。这应该可以像其他应用程序一样正常启动它。

### 3.2.创建 Windows 可执行文件

现在，我们的可执行 JAR 已经准备好并可以工作了，让我们为我们的示例项目生成一个 Windows 可执行文件:

```
jpackage --input . --main-jar MySampleGUIAppn.jar
```

完成此命令需要一段时间。一旦完成，它会在当前工作文件夹中生成一个`exe`文件。可执行文件的文件名将与清单文件中提到的版本号连接在一起。我们将能够像启动任何其他 Windows 应用程序一样启动它。

以下是我们可以使用`jpackage`命令的更多特定于 Windows 的选项:

*   `–type`:指定`msi` 而不是默认的`exe` 格式
*   用控制台窗口启动我们的应用程序
*   `–win-shortcut`:在 Windows 开始菜单中创建快捷方式文件
*   `–win-dir-chooser`:让最终用户指定安装可执行文件的自定义目录
*   `–win-menu –win-menu-group`:让最终用户在开始菜单中指定自定义目录

## 4.结论

在本文中，我们学习了一些关于 JAR 文件和可执行 JAR 文件的基础知识。我们还看到了如何将 Java 程序转换成 JAR 可执行文件，然后再转换成 Microsoft Windows 支持的可执行文件。

与往常一样，GitHub 上的[提供了示例的源代码。](https://web.archive.org/web/20220821150146/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-jar)