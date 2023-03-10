# Maven 依赖范围

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/maven-dependency-scopes>

## 1。概述

Maven 是 Java 生态系统中最流行的构建工具之一，它的核心特性之一是依赖管理。

在本教程中，我们将描述和探索有助于管理 Maven 项目中可传递依赖关系的机制——依赖关系范围。

## 2。传递依赖性

Maven 中有两种类型的依赖:直接依赖和传递依赖。

直接依赖是我们明确包含在项目中的依赖。

这些可以使用`<dependency>`标签来包含:

```java
<dependency>
    <groupId>junit</groupId>
    <artifactId>junit</artifactId>
    <version>4.12</version>
</dependency>
```

**另一方面，传递依赖是直接依赖所要求的。Maven 会自动在我们的项目中包含所需的传递依赖。**

我们可以使用`mvn dependency:tree`命令列出项目中所有的依赖项，包括传递依赖项。

## 3。依赖范围

依赖范围有助于限制依赖的传递性。他们还为不同的构建任务修改类路径。Maven 有六个默认的依赖范围。

重要的是要理解每个作用域——除了`import`——都对传递依赖有影响。

### 3.1。编译

没有提供其他范围时，这是默认范围。

在所有构建任务中，具有此范围的依赖项都可以在项目的类路径中找到。它们还会传播到相关项目。

更重要的是，这些依赖关系也是可传递的:

```java
<dependency>
    <groupId>commons-lang</groupId>
    <artifactId>commons-lang</artifactId>
    <version>2.6</version>
</dependency>
```

### 3.2。提供

我们使用这个作用域来标记应该由 JDK 或容器在运行时提供的**依赖。**

这个范围的一个很好的用例是部署在某个容器中的 web 应用程序，该容器本身已经提供了一些库。例如，这可能是一个已经在运行时提供 Servlet API 的 web 服务器。

在我们的项目中，我们可以用`provided`范围定义这些依赖关系:

```java
<dependency>
    <groupId>javax.servlet</groupId>
    <artifactId>javax.servlet-api</artifactId>
    <version>4.0.1</version>
    <scope>provided</scope>
</dependency>
```

`provided` 依赖项仅在编译时和项目的测试类路径中可用。这些依赖关系也是不可传递的。

### 3.3。运行时间

**运行时需要此范围的依赖关系。**但是我们不需要它们来编译项目代码。因此，用`runtime`范围标记的依赖项将出现在运行时和测试类路径中，但是它们将从编译类路径中消失。

JDBC 驱动程序是应该使用运行时范围的依赖关系的一个很好的例子:

```java
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <version>8.0.28</version>
    <scope>runtime</scope>
</dependency>
```

### 3.4。测试

我们使用这个范围来表示在应用程序的标准运行时不需要依赖，而只是用于测试目的。

依赖关系是不可传递的，只存在于测试和执行类路径中。

此范围的标准用例是向我们的应用程序添加一个测试库，如 JUnit:

```java
<dependency>
    <groupId>junit</groupId>
    <artifactId>junit</artifactId>
    <version>4.12</version>
    <scope>test</scope>
</dependency>
```

### 3.5。系统

**`System`作用域与`provided`作用域非常相似。**主要的区别是`system`要求我们直接指向系统上的一个特定的 jar。

值得一提的是 **[`system`作用域已弃用](https://web.archive.org/web/20221116075508/https://maven.apache.org/guides/introduction/introduction-to-dependency-mechanism.html#system-dependencies)。**

需要记住的重要一点是，如果依赖项不存在或者位于不同于`systemPath`所指向的位置，那么在不同的机器上构建具有`system`范围依赖项的项目可能会失败:

```java
<dependency>
    <groupId>com.baeldung</groupId>
    <artifactId>custom-dependency</artifactId>
    <version>1.3.2</version>
    <scope>system</scope>
    <systemPath>${project.basedir}/libs/custom-dependency-1.3.2.jar</systemPath>
</dependency>
```

### 3.6。导入

**仅对依赖类型`pom`有效。**

`import`表示该依赖关系应该被其 POM 中声明的所有有效依赖关系替换。

这里，下面的`custom-project`依赖项将被替换为自定义项目的`pom.xml` `<dependencyManagement>`部分中声明的所有依赖项。

```java
<dependency>
    <groupId>com.baeldung</groupId>
    <artifactId>custom-project</artifactId>
    <version>1.3.2</version>
    <type>pom</type>
    <scope>import</scope>
</dependency>
```

## 4。范围和传递性

每个依赖范围都以自己的方式影响传递性依赖。这意味着不同的可传递依赖项可能以不同的范围在项目中结束。

然而，范围为`provided`和`test` 的依赖项永远不会包含在主项目中。

让我们详细看看这意味着什么:

*   对于`compile`作用域，所有与`runtime`作用域的依赖关系将被拉入项目中的`runtime`作用域，所有与`compile` 作用域的依赖关系将被拉入项目中的`compile`作用域。
*   对于`provided`范围，项目中的`runtime` 和`compile`范围依赖项将与`provided`范围一起被拉入。
*   对于`test`范围，`runtime` 和`compile` 范围传递依赖项都将与项目中的`test` 范围一起引入。
*   对于`runtime`范围，`runtime` 和`compile`范围传递依赖项都将与项目中的`runtime` 范围一起引入。

## 5。结论

在这篇简短的文章中，我们重点关注了 Maven 依赖范围、它们的目的以及它们如何操作的细节。

要更深入地研究 Maven，文档是一个很好的起点。