# 格拉德勒简介

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/gradle>

[This article is part of a series:](javascript:void(0);)• Introduction to Gradle (current article)[• Ant vs Maven vs Gradle](/web/20220627191442/https://www.baeldung.com/ant-maven-gradle)
[• Writing Custom Gradle Plugins](/web/20220627191442/https://www.baeldung.com/gradle-create-plugin)
[• Creating a Fat Jar in Gradle](/web/20220627191442/https://www.baeldung.com/gradle-fat-jar)

## 1。概述

Gradle 是一个基于 Groovy 的构建管理系统，专门为构建基于 Java 的项目而设计。

安装说明可在[这里](https://web.archive.org/web/20220627191442/https://gradle.org/install/)找到。

## 2。构建模块-项目和任务

在 Gradle 中，构建由一个或多个项目组成，每个项目由一个或多个任务组成。

Gradle 中的一个项目可以组装一个`jar`、`war,`甚至是一个`zip`文件。

任务是一件单独的工作。这可以包括编译类，或者创建和发布 Java/web 档案。

一个简单的任务可以定义为:

```java
task hello {
    doLast {
        println 'Baeldung'
    }
}
```

如果我们在`build.gradle`所在的位置使用`gradle -q hello` 命令执行上述任务，我们应该会在控制台中看到输出。

### 2.1。任务

Gradle 的构建脚本非常棒:

```java
task toLower {
    doLast {
        String someString = 'HELLO FROM BAELDUNG'
        println "Original: "+ someString
        println "Lower case: " + someString.toLowerCase()
    }
}
```

我们可以定义依赖于其他任务的任务。可以通过在任务定义中传递`dependsOn: taskName` 参数来定义任务依赖性:

```java
task helloGradle {
    doLast {
        println 'Hello Gradle!'
    }
}

task fromBaeldung(dependsOn: helloGradle) {
    doLast {
        println "I'm from Baeldung"
    }
}
```

### 2.2。向任务添加行为

我们可以定义一个任务，并用一些额外的行为来增强它:

```java
task helloBaeldung {
    doLast {
        println 'I will be executed second'
    }
}

helloBaeldung.doFirst {
    println 'I will be executed first'
}

helloBaeldung.doLast {
    println 'I will be executed third'
}

helloBaeldung {
    doLast {
        println 'I will be executed fourth'
    }
}
```

`doFirst` 和`doLast`分别在动作列表的顶部和底部添加动作，在一个任务中可以多次定义**。**

### 2.3。添加任务属性

我们还可以定义属性:

```java
task ourTask {
    ext.theProperty = "theValue"
} 
```

这里，我们将`“theValue”`设置为`ourTask`任务的`theProperty` 。

## 3。管理插件

Gradle 中有两种类型的插件—`script,`和`binary.`

为了从附加功能中获益，每个插件都需要经历两个阶段:`resolving`和`applying.`

**`Resolving`表示找到插件 jar 的正确版本，并将其添加到项目的`classpath` 中。**

**`Applying`插件正在执行** `**Plugin.apply(T)**` **项目上的**。

### 3.1。应用脚本插件

在`aplugin.gradle,` 中我们可以定义一个任务:

```java
task fromPlugin {
    doLast {
        println "I'm from plugin"
    }
}
```

如果我们想将这个插件应用到我们的项目`build.gradle` 文件中，我们需要做的就是将这一行添加到我们的`build.gradle`中:

```java
apply from: 'aplugin.gradle' 
```

现在，执行`gradle tasks`命令应该会在任务列表中显示`fromPlugin` 任务。

### 3.2。使用插件 DSL 应用二进制插件

在添加核心二进制插件的情况下，我们可以添加短名称或插件 id:

```java
plugins {
    id 'application'
}
```

现在来自`application` 插件的`run`任务应该可以在项目中执行任何`runnable` jar。要应用一个社区插件，我们必须提到一个完全合格的插件 id:

```java
plugins {
    id "org.shipkit.bintray" version "2.3.5"
}
```

现在，`Shipkit`任务应该在`gradle tasks` 列表中可用。

插件 DSL 的局限性是:

*   它不支持`plugins` 块中的 Groovy 代码
*   `plugins` 块需要是项目构建脚本中的顶级语句(在它之前只允许有`buildscripts{}`块)
*   插件 DSL 不能写在脚本插件、`settings.gradle`文件或初始化脚本中

插件 DSL 还在孵化中。DSL 和其他配置可能会在以后的 Gradle 版本中发生变化。

### 3.3。应用插件的传统程序

我们也可以使用`“apply plugin”`来应用插件:

```java
apply plugin: 'war'
```

如果我们需要添加社区插件，我们必须使用`buildscript{}` 块将外部 jar 添加到构建类路径中。

然后，**我们可以在构建脚本中应用插件，但是** **只能在任何现有的`plugins{}` 块**之后:

```java
buildscript {
    repositories {
        maven {
            url "https://plugins.gradle.org/m2/"
        }
    }
    dependencies {
        classpath "org.shipkit:shipkit:2.3.5"
    }
}
apply plugin: "org.shipkit.bintray-release"
```

## 4。依赖性管理

Gradle 支持一个非常灵活的依赖管理系统，它兼容各种可用的方法。

Gradle 中依赖关系管理的最佳实践是版本控制、动态版本控制、解决版本冲突和管理可传递的依赖关系。

### 4.1。依赖配置

依赖关系被分组到不同的配置中。**一个配置有一个名字，它们可以互相扩展**。

如果我们应用 Java 插件，我们将有`implementation, testImplementation, runtimeOnly`配置可用于分组我们的依赖项。**`default c`配置延伸`runtimeOnly”.`**

### 4.2。声明依赖关系

让我们看一个使用几种不同方式添加一些依赖项(Spring 和 Hibernate)的例子:

```java
dependencies {
    implementation group: 
      'org.springframework', name: 'spring-core', version: '4.3.5.RELEASE'
    implementation 'org.springframework:spring-core:4.3.5.RELEASE',
            'org.springframework:spring-aop:4.3.5.RELEASE'
    implementation(
        [group: 'org.springframework', name: 'spring-core', version: '4.3.5.RELEASE'],
        [group: 'org.springframework', name: 'spring-aop', version: '4.3.5.RELEASE']
    )
    testImplementation('org.hibernate:hibernate-core:5.2.12.Final') {
        transitive = true
    }
    runtimeOnly(group: 'org.hibernate', name: 'hibernate-core', version: '5.2.12.Final') {
        transitive = false
    }
}
```

我们在不同的配置中声明依赖关系:不同格式的`implementation`、`testImplementation`和`runtimeOnly` 。

有时我们需要有多个工件的依赖。在这种情况下，我们可以添加一个只有工件的符号`@extensionName` (或者扩展形式的`ext`)来下载想要的工件:

```java
runtimeOnly "org.codehaus.groovy:groovy-all:[[email protected]](/web/20220627191442/https://www.baeldung.com/cdn-cgi/l/email-protection)"
runtimeOnly group: 'org.codehaus.groovy', name: 'groovy-all', version: '2.4.11', ext: 'jar'
```

这里，我们添加了`@jar`符号，以便只下载没有依赖项的 jar 工件。

要向任何本地文件添加依赖项，我们可以使用类似这样的方法:

```java
implementation files('libs/joda-time-2.2.jar', 'libs/junit-4.12.jar')
implementation fileTree(dir: 'libs', include: '*.jar')
```

**当我们想要避免传递性依赖时，** **我们可以在配置级或者依赖级上做**:

```java
configurations {
    testImplementation.exclude module: 'junit'
}

testImplementation("org.springframework.batch:spring-batch-test:3.0.7.RELEASE"){
    exclude module: 'junit'
}
```

## 5。多项目构建

### 5.1。构建生命周期

在初始化阶段，Gradle 决定哪些项目将参与多项目构建。

这通常在`settings.gradle` 文件中提到，它位于项目根目录下。Gradle 还创建参与项目的实例。

**在配置阶段，所有创建的项目实例都是基于梯度特性按需配置的。**

在这个特性中，只为特定的任务执行配置必需的项目。这样，对于大型多项目构建来说，配置时间会大大减少。这个功能还在孵化中。

最后，**在执行阶段，执行创建和配置的任务子集。**我们可以在`settings.gradle` 和`build.gradle` 文件中包含代码来感知这三个阶段。

在`settings.gradle`中:

```java
println 'At initialization phase.'
```

在`build.gradle` 中:

```java
println 'At configuration phase.'

task configured { println 'Also at the configuration phase.' }

task execFirstTest { doLast { println 'During the execution phase.' } }

task execSecondTest {
    doFirst { println 'At first during the execution phase.' }
    doLast { println 'At last during the execution phase.' }
    println 'At configuration phase.'
}
```

### 5.2。创建多项目构建

我们可以在根文件夹中执行`gradle init` 命令，为`settings.gradle`和`build.gradle`文件创建一个框架。

所有常见的配置都将保存在根构建脚本中:

```java
allprojects {
    repositories {
        mavenCentral() 
    }
}

subprojects {
    version = '1.0'
}
```

设置文件需要包括根项目名称和子项目名称:

```java
rootProject.name = 'multi-project-builds'
include 'greeting-library','greeter'
```

现在我们需要几个名为`greeting-library`和`greeter` 的子项目文件夹来演示多项目构建。每个子项目都需要有一个单独的构建脚本来配置其单独的依赖项和其他必要的配置。

如果我们希望我们的`greeter` 项目依赖于`greeting-library`，我们需要在`greeter`的构建脚本中包含这种依赖:

```java
dependencies {
    implementation project(':greeting-library') 
}
```

## 6。使用 Gradle Wrapper

如果一个 Gradle 项目有用于 Linux 的`gradlew` 文件和用于 Windows 的`gradlew.bat` 文件，我们不需要安装 Gradle 来构建项目。

如果我们在 Windows 中执行`gradlew` `build`，在 Linux 中执行`./gradlew build` ，则会自动下载`gradlew`文件中指定的梯度分布。

如果我们想将 Gradle 包装器添加到我们的项目中:

```java
gradle wrapper --gradle-version 7.2
```

该命令需要从项目的根目录执行。这将创建所有必要的文件和文件夹，以便将 Gradle wrapper 绑定到项目。另一种方法是将包装任务添加到构建脚本中:

```java
wrapper {
    gradleVersion = '7.2'
}
```

现在我们需要执行*包装器*任务，该任务将我们的项目绑定到包装器上。除了`gradlew`文件，在包含 jar 和属性文件的`gradle` 文件夹中还会生成一个`wrapper` 文件夹。

如果我们想切换到 Gradle 的新版本，只需要改变`gradle-` `wrapper.properties`中的一个条目。

## 7 .**。结论**

在本文中，我们看了一下 Gradle，发现它在解决版本冲突和管理传递性依赖方面比其他现有的构建工具具有更大的灵活性。

这篇文章的源代码可以在 GitHub 的[上找到。](https://web.archive.org/web/20220627191442/https://github.com/eugenp/tutorials/tree/master/gradle)

Next **»**[Ant vs Maven vs Gradle](/web/20220627191442/https://www.baeldung.com/ant-maven-gradle)