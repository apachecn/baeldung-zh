# Eclipse STS 中的 Spring 指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/eclipse-sts-spring>

## 1。概述

本文展示了[Eclipse Spring Tool Suite(STS)](https://web.archive.org/web/20220815032147/https://spring.io/guides/gs/sts/)IDE 的一些有用特性，这些特性在开发 [Spring 应用程序](https://web.archive.org/web/20220815032147/https://docs.spring.io/spring/docs/current/spring-framework-reference/htmlsingle/)时非常有用。

首先，我们展示了与使用 Eclipse 构建应用程序的传统方式相比，使用 STS 的好处。

此后，我们将重点关注如何引导应用程序、如何运行它以及如何添加额外的依赖项。最后，我们通过添加应用程序参数来结束。

## 2。STS 主要特性

STS 是一个基于 Eclipse 的开发环境，是为 Spring 应用程序的开发定制的。

它提供了一个现成的环境来实现、调试、运行和部署您的应用程序。它还包括 Pivotal tc Server、Pivotal Cloud Foundry、Git、Maven 和 AspectJ 的集成。STS 是在最新的 Eclipse 版本基础上构建的。

### 2.1。项目配置

STS 了解几乎所有最常见的 Java 项目结构。它解析配置文件，然后显示关于已定义的[bean](https://web.archive.org/web/20220815032147/https://docs.spring.io/spring/docs/current/spring-framework-reference/htmlsingle/#beans-introduction)的详细信息、依赖性、使用的名称空间，此外还提取某些原型的概述。

[![spring-bean-snapshot](img/1b706c67825f578a5d8b1eed8a8a352e.png)](/web/20220815032147/https://www.baeldung.com/wp-content/uploads/2016/07/spring-bean-snapshot.png)

### 2.2。STS 功能概述

Eclipse STS 验证您的项目，并为您的应用程序提供快速修复。例如，当使用 Spring Data JPA 时，IDE 可以用来验证查询方法名(在第 6 节中有更多相关内容)。

STS 还提供了所有 bean 方法及其相互关系的图形视图。您可能希望通过查看菜单`window`、`show view` 和`Spring`下的视图，更仔细地了解 STS 附带的图形编辑器。

STS 还提供了其他有用的特性，不仅仅局限于 Spring 应用程序。建议读者查看完整的特性列表，可以在[这里](https://web.archive.org/web/20220815032147/https://spring.io/guides/gs/sts/)找到。

## 3。创建一个 Spring 应用程序

让我们从引导一个简单的应用程序开始。如果没有 STS，Spring 应用程序通常是通过使用 [Spring Initializer](https://web.archive.org/web/20220815032147/https://start.spring.io/) 网站或者 [Spring Boot CLI](https://web.archive.org/web/20220815032147/https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#cli-init) 创建的。这可以通过点击 STS 中仪表板上的`Create Spring Starter Project` 来简化。

在`New Spring Starter Project`屏幕中，使用默认值或进行自己的调整，然后进入下一个屏幕。选择`Web` 并点击完成。您的`pom.xml`现在应该看起来像这样:

```
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.4.0</version>
    <relativePath/> <!-- lookup parent from repository -->
</parent>

<properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <java.version>1.8</java.version>
</properties>

<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>

    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
    </dependency>
</dependencies>
```

你的 Spring Boot 版本可能有所不同，但你总能在这里找到最新版本。

## 4。运行应用程序

可以通过右键单击项目并选择 run as `Spring Boot App`来启动上述应用程序。如果没有 STS，您最有可能使用以下命令从命令行运行应用程序:

```
$ mvn spring-boot:run
```

默认情况下，Spring 应用程序是由运行在端口 8080 上的 Tomcat 启动的。此时，应用程序在端口 8080 上启动，基本上不做任何其他事情，因为我们还没有实现任何代码。第 8 节向您展示了如何更改默认端口。

## 5。日志和 ANSI 控制台

当您使用 run 命令从 IDE 运行项目时，您会注意到控制台输出了一些漂亮的[颜色编码的](https://web.archive.org/web/20220815032147/https://docs.spring.io/spring-boot/docs/current/reference/html/boot-features-logging.html#boot-features-logging-color-coded-output)日志语句。如果您想将其关闭，请转到`run configurations` …并禁用`Spring Boot`选项卡上的复选框`Enable ANSI console output`。或者，您也可以通过在`application.properties` 文件中设置一个属性值来禁用它。

```
spring.output.ansi.enabled=NEVER
```

关于应用程序日志配置的更多信息可以在[这里](https://web.archive.org/web/20220815032147/https://docs.spring.io/spring-boot/docs/current/reference/html/howto-logging.html)找到。

## 6。JPA 查询名称检查

有时，实现数据访问层可能是一项繁琐的活动。为了实现简单的查询和执行分页，可能需要编写大量的样板代码。Spring Data JPA(JPA)旨在极大地促进数据访问层的实现。本节说明了将 JPA 与 STS 结合使用的一些好处。

首先，将 JPA 的以下依赖项添加到之前生成的`pom.xml`:

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
<dependency>
    <groupId>com.h2database</groupId>
    <artifactId>h2</artifactId>
</dependency>
```

你可能已经注意到在上面的声明中没有指定`version`。这是因为依赖关系是由`spring-boot-starter-parent`管理的。

为了让 JPA 工作，需要正确定义实体管理器和事务管理器。然而，Spring 会自动为您配置这些。留给开发人员的唯一事情是创建实际的实体类。这些实体由实体管理器管理，而实体管理器又由容器创建。例如，让我们像这样创建一个实体类`Foo`:

```
@Entity
public class Foo implements Serializable {
    @Id
    @GeneratedValue
    private Integer id;
    private String name;

    // Standard getters and setters
}
```

容器从配置包的根扫描所有用`@Entity`标注的类。接下来，我们为`Foo`实体创建一个 JPA 存储库:

```
public interface FooRepository extends JpaRepository<Foo, Integer> {
    public Foo findByNames(String name);
}
```

此时，您可能已经注意到，IDE 现在用一个异常标记此查询方法:

```
Invalid derived query! No property names found for type Foo! 
```

这当然是因为我们不小心在 JPA 存储库的方法名中写了一个‘s’。要解决这个问题，请像这样移除乱真的:

```
public Foo findByName(String name);
```

注意，config 类中没有使用`@EnableJpaRepositories`。这是因为容器的`AutoConfigration`为项目预先注册了一个。

## 7。罐子类型搜索

“Jar 类型搜索”是在 [STS 3.5.0](https://web.archive.org/web/20220815032147/https://docs.spring.io/sts/nan/v350/NewAndNoteworthy.html) 中引入的一个特性。它在项目中为(还)不在类路径上的类提供了内容辅助建议。STS 可以帮助您将依赖项添加到 POM 文件中，以防它们还不在类路径中。

例如，让我们向`Foo`实体类添加一行。为了让这个例子正常工作，请首先确保 `java.util.List`的 import 语句已经存在。现在我们可以添加谷歌番石榴如下:

```
private List<String> strings = Lists // ctrl + SPACE to get code completion
```

IDE 会建议将几个依赖项添加到类路径中。从`com.google.common.collect,`添加依赖关系按回车键，从`Guava`添加依赖关系。番石榴罐现在会自动添加到您的`pom.xml` 文件中，如下所示:

```
<dependency>
    <groupId>com.google.guava</groupId>
    <artifactId>guava</artifactId>
    <version>31.0.1-jre</version>
</dependency> 
```

从版本 [STS 3.8.0](https://web.archive.org/web/20220815032147/https://docs.spring.io/sts/nan/v380/NewAndNoteworthy.html) 开始，在 STS 对您的`pom.xml.`进行更改之前，您会得到一个确认对话框

## 8。添加应用程序参数

Spring 的另一个强大功能是支持外部配置，这些配置可以通过多种方式传递给应用程序，例如作为命令行参数，在属性或 YAML 文件中指定，或者作为系统属性。在本节中，我们将重点介绍如何使用 STS 添加一个配置选项作为应用程序启动参数。这可以通过配置 Tomcat 在不同的端口上启动来说明。

为了在默认端口之外的 Tomcat 端口上运行应用程序，您可以使用下面的命令，其中自定义端口被指定为命令行参数:

```
mvn spring-boot:run -Drun.arguments="--server.port=7070"
```

使用 STS 时，您必须进入`run`菜单。从运行配置对话框中选择`run configurations` …从左侧面板中选择`Spring Boot App`，然后选择`demo – DemoApplication`(如果您没有选择默认项目，这将会有所不同)。从`Program Arguments`窗口中的`(x)= Arguments`选项卡键入

```
--server.port=7070
```

还有`run`。您应该会在控制台中看到类似如下所示的输出:

```
.
.
2016-07-06 13:51:40.999  INFO 8724 --- [           main] s.b.c.e.t.TomcatEmbeddedServletContainer : Tomcat started on port(s): 7070 (http)
2016-07-06 13:51:41.006  INFO 8724 --- [           main] com.baeldung.boot.DemoApplication        : Started DemoApplication in 6.245 seconds (JVM running for 7.34)
```

## 9。结论

在本文中，我们展示了在 STS 中开发 Spring 项目的基础。我们展示的一些内容包括 STS 中应用程序的执行、Spring Data JPA 开发期间的支持以及命令行参数的使用。然而，由于 STS 提供了一组丰富的特性，所以在开发过程中可能会用到许多更有用的特性。

本文的**完整实现**可以在 [github 项目](https://web.archive.org/web/20220815032147/https://github.com/eugenp/tutorials/tree/master/spring-boot-modules/spring-boot-mvc-4)中找到——这是一个基于 Eclipse 的项目，所以它应该很容易导入和运行。