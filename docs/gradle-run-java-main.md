# 使用 Gradle 运行 Java main 方法

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/gradle-run-java-main>

## 1.介绍

在本教程中，我们将探索使用 Gradle 执行一个 [Java `main`方法](/web/20220916112551/https://www.baeldung.com/java-main-method)的不同方法。

## 2.Java `main`方法

有几种方法可以让我们用 Gradle 运行 Java `main`方法。让我们使用一个简单的程序来仔细查看它们，该程序将一条消息打印到标准输出中:

```java
public class MainClass {
    public static void main(String[] args) {
        System.out.println("Goodbye cruel world ...");
    }
}
```

## 3.使用应用程序插件运行

Application plugin 是一个核心 Gradle 插件，它定义了一组现成的任务，帮助我们打包和分发我们的应用程序。

让我们首先在我们的`build.gradle`文件中插入以下内容:

```java
plugins {
    id "application"
}
apply plugin : "java" 
ext {
   javaMainClass = "com.baeldung.gradle.exec.MainClass"
}

application {
    mainClassName = javaMainClass
}
```

插件自动生成一个名为`run`的任务，只需要我们将它指向`main`类。第 9 行的闭包正是这样做的，它允许我们触发任务:

```java
~/work/baeldung/tutorials/gradle-java-exec> ./gradlew run

> Task :run
Goodbye cruel world ...

BUILD SUCCESSFUL in 531ms
2 actionable tasks: 1 executed, 1 up-to-date
```

## 4.运行`JavaExec`任务

接下来，让我们在`JavaExec`任务类型的帮助下实现一个用于运行`main`方法的定制任务:

```java
task runWithJavaExec(type: JavaExec) {
    group = "Execution"
    description = "Run the main class with JavaExecTask"
    classpath = sourceSets.main.runtimeClasspath
    main = javaMainClass
}
```

我们需要在第 5 行定义`main`类，此外，还要指定类路径。类路径是根据构建输出的默认属性计算出来的，包含编译后的类实际所在的正确路径。

注意，在每个场景中，我们**使用了`main`类**的完全限定名，包括包。

让我们使用`JavaExec`来运行我们的例子:

```java
~/work/baeldung/tutorials/gradle-java-exec> ./gradlew runWithJavaExec

> Task :runWithJavaExec
Goodbye cruel world ...

BUILD SUCCESSFUL in 526ms
2 actionable tasks: 1 executed, 1 up-to-date
```

## 5.运行`Exec`任务

最后，我们可以使用基本的`Exec`任务类型执行我们的`main`类。因为这个选项为我们提供了以多种方式配置执行的可能性，所以让我们实现三个定制任务并分别讨论它们。

### 5.1.从编译后的生成输出运行

首先，我们创建一个定制的`Exec`任务，其行为类似于`JavaExec`:

```java
task runWithExec(type: Exec) {
    dependsOn build
    group = "Execution"
    description = "Run the main class with ExecTask"
    commandLine "java", "-classpath", sourceSets.main.runtimeClasspath.getAsPath(), javaMainClass
}
```

我们可以运行任何可执行文件(在本例中是 java ),并为它的运行传递必要的参数。

我们配置了类路径并指向第 5 行的`main`类，我们还在第 2 行添加了对`build`任务的依赖。这是必要的，因为我们只能在编译后运行我们的`main`类:

```java
~/work/baeldung/tutorials/gradle-java-exec> ./gradlew runWithExec

> Task :runWithExec
Goodbye cruel world ...

BUILD SUCCESSFUL in 666ms
6 actionable tasks: 6 executed 
```

### 5.2.从输出 Jar 运行

第二种方法依赖于我们的小应用程序的 jar 打包:

```java
task runWithExecJarOnClassPath(type: Exec) {
    dependsOn jar
    group = "Execution"
    description = "Run the mainClass from the output jar in classpath with ExecTask"
    commandLine "java", "-classpath", jar.archiveFile.get(), javaMainClass
} 
```

注意第 2 行中 jar 任务的依赖性和第 5 行中 java 可执行文件的第二个参数。**我们使用一个普通的 jar，所以我们需要用第四个参数指定入口点:**

```java
~/work/baeldung/tutorials/gradle-java-exec> ./gradlew runWithExecJarOnClassPath

> Task :runWithExecJarOnClassPath
Goodbye cruel world ...

BUILD SUCCESSFUL in 555ms
3 actionable tasks: 3 executed
```

### 5.3.从可执行输出 Jar 运行

第三种方式也依赖于 jar 打包，但是我们在一个`manifest`属性的帮助下定义入口点:

```java
jar {
    manifest {
        attributes(
            "Main-Class": javaMainClass
        )
    }
}

task runWithExecJarExecutable(type: Exec) {
    dependsOn jar
    group = "Execution"
    description = "Run the output executable jar with ExecTask"
    commandLine "java", "-jar", jar.archiveFile.get()
} 
```

在这里，**我们不再需要指定类路径**，我们可以简单地运行 jar:

```java
~/work/baeldung/tutorials/gradle-java-exec> ./gradlew runWithExecJarExecutable

> Task :runWithExecJarExecutable
Goodbye cruel world ...

BUILD SUCCESSFUL in 572ms
3 actionable tasks: 3 executed
```

## 6.结论

在本文中，我们探索了使用 Gradle 运行 Java `main`方法的各种方式。

开箱即用的应用程序插件提供了一个最低限度可配置的任务来运行我们的方法。任务类型允许我们在不指定任何插件的情况下运行方法。

最后，通用的`Exec`任务类型可以与 java 可执行文件进行各种组合，以获得相同的结果，但是需要依赖于其他任务。

像往常一样，本教程的源代码可以在 GitHub 上的[处获得。](https://web.archive.org/web/20220916112551/https://github.com/eugenp/tutorials/tree/master/gradle-modules/gradle-5)