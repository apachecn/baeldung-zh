# 在 Maven 中设置 Java 版本

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/maven-java-version>

## 1。概述

在这个快速教程中，我们将展示如何**在 Maven 中设置 Java 版本。**

在继续之前，我们可以**检查 Maven 的默认 JDK 版本。**运行`mvn -v`命令将显示 Maven 运行的 Java 版本。

## 延伸阅读:

## [Maven 简介指南](/web/20220722223307/https://www.baeldung.com/maven-profiles)

Learn how to work with Maven profiles to be able to create different build configurations.[Read more](/web/20220722223307/https://www.baeldung.com/maven-profiles) →

## [Maven 编译器插件](/web/20220722223307/https://www.baeldung.com/maven-compiler-plugin)

Learn how to use the Maven compiler plugin, used to compile the source code of a Maven project.[Read more](/web/20220722223307/https://www.baeldung.com/maven-compiler-plugin) →

## 2。使用编译器插件

我们可以在[编译器插件](/web/20220722223307/https://www.baeldung.com/maven-compiler-plugin)中指定想要的 Java 版本。

### 2.1。编译器插件

第一个选项是在编译器插件属性中设置版本:

```java
<properties>
    <maven.compiler.target>1.8</maven.compiler.target>
    <maven.compiler.source>1.8</maven.compiler.source>
</properties>
```

Maven 编译器接受–`target`和–`source `版本的这个命令。如果我们想使用 Java 8 的语言特性，应该将–`source`设置为`1.8`。

此外，为了使编译后的类与 JVM 1.8 兼容，值–`target`应该是`1.8`。

两者的默认值都是 1.6 版本。

或者，我们可以直接配置编译器插件:

```java
<plugins>
    <plugin>    
        <artifactId>maven-compiler-plugin</artifactId>
        <configuration>
            <source>1.8</source>
            <target>1.8</target>
        </configuration>
    </plugin>
</plugins>
```

除了`-source`和`-target`版本之外，`maven-compiler-plugin`还有额外的配置属性，允许我们对编译过程有更多的控制。

### 2.2。Java 9 及更高版本

此外，**从 JDK 9 版本开始，我们可以使用新的`-release`命令行选项。**这个新的参数将自动配置编译器产生类文件，这些类文件将链接到给定平台版本的实现。

**默认情况下，`-source`和`-target`选项不保证交叉编译。**

这意味着我们不能在旧版本的平台上运行我们的应用程序。此外，为了编译和运行旧 Java 版本的程序，我们还需要指定`-bootclasspath`选项。

**为了正确地交叉编译，新的`-release`选项替换了三个标志:`-source,` `-target and -bootclasspath`。**

转换我们的示例后，我们可以为插件属性声明以下内容:

```java
<properties>
    <maven.compiler.release>7</maven.compiler.release>
</properties>
```

而对于从 3.6 版本开始的`maven-compiler-plugin`，我们可以这样写:

```java
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-compiler-plugin</artifactId>
    <version>3.8.0</version>
    <configuration>
        <release>7</release>
    </configuration>
</plugin>
```

注意**我们可以在新的`<release>`属性中添加 Java 版本。**在这个例子中，我们为 Java 7 编译我们的应用程序。

此外，我们不需要在我们的机器上安装 JDK 7。Java 9 已经包含了将新语言特性与 JDK 7 联系起来的所有信息。

## 3。Spring Boot 规格

Spring Boot 应用程序在`the pom.xml `文件的`properties`标签中指定了 JDK 版本。

首先，我们需要添加*spring-boot-starter-parent*作为我们项目的父项目:

```java
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
</parent>
```

这个父 POM 允许我们配置默认插件和多个属性，包括 Java 版本——默认情况下，Java 版本是`1.8`。

然而，我们可以通过指定`java.version`属性来覆盖父节点的默认版本:

```java
<properties>
    <java.version>9</java.version>
</properties>
```

**通过设置`java.version`属性，我们声明源和目标 Java 版本都等于`1.9`。**

最重要的是，我们应该记住这个属性是一个 Spring Boot 规范。另外，**从 Spring Boot 2.0 开始，Java 8 是最低版本。**

这意味着我们不能为旧的 JDK 版本使用或配置 Spring Boot。

## 4。结论

这个快速教程演示了在我们的 Maven 项目中设置 Java 版本的可能方法。

以下是主要要点的总结:

*   **只有在 Spring Boot 应用程序中才能使用`<java.version>`。**
*   对于简单的情况，`maven.compiler.source `和`maven.compiler.target `属性应该是最合适的。
*   **最后，为了更好地控制编译过程，使用`maven-compiler-plugin` 配置设置。**