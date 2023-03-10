# Java 14 中的 jpackage 指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java14-jpackage>

## 1。概述

在本教程中，我们将探索在 [Java 14](https://web.archive.org/web/20220813060412/https://openjdk.java.net/projects/jdk/14/) 中引入的名为`[jpackage](https://web.archive.org/web/20220813060412/https://openjdk.java.net/jeps/343)`的新打包工具。

## 2。简介

**`jpackage`是一个命令行工具，用于为 Java 应用程序创建本地安装程序和包。**

这是`jdk.incubator.jpackage`模块下的孵化功能。换句话说，该工具的命令行选项或应用程序布局还不稳定。一旦稳定，Java SE 平台或 JDK 将在 LTE 版本中包含该特性。

## 3。为什么`jpackage?`

在分发软件时，将可安装包交付给最终用户是一种标准做法。该软件包与用户的本机平台兼容，并隐藏了内部依赖关系和设置配置。例如，我们在 macOS 上使用 DMG 文件，在 Windows 上使用 MSI 文件。

这允许以最终用户熟悉的方式分发、安装和卸载应用程序。

允许开发者为他们的 JAR 文件创建这样一个可安装的包。**用户不必显式复制 JAR 文件，甚至不必安装 Java 来运行应用程序。**可安装软件包会处理所有这些事情。

## 4.打包先决条件

使用`jpackage`命令的关键先决条件是:

1.  用于打包的系统必须包含要打包的应用程序、JDK 和打包工具所需的软件。
2.  而且，它需要有`jpackage`使用的底层打包工具:
    *   Linux 上的 RPM、DEB:在 Red Hat Linux 上，我们需要`rpm-build`包；在 Ubuntu Linux 上，我们需要`fakeroot`包
    *   macOS 上的 DMG PKG:当使用`–mac-sign`选项请求对包进行签名，以及使用`–icon`选项定制 DMG 映像时，需要 Xcode 命令行工具
    *   在 Windows 上:在 Windows 上，我们需要第三方工具 WiX 3.0 或更高版本
3.  最后，应用程序包必须在目标平台上构建。这意味着要为多个平台打包应用程序，我们必须在每个平台上运行打包工具。

## 5。包创建

让我们为应用程序 JAR 创建一个样例包。如前所述，应用程序 JAR 应该是预先构建的，它将被用作`jpackage`工具的输入。

例如，我们可以使用以下命令创建一个包:

```java
jpackage --input target/ \
  --name JPackageDemoApp \
  --main-jar JPackageDemoApp.jar \
  --main-class com.baeldung.java14.jpackagedemoapp.JPackageDemoApp \
  --type dmg \
  --java-options '--enable-preview'
```

让我们看一下使用的每个选项:

*   `–input`:输入 jar 文件的位置
*   `–name`:给可安装包起一个名字
*   `–main-jar`:应用程序启动时启动的 JAR 文件
*   `–main-class`:在应用程序启动时启动的 JAR 中的主类名。如果主 JAR 中的`MANIFEST.MF`文件包含主类名，这是可选的。
*   我们想要创建什么样的安装程序？这取决于我们运行`jpackage`命令的基础操作系统。在 macOS 上，我们可以将包类型作为 DMG 或 PKG 来传递。该工具在 Windows 上支持 MSI 和 EXE 选项，在 Linux 上支持 DEB 和 RPM 选项。
*   `–java-options`:传递给 Java 运行时的选项

上面的命令将为我们创建`JPackageDemoApp.dmg`文件。

然后我们可以使用这个文件在 macOS 平台上安装应用程序。安装完成后，我们就可以像使用其他软件一样使用这个应用程序了。

## 6。结论

在本文中，我们看到了 Java 14 中引入的`jpackage`命令行工具的用法。