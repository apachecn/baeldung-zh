# 在 Gradle 中传递命令行参数

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/gradle-command-line-arguments>

## 1.概观

有时，我们想从 [Gradle](/web/20221010114344/https://www.baeldung.com/gradle) 开始执行各种需要输入参数的程序。

在这个快速教程中，我们将看到如何从 Gradle 传递命令行参数。

## 2.输入参数的类型

当我们想从 Gradle CLI 传递输入参数时，我们有两种选择:

*   用`-D`标志设置系统属性
*   用`-P`标志设置项目属性

一般来说，**我们应该使用项目属性，除非我们想在 JVM** 中定制设置。

虽然劫持系统属性来传递我们的输入是可能的，但是我们应该避免这样做。

让我们看看这些属性的作用。首先，我们配置我们的`build.gradle`:

```java
apply plugin: "java"
description = "Gradle Command Line Arguments examples"

task propertyTypes(){
    doLast{
        if (project.hasProperty("args")) {
            println "Our input argument with project property ["+project.getProperty("args")+"]"
        }
        println "Our input argument with system property ["+System.getProperty("args")+"]"
    }
} 
```

请注意，我们在任务中阅读它们的方式不同。

我们这样做是因为 **`project.` *getProperty()* 在我们的属性没有被定义**的情况下抛出一个 *MissingPropertyException* 。

与项目属性不同，`System.getProperty()`在属性未定义的情况下返回一个`null`值。

接下来，让我们运行任务并查看其输出:

```java
$ ./gradlew propertyTypes -Dargs=lorem -Pargs=ipsum

> Task :cmd-line-args:propertyTypes
Our input argument with project property [ipsum]
Our input argument with system property [lorem] 
```

## 3.传递命令行参数

到目前为止，我们已经看到了如何读取属性。实际上，我们需要将这些属性作为参数发送给我们选择的程序。

### 3.1.向 Java 应用程序传递参数

在之前的教程中，我们解释了如何从 Gradle 运行 Java 主类。让我们在此基础上，看看如何传递参数。

首先，让我们**使用我们`build.gradle`** 中的应用插件:

```java
apply plugin: "java"
apply plugin: "application"
description = "Gradle Command Line Arguments examples"

// previous declarations

ext.javaMainClass = "com.baeldung.cmd.MainClass"

application {
    mainClassName = javaMainClass
} 
```

现在，让我们来看看我们的主类:

```java
public class MainClass {
    public static void main(String[] args) {
        System.out.println("Gradle command line arguments example");
        for (String arg : args) {
            System.out.println("Got argument [" + arg + "]");
        }
    }
} 
```

接下来，让我们用一些参数来运行它:

```java
$ ./gradlew :cmd-line-args:run --args="lorem ipsum dolor"

> Task :cmd-line-args:run
Gradle command line arguments example
Got argument [lorem]
Got argument [ipsum]
Got argument [dolor]
```

这里，我们不使用属性来传递参数。相反，**我们传递*–args*标志和相应的输入**。

这是一个由应用程序插件提供的很好的包装器。然而，**这只能从 Gradle 4.9 开始使用**。

让我们看看使用`JavaExec`任务的**会是什么样子。**

首先，我们需要在我们的`build.gradle`中定义它:

```java
ext.javaMainClass = "com.baeldung.cmd.MainClass"
if (project.hasProperty("args")) {
    ext.cmdargs = project.getProperty("args")
} else { 
    ext.cmdargs = ""
}
task cmdLineJavaExec(type: JavaExec) {
    group = "Execution"
    description = "Run the main class with JavaExecTask"
    classpath = sourceSets.main.runtimeClasspath
    main = javaMainClass
    args cmdargs.split()
} 
```

让我们仔细看看我们做了什么。我们首先**从项目属性**中读取参数。

因为这包含了一个字符串中的所有参数，所以我们**使用 *split* 方法来获得一个参数数组**。

接下来，我们**将这个数组传递给我们的`JavaExec`任务**的*参数*属性。

让我们看看当我们运行这个任务时会发生什么，用`-P`选项传递项目属性:

```java
$ ./gradlew cmdLineJavaExec -Pargs="lorem ipsum dolor"

> Task :cmd-line-args:cmdLineJavaExec
Gradle command line arguments example
Got argument [lorem]
Got argument [ipsum]
Got argument [dolor] 
```

### 3.2.将参数传递给其他应用程序

在某些情况下，我们可能希望将一些参数传递给来自 Gradle 的第三方应用程序。

幸运的是，我们可以**使用更通用的*执行*任务**来做到这一点:

```java
if (project.hasProperty("args")) {
    ext.cmdargs = project.getProperty("args")
} else { 
    ext.cmdargs = "ls"
}

task cmdLineExec(type: Exec) {
    group = "Execution"
    description = "Run an external program with ExecTask"
    commandLine cmdargs.split()
} 
```

这里，我们**使用任务的*命令行*属性来传递可执行文件以及任何参数**。同样，我们基于空格分割输入。

让我们看看如何运行 *ls* 命令:

```java
$ ./gradlew cmdLineExec -Pargs="ls -ll"

> Task :cmd-line-args:cmdLineExec
total 4
drwxr-xr-x 1 user 1049089    0 Sep  1 17:59 bin
drwxr-xr-x 1 user 1049089    0 Sep  1 18:30 build
-rw-r--r-- 1 user 1049089 1016 Sep  3 15:32 build.gradle
drwxr-xr-x 1 user 1049089    0 Sep  1 17:52 src
```

如果我们不想在任务中硬编码可执行文件，这可能非常有用。

## 4.结论

在这个快速教程中，我们看到了如何从 Gradle 传递输入参数。

首先，我们解释了可以使用的属性类型。虽然我们可以使用系统属性来传递输入参数，但我们应该更喜欢使用项目属性。

然后，我们探索了向 Java 或外部应用程序传递命令行参数的不同方法。

像往常一样，完整的代码可以在 GitHub 上找到[。](https://web.archive.org/web/20221010114344/https://github.com/eugenp/tutorials/tree/master/gradle-modules/gradle-5/cmd-line-args)