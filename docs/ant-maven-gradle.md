# 蚂蚁 vs Maven vs Gradle

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/ant-maven-gradle>

[This article is part of a series:](javascript:void(0);)[• Introduction to Gradle](/web/20220703135935/https://www.baeldung.com/gradle)
• Ant vs Maven vs Gradle (current article)[• Writing Custom Gradle Plugins](/web/20220703135935/https://www.baeldung.com/gradle-create-plugin)
[• Creating a Fat Jar in Gradle](/web/20220703135935/https://www.baeldung.com/gradle-fat-jar)

## 1。简介

在本文中，我们将探索主导 JVM 生态系统的三种 Java 构建自动化工具——Ant、Maven 和 Gradle。

我们将逐一介绍它们，并探索 Java 构建自动化工具是如何发展的。

## 2。阿帕奇蚂蚁

最初， [Make](https://web.archive.org/web/20220703135935/https://www.gnu.org/software/make/) 是除了本土解决方案**之外唯一可用的构建自动化工具**。 Make 自 1976 年问世以来，在早期的 Java 时代，它被用于构建 Java 应用程序。

然而，C 程序的许多惯例并不适合 Java 生态系统，所以 Ant 最终成为了一个更好的替代者。

**[Apache Ant](https://web.archive.org/web/20220703135935/https://ant.apache.org/) (“另一个简洁的工具”)是一个 Java 库，用于自动化 Java 应用程序的构建过程**。此外，Ant 可以用于构建非 Java 应用程序。它最初是 Apache Tomcat 代码库的一部分，在 2000 年作为一个独立的项目发布。

在许多方面，Ant 与 Make 非常相似，而且非常简单，因此任何人都可以开始使用它，而无需任何特殊的先决条件。Ant 构建文件是用 XML 编写的，按照惯例，它们被称为`build.xml`。

构建过程的不同阶段被称为“目标”。

下面是一个简单 Java 项目的`build.xml`文件的例子，它带有`HelloWorld`主类:

```java
<project>
    <target name="clean">
        <delete dir="classes" />
    </target>

    <target name="compile" depends="clean">
        <mkdir dir="classes" />
        <javac srcdir="src" destdir="classes" />
    </target>

    <target name="jar" depends="compile">
        <mkdir dir="jar" />
        <jar destfile="jar/HelloWorld.jar" basedir="classes">
            <manifest>
                <attribute name="Main-Class" 
                  value="antExample.HelloWorld" />
            </manifest>
        </jar>
    </target>

    <target name="run" depends="jar">
        <java jar="jar/HelloWorld.jar" fork="true" />
    </target>
</project>
```

这个构建文件定义了四个目标:`clean`、`compile`、`jar`和`run`。例如，我们可以通过运行以下命令来编译代码:

```java
ant compile
```

这将首先触发目标`clean`，它将删除“classes”目录。之后，目标`compile`将重新创建目录并将 src 文件夹编译到其中。

Ant 的主要好处是它的灵活性。Ant 没有强加任何编码约定或项目结构。因此，这意味着 Ant 要求开发人员自己编写所有命令，这有时会导致难以维护的巨大 XML 构建文件。

因为没有约定，仅仅知道 Ant 并不意味着我们会很快理解任何 Ant 构建文件。适应一个不熟悉的 Ant 文件可能需要一些时间，这与其他较新的工具相比是一个缺点。

起初，Ant 没有对依赖管理的内置支持。然而，随着依赖管理在后来几年成为必须，Apache Ivy 作为 Apache Ant 项目的子项目被开发出来。它与 Apache Ant 集成在一起，并且遵循相同的设计原则。

然而，由于没有对依赖管理的内置支持，以及在处理不可管理的 XML 构建文件时遇到的挫折，Ant 最初的局限性导致了 Maven 的产生。

## 3。阿巴契人的胃

Apache Maven 是一个依赖管理和构建自动化工具，主要用于 Java 应用程序。 **Maven 继续像 Ant 一样使用 XML 文件，但是以一种更易于管理的方式。**这里的游戏名称是约定胜于配置。

虽然 Ant 提供了灵活性，并要求一切从头开始编写，但 Maven 依赖于约定，并提供预定义的命令(目标)。

简而言之，Maven 允许我们专注于我们的构建应该做什么，并为我们提供了这样做的框架。Maven 的另一个积极方面是它提供了对依赖管理的内置支持。

Maven 的配置文件包含构建和依赖管理指令，按照惯例称为`pom.xml`。此外，Maven 还规定了严格的项目结构，而 Ant 也提供了灵活性。

下面是一个简单 Java 项目的`pom.xml`文件的例子，它带有前面的`HelloWorld`主类:

```java
<project  
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 
      http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <groupId>baeldung</groupId>
    <artifactId>mavenExample</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <description>Maven example</description>

    <dependencies>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.12</version>
            <scope>test</scope>
        </dependency>
    </dependencies>
</project>
```

然而，现在项目结构也已经标准化，并且符合 Maven 约定:

```java
+---src
|   +---main
|   |   +---java
|   |   |   \---com
|   |   |       \---baeldung
|   |   |           \---maven
|   |   |                   HelloWorld.java
|   |   |                   
|   |   \---resources
|   \---test
|       +---java
|       \---resources
```

与 Ant 相反，不需要手动定义构建过程中的每个阶段。相反，我们可以简单地调用 Maven 的内置命令。

例如，我们可以通过运行以下命令来编译代码:

```java
mvn compile
```

正如官方页面所指出的，Maven 的核心可以被认为是一个插件执行框架，因为所有的工作都是由插件完成的。 Maven 支持广泛的[可用插件](https://web.archive.org/web/20220703135935/https://maven.apache.org/plugins/)，每个插件都可以额外配置。

其中一个可用的插件是 Apache Maven 依赖插件，它有一个`copy-dependencies`目标，将我们的依赖项复制到一个指定的目录。

为了展示这个插件的运行，让我们将这个插件包含在我们的`pom.xml`文件中，并为我们的依赖项配置一个输出目录:

```java
<build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-dependency-plugin</artifactId>
            <executions>
                <execution>
                    <id>copy-dependencies</id>
                    <phase>package</phase>
                    <goals>
                        <goal>copy-dependencies</goal>
                    </goals>
                    <configuration>
                        <outputDirectory>target/dependencies
                          </outputDirectory>
                    </configuration>
                </execution>
            </executions>
        </plugin>
    </plugins>
</build>
```

这个插件将在`package` 阶段执行，所以如果我们运行:

```java
mvn package
```

我们将执行这个插件，并将依赖项复制到目标/依赖项文件夹。

还有一篇关于如何使用不同的 Maven 插件创建可执行 JAR 的文章。此外，关于 Maven 的详细概述，请看一下[这篇关于 Maven](/web/20220703135935/https://www.baeldung.com/maven) 的核心指南，其中探讨了一些 Maven 的关键特性。

Maven 变得非常受欢迎，因为构建文件现在已经标准化，与 Ant 相比，维护构建文件花费的时间要少得多。然而，尽管比 Ant 文件更加标准化，Maven 配置文件仍然会变得庞大而笨重。

Maven 的严格约定带来的代价是远不如 Ant 灵活。目标定制非常困难，所以与 Ant 相比，编写定制构建脚本要困难得多。

尽管 Maven 在使应用程序的构建过程更容易和更标准化方面做了一些重要的改进，但由于不如 Ant 灵活，它仍然要付出代价。这导致了 Gradle 的诞生，它结合了两个世界的精华——Ant 的灵活性和 Maven 的特性。

## 4。度

Gradle 是一个依赖管理和构建自动化工具，**是基于 Ant 和 Maven 的概念构建的。**

关于 Gradle，我们首先要注意的一点是，它不像 Ant 或 Maven 那样使用 XML 文件。

随着时间的推移，开发人员对拥有和使用特定领域语言越来越感兴趣——简而言之，这将允许他们使用为特定领域定制的语言来解决特定领域中的问题。

这是由 Gradle 采用的，它使用基于 [Groovy](https://web.archive.org/web/20220703135935/http://groovy-lang.org/) 或 [Kotlin](https://web.archive.org/web/20220703135935/https://kotlinlang.org/) 的 DSL。这导致了更小的配置文件和更少的混乱，因为这种语言是专门为解决特定领域的问题而设计的。按照惯例，Gradle 的配置文件在 Groovy 中称为`build.gradle`，在 Kotlin 中称为`build.gradle.kts`。

注意，对于自动完成和错误检测，Kotlin 提供了比 Groovy 更好的 IDE 支持。

下面是一个简单 Java 项目的`build.gradle`文件的例子，它带有前面的`HelloWorld`主类:

```java
apply plugin: 'java'

repositories {
    mavenCentral()
}

jar {
    baseName = 'gradleExample'
    version = '0.0.1-SNAPSHOT'
}

dependencies {
    testImplementation 'junit:junit:4.12'
}
```

我们可以通过运行以下命令来编译代码:

```java
gradle classes
```

在其核心，Gradle 故意提供很少的功能。**插件增加了所有有用的功能。**在我们的例子中，我们使用了`java`插件，它允许我们编译 Java 代码和其他有价值的特性。

Gradle 将其构建步骤命名为“任务”，而不是 Ant“目标”或 Maven 的“阶段”。对于 Maven，我们使用 Apache Maven 依赖插件，将依赖项复制到指定的目录是一个特定的目标。有了 Gradle，我们可以通过使用任务来完成同样的任务:

```java
task copyDependencies(type: Copy) {
   from configurations.compile
   into 'dependencies'
}
```

我们可以通过执行以下命令来运行该任务:

```java
gradle copyDependencies
```

## 5。结论

在本文中，我们介绍了 Ant、Maven 和 Gradle——三种 Java 构建自动化工具。

毫不奇怪，Maven 占据了今天构建工具市场的大部分份额。

然而，Gradle 在更复杂的代码库中有很好的应用，原因如下:

*   许多开源项目，比如 Spring，现在都在使用它
*   由于它的增量构建，在大多数情况下，它比 Maven 更快
*   它提供[高级分析和调试服务](https://web.archive.org/web/20220703135935/https://scans.gradle.com/)

然而，Gradle 似乎有更陡峭的学习曲线，尤其是如果你不熟悉 Groovy 或 Kotlin 的话。

Next **»**[Writing Custom Gradle Plugins](/web/20220703135935/https://www.baeldung.com/gradle-create-plugin)**«** Previous[Introduction to Gradle](/web/20220703135935/https://www.baeldung.com/gradle)