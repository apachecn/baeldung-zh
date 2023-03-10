# 覆盖 Spring Boot 托管相关性版本

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-boot-override-dependency-versions>

## 1.介绍

Spring Boot 是快速启动新项目的绝佳框架。它帮助开发人员快速创建新应用程序的方法之一是定义一组适合大多数用户的依赖项。

然而，在某些情况下，**可能需要覆盖一个或多个依赖版本**。

在本教程中，我们将看看如何覆盖 Spring Boot 托管依赖项及其版本。

## 2.Spring Boot 物料清单

让我们先看看 Spring Boot 是如何管理依赖关系的。简而言之，Spring Boot 使用一个[物料清单(BOM)](/web/20220727020730/https://www.baeldung.com/spring-maven-bom) 来定义依赖关系和版本。

大多数 Spring Boot 项目继承自 [spring-boot-starter-parent 工件](https://web.archive.org/web/20220727020730/https://search.maven.org/artifact/org.springframework.boot/spring-boot-starter-parent)，后者本身继承自 [spring-boot-dependencies](https://web.archive.org/web/20220727020730/https://search.maven.org/search?q=g:org.springframework.boot%20AND%20a:spring-boot-dependencies) 工件。**后一个工件是 Spring Boot BOM** ，它只是一个 Maven POM 文件，有一个很大的`dependencyManagement`部分:

```java
<dependencyManagement>
    <dependencies>
        <dependency>
            ...
        </dependency>
        <dependency>
            ...
        </dependency>
    </dependencies>
</dependencyManagement>
```

通过使用 Maven 的`dependencyManagement`，**，BOM 可以指定默认的库版本，如果我们的应用程序选择使用它们的话**。让我们看一个例子。

Spring Boot BOM 中的一个条目如下:

```java
<dependency>
    <groupId>org.apache.activemq</groupId>
    <artifactId>activemq-amqp</artifactId>
    <version>${activemq.version}</version>
</dependency>
```

这意味着项目中依赖于 ActiveMQ 的任何工件都将默认获得这个版本。

另外，注意**版本是使用属性占位符**指定的。这是 Spring Boot BOM 中的一种常见做法，它在自己的`properties`部分中提供了该属性和其他属性的值。

## 3.覆盖 Spring Boot 托管相关性版本

现在我们已经了解了 Spring Boot 如何管理依赖版本，让我们看看我们可以覆盖它们。

### 3.1.专家

对于 Maven，我们有两个选项来覆盖 Spring Boot 托管的依赖项。首先，对于 Spring Boot BOM 用属性占位符**指定版本的任何依赖项，我们只需要在我们的项目 POM** 中设置该属性:

```java
<properties>
    <activemq.version>5.16.3</activemq.version>
</properties>
```

这将导致任何使用`activemq.version`属性的依赖项使用我们指定的版本，而不是 Spring Boot BOM 中的版本。

此外，如果版本是在 BOM 中的`dependency`标记中显式指定的，而不是作为占位符，那么我们可以简单地在项目依赖项中显式覆盖`version`:

```java
<dependency>
    <groupId>org.apache.activemq</groupId>
    <artifactId>activemq-amqp</artifactId>
    <version>5.16.3</version>
</dependency>
```

### 3.2\. Gradle

Gradle 需要一个插件来支持来自 Spring Boot BOM 的依赖管理。因此，要开始，我们必须包含插件并导入 BOM:

```java
apply plugin: "io.spring.dependency-management"
dependencyManagement {
  imports {
    mavenBom 'io.spring.platform:platform-bom:2.5.5'
  }
}
```

现在，如果我们想要覆盖依赖关系的特定版本，我们只需要从 BOM 中指定相应的属性作为 Gradle `ext`属性:

```java
ext['activemq.version'] = '5.16.3'
```

如果 BOM 中没有要覆盖的属性，我们可以在声明依赖关系时直接指定版本:

```java
compile 'org.apache.activemq:activemq-amqp:5.16.3'
```

### 3.3.警告

这里有几个警告值得一提。

首先，重要的是要记住 Spring Boot 是使用 BOM 中指定的库版本构建和测试的。任何时候我们指定不同的库版本，都有引入不兼容性的风险。因此，当我们偏离标准依赖版本时，测试我们的应用程序是至关重要的。

此外，请记住**这些提示仅适用于我们使用 Spring Boot 材料清单(BOM)** 时。对于 Maven 来说，这意味着使用 Spring Boot 父节点。对于 Gradle 来说，这意味着使用 Spring dependencies 插件。

## 4.查找依赖版本

我们已经看到了 Spring Boot 如何管理依赖版本，以及如何覆盖它们。在这一节中，我们将看看如何找到我们的项目正在使用的库的版本。这有助于识别库版本，并确认我们应用于项目的任何覆盖都被执行。

### 4.1.专家

Maven 提供了一个[目标](/web/20220727020730/https://www.baeldung.com/maven-goals-phases)，我们可以用它来显示所有依赖项及其版本的列表。例如，如果我们运行以下命令:

```java
mvn dependency:tree
```

我们应该看到类似于以下内容的输出:

```java
[INFO] com.baeldung:dependency-demo:jar:0.0.1-SNAPSHOT
[INFO] +- org.springframework.boot:spring-boot-starter-web:jar:2.5.7-SNAPSHOT:compile
[INFO] |  +- org.springframework.boot:spring-boot-starter:jar:2.5.7-SNAPSHOT:compile
[INFO] |  |  +- org.springframework.boot:spring-boot:jar:2.5.7-SNAPSHOT:compile
[INFO] |  |  +- org.springframework.boot:spring-boot-autoconfigure:jar:2.5.7-SNAPSHOT:compile
[INFO] |  |  +- org.springframework.boot:spring-boot-starter-logging:jar:2.5.7-SNAPSHOT:compile
[INFO] |  |  |  +- ch.qos.logback:logback-classic:jar:1.2.6:compile
[INFO] |  |  |  |  \- ch.qos.logback:logback-core:jar:1.2.6:compile
```

输出显示了项目依赖的所有工件和版本。这些依赖关系呈现在一个树形结构中，使得识别每个工件是如何导入到项目中变得容易。

在上面的例子中，`logback-classic`工件是`spring-boot-starter-logging`库的一个依赖项，而后者本身是`spring-boot-starter`模块的一个依赖项。因此，我们可以沿着树向上导航回到我们的顶级项目。

### 4.2\. Gradle

Gradle 提供了一个生成类似依赖树的任务。例如，如果我们运行以下命令:

```java
gradle dependencies
```

我们将得到类似于以下内容的输出:

```java
compileClasspath - Compile classpath for source set 'main'.
\--- org.springframework.boot:spring-boot-starter-web -> 1.3.8.RELEASE
     +--- org.springframework.boot:spring-boot-starter:1.3.8.RELEASE
     |    +--- org.springframework.boot:spring-boot:1.3.8.RELEASE
     |    |    +--- org.springframework:spring-core:4.2.8.RELEASE
     |    |    \--- org.springframework:spring-context:4.2.8.RELEASE
     |    |         +--- org.springframework:spring-aop:4.2.8.RELEASE
```

就像 Maven 输出一样，我们可以很容易地识别为什么每个工件都被引入到项目中，以及所使用的版本。

## 5.结论

在文章中，我们已经了解了 Spring Boot 如何管理依赖版本。我们还看到了如何在 Maven 和 Gradle 中覆盖这些依赖版本。最后，我们看到了如何在两种项目类型中验证依赖版本。