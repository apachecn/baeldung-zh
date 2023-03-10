# jlink 指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/jlink>

## 1.概观

**[`jlink`](https://web.archive.org/web/20221123135231/https://docs.oracle.com/javase/9/tools/jlink.htm#JSWOR-GUID-CECAC52B-CFEE-46CB-8166-F17A8E9280E9) 是一个生成自定义 Java 运行时映像的工具，该映像仅包含给定应用程序所需的平台模块。**

这种运行时映像的行为与 JRE 完全一样，但只包含我们挑选的模块和它们运行所需的依赖关系。模块化运行时映像的概念在 [JEP 220](https://web.archive.org/web/20221123135231/https://openjdk.java.net/jeps/220) 中引入。

在本教程中，我们将学习如何使用`jlink`创建一个定制的 JRE，我们还将运行并测试我们的模块在 JRE 中的功能是否正确。

## 2.需要创建自定义 JRE

让我们通过一个例子来理解定制运行时映像背后的动机。

我们将创建一个简单的模块化应用程序。要了解更多关于创建模块化应用程序的信息，请参考我们关于模块化的文章。

首先，让我们创建一个`HelloWorld`类和一个相应的模块:

```java
public class HelloWorld {
    private static final Logger LOG = Logger.getLogger(HelloWorld.class.getName());
    public static void main(String[] args) {
        LOG.info("Hello World!");
    }
}
```

```java
module jlinkModule {
    requires java.logging;
}
```

要运行这个程序，我们只需要`HelloWorld,``String``Logger`和`Object` 类。

尽管这个程序只需要四个类来运行，JRE 中所有预定义的类也会被执行，即使我们的程序不需要它们。

所以，要运行一个小程序，就得维护一个完整的 JRE，简直就是浪费内存。

因此，定制的 JRE 是运行我们的示例的最佳选择。

**使用`jlink`，我们可以创建我们自己的小型 JRE，它只包含我们想要使用的相关类，而不会浪费内存，因此，我们将看到性能的提高。**

## 3。构建定制的 Java 运行时映像

我们将执行一系列简单的步骤来创建定制的 JRE 映像。

### 3.1.编译模块

首先，让我们从命令行编译上面提到的程序:

```java
javac -d out module-info.java
```

```java
javac -d out --module-path out com\baeldung\jlink\HelloWorld.java
```

现在，让我们运行程序:

```java
java --module-path out --module jlinkModule/com.baeldung.jlink.HelloWorld
```

输出将是:

```java
Mar 13, 2019 10:15:40 AM com.baeldung.jlink.HelloWorld main
INFO: Hello World!
```

### 3.2.使用`jdeps`列出从属模块

为了使用`jlink`，我们需要知道应用程序使用的 JDK 模块列表，以及我们应该包含在自定义 JRE 中的模块列表。

让我们使用 `[jdeps](https://web.archive.org/web/20221123135231/https://docs.oracle.com/javase/9/tools/jdeps.htm#JSWOR690)` 命令来获取应用程序中使用的依赖模块:

```java
jdeps --module-path out -s --module jlinkModule
```

输出将是:

```java
jlinkModule -> java.base
jlinkModule -> java.logging
```

这是有意义的，因为`java.base`是 Java 代码库所需的最小模块，而 `java.logging`由我们程序中的记录器使用。

### 3.3.使用`jlink`创建自定义 JRE

要为基于模块的应用程序创建定制的 JRE，我们可以使用`jlink` 命令。下面是它的基本语法:

```java
jlink [options] –module-path modulepath
  –add-modules module [, module…]
  --output <target-directory>
```

现在，让我们使用 Java 11 为我们的程序创建一个定制的 JRE:

```java
jlink --module-path "%JAVA_HOME%\jmods";out
  --add-modules jlinkModule
  --output customjre
```

这里，`–add-modules`参数后的值告诉`jlink`JRE 中包含哪个模块。

最后，`–output`参数旁边的`customjre` 定义了应该生成自定义 JRE 的目标目录。

注意，在本教程中，我们使用 Windows shell 来执行所有命令。Linux 和 Mac 用户可能需要稍微调整一下。

### 3.4.使用生成的图像运行应用程序

现在，我们有了由`jlink`创建的自定义 JRE。

为了测试我们的 JRE，让我们尝试通过在我们的`customjre`目录的`bin`文件夹中导航并运行下面的命令来运行我们的模块:

```java
java --module jlinkModule/com.baeldung.jlink.HelloWorld
```

同样，我们使用的 Windows shell 在前进到路径之前会在当前目录中查找任何可执行文件。我们需要额外注意实际运行我们的自定义 JRE，而不是在 Linux 或 Mac 上解析路径的 `java` 。

## 4.使用启动器脚本创建自定义 JRE

可选地，**我们也可以用可执行的`launcher`脚本**创建一个定制的 JRE。

为此，我们需要运行带有额外的**参数`–launcher`的`jlink` 命令，用我们的模块和主类**创建我们的启动器:

```java
jlink --launcher customjrelauncher=jlinkModule/com.baeldung.jlink.HelloWorld
  --module-path "%JAVA_HOME%\jmods";out
  --add-modules jlinkModule
  --output customjre
```

这将在我们的`customjre/bin`目录中生成两个脚本:`customjrelauncher.bat`和`customjrelauncher` 。

让我们运行脚本:

```java
customjrelauncher.bat
```

输出将是:

```java
Mar 18, 2019 12:34:21 AM com.baeldung.jlink.HelloWorld main
INFO: Hello World!
```

## 5.结论

在本教程中，我们已经学习了如何使用`jlink`创建一个定制的、模块化的 JRE，它只包含我们的模块所需的最少文件。我们还研究了如何创建一个带有启动器脚本的定制 JRE，以便于执行和发布。

定制的、模块化的 Java 运行时映像非常强大。创建定制 JRE 的目标很明确:节省内存，提高性能，同时增强安全性和可维护性。轻量级定制 JREs 也使我们能够为小型设备创建可伸缩的应用程序。

本教程中使用的代码片段[可从 Github](https://web.archive.org/web/20221123135231/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-11) 获得。