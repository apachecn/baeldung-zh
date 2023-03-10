# 使用 Maven 与可执行 War/Jar 运行 Spring Boot 应用程序

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-boot-run-maven-vs-executable-jar>

## 1.介绍

在本教程中，我们将探索通过 *mvn spring-boot:run* 命令启动 Spring Boot web 应用程序和通过`java -jar`命令将它编译成 jar/war 包后运行它之间的区别。

出于本教程的目的，我们假设熟悉 Spring Boot `repackage`目标的配置。关于这个话题的更多细节，请阅读[与 Spring Boot](/web/20221206051807/https://www.baeldung.com/deployable-fat-jar-spring-boot) 一起创建一个胖罐子应用程序。

## 2.Spring Boot Maven 插件

当编写 Spring Boot 应用程序时， [Spring Boot Maven 插件](https://web.archive.org/web/20221206051807/https://docs.spring.io/spring-boot/docs/current/maven-plugin/reference/htmlsingle/)是构建、测试和打包我们代码的推荐工具。

这个插件附带了许多方便的特性，例如:

*   它为我们解析了正确的依赖版本
*   它可以将我们所有的依赖项(包括嵌入式应用服务器，如果需要的话)打包到一个单独的、可运行的 fat jar/war 中，并且还将:
    *   为我们管理类路径配置，这样我们就可以跳过`java -jar`命令中的长`-cp`选项
    *   实现一个定制的`ClassLoader`来定位和加载现在嵌套在包中的所有外部 jar 库
    *   自动找到`main()`方法并在清单中配置它，这样我们就不必在`java -jar` 命令中指定主类

## 3.用 Maven 以分解的形式运行代码

当我们开发 web 应用程序时，我们可以利用`Spring Boot Maven plugin:`的另一个非常有趣的特性，即在嵌入式应用服务器中自动部署 web 应用程序的能力。

我们只需要一个依赖项来让插件知道我们想要使用 Tomcat 来运行我们的代码:

```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId> 
</dependency>
```

现在，当在我们的项目根文件夹中执行`mvn spring-boot:run`命令时，插件读取 pom 配置并理解我们需要一个 web 应用程序容器。

**执行`mvn spring-boot:run`命令触发 Apache Tomcat 的下载，并初始化 Tomcat 的启动:**

```java
$ mvn spring-boot:run
...
...
[INFO] --------------------< com.baeldung:spring-boot-ops >--------------------
[INFO] Building spring-boot-ops 0.0.1-SNAPSHOT
[INFO] --------------------------------[ war ]---------------------------------
[INFO]
[INFO] >>> spring-boot-maven-plugin:2.1.3.RELEASE:run (default-cli) > test-compile @ spring-boot-ops >>>
Downloading from central: https://repo.maven.apache.org/maven2/org/apache/tomcat/embed/tomcat-embed-core/9.0.16/tomcat-embed-core-9.0.16.pom
Downloaded from central: https://repo.maven.apache.org/maven2/org/apache/tomcat/embed/tomcat-embed-core/9.0.16/tomcat-embed-core-9.0.16.pom (1.8 kB at 2.8 kB/s)
...
...
[INFO] --- spring-boot-maven-plugin:2.1.3.RELEASE:run (default-cli) @ spring-boot-ops ---
...
...
11:33:36.648 [main] INFO  o.a.catalina.core.StandardService - Starting service [Tomcat]
11:33:36.649 [main] INFO  o.a.catalina.core.StandardEngine - Starting Servlet engine: [Apache Tomcat/9.0.16]
...
...
11:33:36.952 [main] INFO  o.a.c.c.C.[Tomcat].[localhost].[/] - Initializing Spring embedded WebApplicationContext
...
...
11:33:48.223 [main] INFO  o.a.coyote.http11.Http11NioProtocol - Starting ProtocolHandler ["http-nio-8080"]
11:33:48.289 [main] INFO  o.s.b.w.e.tomcat.TomcatWebServer - Tomcat started on port(s): 8080 (http) with context path ''
11:33:48.292 [main] INFO  org.baeldung.boot.Application - Started Application in 22.454 seconds (JVM running for 37.692)
```

当日志显示包含“Started Application”的行时，我们的 web 应用程序就可以通过位于 http://localhost:8080/的浏览器进行查询了

## 4.将代码作为独立的打包应用程序运行

一旦我们通过了开发阶段，并逐步将我们的应用程序投入生产，我们就需要打包我们的应用程序。

不幸的是，如果我们使用一个`jar`包，基本的 Maven `package`目标不包括任何外部依赖。这意味着我们只能在更大的项目中把它作为一个库来使用。

**为了规避这个限制，** **我们需要利用 Maven Spring Boot 插件 `repackage`的目标来运行我们的 jar/war 作为一个独立的应用程序。**

### 4.1.配置

通常，我们只需要配置构建插件:

```java
<build>
    <plugins>
        ...
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
        </plugin>
        ...
    </plugins>
</build>
```

由于我们的示例项目包含不止一个主类，我们必须告诉 Java 运行哪个类，或者通过配置插件:

```java
<plugin>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-maven-plugin</artifactId>
    <executions>
        <execution>
            <configuration>
                <mainClass>com.baeldung.webjar.WebjarsdemoApplication</mainClass>
            </configuration>
        </execution>
    </executions>
</plugin>
```

或者设置`start-class`属性:

```java
<properties>
    <start-class>com.baeldung.webjar.WebjarsdemoApplication</start-class>
</properties>
```

### 4.2.运行应用程序

现在，我们可以用两个简单的命令运行示例 war:

```java
$ mvn clean package spring-boot:repackage
$ java -jar target/spring-boot-ops.war
```

关于如何运行 jar 文件的更多细节可以在我们的文章[使用命令行参数运行 JAR 应用程序](/web/20221206051807/https://www.baeldung.com/java-run-jar-with-arguments)中找到。

### 4.3.在战争文件里

**为了更好地理解上面提到的命令如何运行一个完整的服务器应用程序，我们可以看一下我们的`spring-boot-ops.war`。**

如果我们解压缩并窥视内部，我们会发现通常的疑点:

*   `META-INF`，带有自动生成的`MANIFEST.MF`
*   `WEB-INF/classes`，包含我们编译的类
*   `WEB-INF/lib`，它保存了我们的 war 依赖项和嵌入的 Tomcat jar 文件

这还不是全部，因为有一些特定于我们的 fat 包配置的文件夹:

*   `WEB-INF/lib-provided`，包含运行嵌入式时需要的外部库，但部署时不需要
*   `org/springframework/boot/loader`，它保存了 Spring Boot 定制的类加载器。这个库负责加载我们的外部依赖项，并使它们在运行时可访问。

### 4.4.在战争清单里面

如前所述，Maven Spring Boot 插件找到主类并生成运行`java`命令所需的配置。

产生的`MANIFEST.MF`有一些额外的行:

```java
Start-Class: com.baeldung.webjar.WebjarsdemoApplication
Main-Class: org.springframework.boot.loader.WarLauncher
```

特别是，我们可以观察到最后一个指定了要使用的 Spring Boot 类加载器。

### 4.5.在 Jar 文件中

由于默认的打包策略，无论我们是否使用`Spring Boot Maven Plugin`，我们的`war packaging`场景并没有太大的不同。

为了更好地理解插件的优点，我们可以尝试将 pom `packaging`配置更改为`jar,`并再次运行`mvn clean package`。

我们现在可以观察到，我们的 fat jar 与之前的 war 文件有一点不同:

*   我们所有的类和资源文件夹现在都位于`BOOT-INF/classes.`下
*   保存所有的外部库。

**如果没有这个插件，`lib`文件夹就不会存在，`BOOT-INF/classes`的所有内容都会位于这个包的根目录下。**

### 4.6.在罐子清单里面

`MANIFEST.` MF 也有所改变，增加了以下几条线:

```java
Spring-Boot-Classes: BOOT-INF/classes/
Spring-Boot-Lib: BOOT-INF/lib/
Spring-Boot-Version: 2.1.3.RELEASE
Main-Class: org.springframework.boot.loader.JarLauncher
```

`Spring-Boot-Classes`和`Spring-Boot-Lib`特别有趣，因为它们告诉我们类装入器将在哪里找到类和外部库。

## 5.如何选择

当分析工具时，我们必须考虑这些工具被创造出来的目的。我们是想简化开发，还是想确保顺利部署和可移植性？让我们来看看受此选择影响最大的阶段。

### 5.1.发展

作为开发人员，我们经常把大部分时间花在编码上，而不需要花很多时间来设置我们的环境以在本地运行代码。在简单的应用程序中，这通常不是问题。但是对于更复杂的项目，我们可能需要设置环境变量、启动服务器和填充数据库。

**每次我们想要运行应用程序时都配置正确的环境是非常不切实际的**，尤其是在必须同时运行多个服务的情况下。

这就是用 Maven 运行代码帮助我们的地方。我们已经在本地签出了整个代码库，所以我们可以利用 pom 配置和资源文件。我们可以设置环境变量，生成内存数据库，甚至下载正确的服务器版本，并使用一个命令部署我们的应用程序。

即使在多模块代码库中，每个模块需要不同的变量和服务器版本，我们也可以通过 Maven 概要文件轻松运行正确的环境。

### 5.2.生产

我们越接近生产，话题就越转向稳定性和安全性。这就是为什么我们不能将用于我们的开发机器的过程应用到有真实客户的服务器上。

**在这个阶段通过 Maven 运行代码是很糟糕的做法，原因有很多:**

*   首先，我们需要安装 Maven。
*   然后，仅仅因为我们需要编译代码，我们需要完整的 Java 开发工具包(JDK)。
*   接下来，我们必须将代码库复制到我们的服务器上，将我们所有的专有代码以纯文本的形式保存下来。
*   `mvn`命令必须执行生命周期的所有阶段(查找源代码、编译和运行)。
*   由于前一点，我们也浪费了 CPU，在云服务器的情况下，浪费了金钱。
*   Maven 产生多个 Java 进程，每个进程都使用内存(默认情况下，它们都使用与父进程相同的内存量)。
*   最后，如果我们有多台服务器要部署，那么在每台服务器上重复上述所有步骤。

这些只是**将应用作为一个包交付给生产**更实际的几个原因。

## 6.结论

在本文中，我们探讨了通过 Maven 和通过`java -jar`命令运行代码的区别。我们还快速浏览了一些实际案例场景。

本文中使用的源代码可以从 GitHub 上的[处获得。](https://web.archive.org/web/20221206051807/https://github.com/eugenp/tutorials/tree/master/spring-boot-modules/spring-boot-artifacts)