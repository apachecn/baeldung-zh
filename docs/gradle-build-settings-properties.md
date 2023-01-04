# Gradle: build.gradle vs. settings.gradle vs. gradle.properties

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/gradle-build-settings-properties>

## 1.概观

在本文中，**我们将看看 Gradle Java 项目**的不同配置文件。**此外，我们将看到实际建造的细节。**

你可以查看这篇文章来了解关于 Gradle 的一般介绍。

## 2.`build.gradle`

让我们假设我们只是通过运行`gradle init –type java-application`来创建一个新的 Java 项目。这将给我们留下一个新项目，其目录和文件结构如下:

```java
build.gradle
gradle    
    wrapper
        gradle-wrapper.jar
        gradle-wrapper.properties
gradlew
gradlew.bat
settings.gradle
src
    main
        java  
            App.java
    test      
        java
            AppTest.java
```

我们可以将`build.gradle`文件视为项目的心脏或大脑。我们示例的结果文件如下所示:

```java
plugins {
    id 'java'
    id 'application'
}

mainClassName = 'App'

dependencies {
    compile 'com.google.guava:guava:23.0'

    testCompile 'junit:junit:4.12'
}

repositories {
    jcenter()
}
```

它由 Groovy 代码组成，或者更准确地说，是一种用于描述构建的基于 Groovy 的 DSL(领域特定语言)。我们可以在这里定义我们的依赖关系，还可以添加一些东西，比如用于依赖关系解析的 Maven 存储库。

Gradle 的基本构件是项目和任务。在这种情况下，由于应用了`java`插件，构建 Java 项目的所有必要任务都是隐式定义的。其中一些任务是`assemble`、`check`、`build`、`jar`、`javadoc`、`clean`等等。

这些任务也是以这样一种方式建立的，它们描述了一个 Java 项目的有用的依赖图，这意味着它通常足以执行构建任务，Gradle(和 Java 插件)将确保所有必要的任务都被执行。

如果我们需要额外的特殊任务，比如构建 Docker 映像，它也会进入到`build.gradle`文件中。最简单的任务定义如下:

```java
task hello {
    doLast {
        println 'Hello Baeldung!'
    }
}
```

我们可以通过将任务指定为 Gradle CLI 的参数来运行任务，如下所示:

```java
$ gradle -q hello
Hello Baeldung!
```

它不会做任何有用的事情，但会打印出“Hello Baeldung！”当然了。

在多项目构建的情况下，我们可能会有多个不同的`build.gradle`文件，每个项目一个。

对一个 [`Project`](https://web.archive.org/web/20220831003306/https://docs.gradle.org/current/javadoc/org/gradle/api/Project.html) 实例执行`build.gradle`文件，每个子项目创建一个项目实例。上面的任务可以在`build.gradle`文件中定义，作为 [`Task`](https://web.archive.org/web/20220831003306/https://docs.gradle.org/current/javadoc/org/gradle/api/Task.html) 对象集合的一部分驻留在`Project`实例中。任务本身由多个作为有序列表的动作组成。

在前面的例子中，我们添加了一个 Groovy 闭包，用于打印“Hello Baeldung！”要结束这个列表，通过调用我们的`hello` `Task`对象上的`doLast(Closure action)`。在执行`Task`的过程中，Gradle 通过调用`Action.execute(T)`方法，依次执行它的每个`Actions`。

## 3.`settings.gradle`

Gradle 还生成一个`settings.gradle`文件:

```java
rootProject.name = 'gradle-example'
```

`settings.gradle`文件也是一个 Groovy 脚本。

与`build.gradle`文件相反，每次构建只执行一个`settings.gradle`文件。我们可以使用它来定义多项目构建的项目。

此外，我们还可以将代码注册为一个构建的不同生命周期挂钩的一部分。

该框架要求在多项目构建中存在`settings.gradle`,而在单项目构建中是可选的。

在创建了构建的 [`Settings`](https://web.archive.org/web/20220831003306/https://docs.gradle.org/current/dsl/org.gradle.api.initialization.Settings.html) 实例之后，通过对其执行文件并由此对其进行配置来使用该文件。这意味着我们在`settings.gradle`文件中定义子项目，如下所示:

```java
include 'foo', 'bar'
```

当创建构建时，Gradle 正在调用`Settings`实例上的`void include(String… projectPaths)`方法。

## 4.`gradle.properties`

**Gradle 默认不创建`gradle.properties`文件。它可以驻留在不同的位置，例如在项目根目录中，在`GRADLE_USER_HOME`内部或者在由`-Dgradle.user.home`命令行标志指定的位置。**

这个文件由键值对组成。我们可以使用它来配置框架本身的行为，这是使用命令行标志进行配置的一种替代方法。

可能的密钥示例如下:

*   `org.gradle.caching=(true,false)`
*   `org.gradle.daemon=(true,false)`
*   `org.gradle.parallel=(true,false)`
*   `org.gradle.logging.level=(quiet,warn,lifecycle,info,debug)`

此外，您可以使用这个文件直接向`Project`对象添加属性，例如，带有名称空间:`org.gradle.project.property_to_set`的属性

另一个用例是像这样指定 JVM 参数:

```java
org.gradle.jvmargs=-Xmx2g -XX:MaxPermSize=256m -XX:+HeapDumpOnOutOfMemoryError -Dfile.encoding=UTF-8
```

请注意，它需要启动一个 JVM 进程来解析`gradle.properties`文件。这意味着这些 JVM 参数只影响单独启动的 JVM 进程。

## 5.简而言之

我们可以将 Gradle 构建的一般生命周期总结如下，假设我们不将它作为守护进程运行:

*   它作为一个新的 JVM 进程启动
*   它解析`gradle.properties`文件并相应地配置 Gradle
*   接下来，它为构建创建一个`Settings`实例
*   然后，它根据`Settings`对象评估`settings.gradle`文件
*   它基于配置的`Settings`对象创建了一个`Projects`的层次结构
*   最后，它针对项目执行每个`build.gradle`文件

## 6.结论

我们已经看到，不同的 Gradle 配置文件如何满足不同的开发目的。我们可以根据项目的需要，使用它们来配置 Gradle 版本以及 Gradle 本身。