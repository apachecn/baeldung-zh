# Spring Boot 面试问题

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-boot-interview-questions>

## 1。概述

自推出以来，Spring Boot 一直是 Spring 生态系统中的关键角色。这个项目通过它的自动配置能力使我们的生活变得更加容易。

在本教程中，我们将讨论一些在求职面试中可能出现的与 Spring Boot 有关的最常见的问题。

## 延伸阅读:

## [Top Spring 框架面试问题](/web/20220904110305/https://www.baeldung.com/spring-interview-questions)

A quick discussion of common questions about the Spring Framework that might come up during a job interview.[Read more](/web/20220904110305/https://www.baeldung.com/spring-interview-questions) →

## [春天和 Spring Boot 的比较](/web/20220904110305/https://www.baeldung.com/spring-vs-spring-boot)

Understand the difference between Spring and Spring Boot.[Read more](/web/20220904110305/https://www.baeldung.com/spring-vs-spring-boot) →

## 2。问题

### Q1。什么是 Spring Boot，它的主要特征是什么？

Spring Boot 本质上是一个建立在 Spring 框架之上的快速应用开发框架。凭借其自动配置和嵌入式应用服务器支持，以及广泛的文档和社区支持，Spring Boot 是迄今为止 Java 生态系统中最受欢迎的技术之一。

以下是一些突出的特点:

*   [Starters](/web/20220904110305/https://www.baeldung.com/spring-boot-starters)–一组依赖描述符，一次性包含相关的依赖关系
*   [自动配置](/web/20220904110305/https://www.baeldung.com/spring-boot-annotations#enable-autoconfiguration)——一种根据类路径中存在的依赖关系自动配置应用程序的方法
*   [致动器](/web/20220904110305/https://www.baeldung.com/spring-boot-actuators)–获得监控等生产就绪功能
*   [安全](/web/20220904110305/https://www.baeldung.com/security-spring)
*   [测井](/web/20220904110305/https://www.baeldung.com/spring-boot-logging)

### Q2。春天和 Spring Boot 有什么不同？

Spring 框架提供了多种特性，使得 web 应用程序的开发更加容易。这些特性包括依赖注入、数据绑定、面向方面编程、数据访问等等。

这些年来，Spring 变得越来越复杂，这种应用程序需要的配置数量令人望而生畏。这就是 Spring Boot 派上用场的地方——它使配置 Spring 应用程序变得轻而易举。

本质上，虽然 Spring 没有被个人化，但 Spring Boot 对平台和库有自己的观点，让我们可以快速开始。

以下是 Spring Boot 带来的两个最重要的好处:

*   根据在类路径上找到的工件自动配置应用程序
*   提供生产应用程序常见的非功能性特性，如安全或健康检查

请查看我们的另一个教程，了解香草春天和 Spring Boot 的详细对比。

### Q3。我们如何用 Maven 建立一个 Spring Boot 应用程序？

我们可以像对待任何其他库一样，将 Spring Boot 包含在 Maven 项目中。然而，最好的方法是从`spring-boot-starter-parent`项目继承，并声明对 [Spring Boot 启动器](/web/20220904110305/https://www.baeldung.com/spring-boot-starters)的依赖。这样做可以让我们的项目重用 Spring Boot 的默认设置。

继承`spring-boot-starter-parent`项目很简单——我们只需要在`pom.xml`中指定一个`parent`元素:

```java
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.4.0.RELEASE</version>
</parent>
```

我们可以在 [Maven Central](https://web.archive.org/web/20220904110305/https://search.maven.org/search?q=g:org.springframework.boot%20AND%20a:spring-boot-starter-parent&core=gav) 上找到`spring-boot-starter-parent`的最新版本。

使用起始父项目很方便，但并不总是可行的。例如，如果我们公司要求所有项目都从标准 POM 继承，我们仍然可以从使用[自定义父项目](/web/20220904110305/https://www.baeldung.com/spring-boot-dependency-management-custom-parent)的 Spring Boot 依赖性管理中受益。

### Q4。什么是 Spring Initializr？

Spring Initializr 是创建 Spring Boot 项目的一种便捷方式。

我们可以去 [Spring Initializr](https://web.archive.org/web/20220904110305/https://start.spring.io/) 站点，选择一个依赖管理工具(Maven 或者 Gradle)，一种语言(Java，Kotlin 或者 Groovy)，一个打包方案(Jar 或者 War)，版本和依赖，下载项目。

这个**为我们**创建了一个框架项目，并节省了设置时间，这样我们就可以专注于添加业务逻辑。

甚至当我们使用 IDE(比如 STS 或带 STS 插件的 Eclipse)的新建项目向导来创建 Spring Boot 项目时，它也使用 Spring Initializr。

### Q5。有哪些 Spring Boot 首发球员可供选择？

每个入门者都扮演着我们需要的所有 Spring 技术的一站式商店的角色。然后，其他必需的依赖项被以一种一致的方式引入和管理。

所有首发都在`org.springframework.boot`组下，他们的名字以`spring-boot-starter-`开头。**这种命名模式使得查找初学者变得容易，尤其是在使用支持按名称搜索依赖项的 ide 时。**

在撰写本文时，我们有 50 多个启动器可供使用。在这里，我们将列出最常见的:

*   `spring-boot-starter`:核心启动器，包括自动配置支持、日志和 YAML
*   `spring-boot-starter-aop`:使用 Spring AOP 和 AspectJ 进行面向方面的编程
*   `spring-boot-starter-data-jpa:`用于将 Spring 数据 JPA 与 Hibernate 一起使用
*   `spring-boot-starter-security`:用于使用 Spring Security
*   `spring-boot-starter-test`:用于测试 Spring Boot 应用程序
*   使用 Spring MVC 构建 web 应用，包括 RESTful 应用

如需完整的启动列表，请参见[本资源库](https://web.archive.org/web/20220904110305/https://github.com/spring-projects/spring-boot/tree/master/spring-boot-project/spring-boot-starters)。

想了解更多关于 Spring Boot 首发的信息，请看一下[Spring Boot 首发简介](/web/20220904110305/https://www.baeldung.com/spring-boot-starters)。

### Q6。如何禁用特定的自动配置？

如果我们想要禁用一个特定的自动配置，我们可以使用`@EnableAutoConfiguration`注释的`exclude`属性来指示它。

例如，这段代码中和了`DataSourceAutoConfiguration`:

```java
// other annotations
@EnableAutoConfiguration(exclude = DataSourceAutoConfiguration.class)
public class MyConfiguration { }
```

如果我们使用`@SpringBootApplication`注释启用自动配置——它使用`@EnableAutoConfiguration`作为元注释——我们可以使用相同名称的属性禁用自动配置:

```java
// other annotations
@SpringBootApplication(exclude = DataSourceAutoConfiguration.class)
public class MyConfiguration { }
```

我们还可以使用`spring.autoconfigure.exclude`环境属性禁用自动配置。`application.properties`文件中的这个设置和以前做的一样:

```java
spring.autoconfigure.exclude=org.springframework.boot.autoconfigure.jdbc.DataSourceAutoConfiguration
```

### Q7。如何注册自定义自动配置？

要注册一个自动配置类，我们必须在`META-INF/spring.factories`文件中的`EnableAutoConfiguration`键下面列出它的完全限定名:

```java
org.springframework.boot.autoconfigure.EnableAutoConfiguration=com.baeldung.autoconfigure.CustomAutoConfiguration
```

如果我们用 Maven 构建一个项目，那么这个文件应该放在`resources/META-INF`目录中，在`package`阶段，这个目录将会在提到的位置结束。

### Q8。当 Bean 存在时，如何告诉自动配置退出？

要指示自动配置类在 bean 已经存在时后退，我们可以使用`@ConditionalOnMissingBean`注释。

该注释最引人注目的属性是:

*   `value`–要检查的豆子类型
*   `name`–要检查的豆子的名称

当放置在用`@Bean`修饰的方法上时，目标类型默认为方法的返回类型:

```java
@Configuration
public class CustomConfiguration {
    @Bean
    @ConditionalOnMissingBean
    public CustomService service() { ... }
}
```

### Q9。如何将 Spring Boot Web 应用程序部署为 Jar 和 War 文件？

传统上，我们将 web 应用程序打包成一个 WAR 文件，然后将其部署到外部服务器中。这样做允许我们在同一台服务器上安排多个应用程序。当 CPU 和内存不足时，这是节省资源的好方法。

但是事情已经改变了。现在计算机硬件相当便宜，注意力已经转向服务器配置。在部署过程中配置服务器的一个小错误可能会导致灾难性的后果。

**Spring 通过提供一个插件，即`spring-boot-maven-plugin`，将一个 web 应用程序打包成一个可执行的 JAR，解决了这个问题。**

要包含这个插件，只需在`pom.xml`中添加一个`plugin`元素:

```java
<plugin>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-maven-plugin</artifactId>
</plugin>
```

有了这个插件，我们将在执行`package`阶段后得到一个大罐子。这个 JAR 包含所有必需的依赖项，包括一个嵌入式服务器。因此，我们不再需要担心配置外部服务器。

然后，我们可以像运行普通的可执行 JAR 一样运行应用程序。

注意，`pom.xml`文件中的`packaging`元素必须设置为`jar`才能构建一个 JAR 文件:

```java
<packaging>jar</packaging>
```

如果我们不包含这个元素，它也默认为`jar`。

为了构建一个 WAR 文件，我们将`packaging`元素改为`war`:

```java
<packaging>war</packaging>
```

并离开打包文件的容器依赖性:

```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-tomcat</artifactId>
    <scope>provided</scope>
</dependency>
```

在执行 Maven `package`阶段之后，我们将拥有一个可部署的 WAR 文件。

### Q10。如何在命令行应用程序中使用 Spring Boot？

就像任何其他 Java 程序一样，Spring Boot 命令行应用程序必须有一个`main`方法。

这个方法作为一个入口点，调用`SpringApplication#run` 方法来引导应用程序:

```java
@SpringBootApplication
public class MyApplication {
    public static void main(String[] args) {
        SpringApplication.run(MyApplication.class);
        // other statements
    }
}
```

然后,`SpringApplication`类启动一个 Spring 容器并自动配置 beans。

注意，我们必须将一个配置类传递给`run`方法，作为主要的配置源。按照惯例，这个参数就是入口类本身。

调用`run`方法后，我们可以像在常规程序中一样执行其他语句。

### Q11。外部配置的可能来源是什么？

Spring Boot 提供了对外部配置的支持，允许我们在不同的环境中运行相同的应用程序。我们可以使用属性文件、YAML 文件、环境变量、系统属性和命令行选项参数来指定配置属性。

然后，我们可以使用`@Value`注释、通过 [`@ConfigurationProperties`注释](/web/20220904110305/https://www.baeldung.com/configuration-properties-in-spring-boot)或`Environment`抽象绑定的对象来访问这些属性。

### Q12。Spring Boot 支持宽松约束意味着什么？

Spring Boot 中的宽松绑定适用于[配置属性的类型安全绑定](/web/20220904110305/https://www.baeldung.com/configuration-properties-in-spring-boot)。

使用宽松绑定，属性的键不需要与属性名完全匹配。这样的环境属性可以用 camelCase、kebab-case、snake_case 或用下划线分隔的大写字母来编写。

例如，如果带有`@ConfigurationProperties`注释的 bean 类中的一个属性被命名为`myProp`，那么它可以被绑定到这些环境属性中的任何一个:`myProp`、`my-prop`、`my_prop`或`MY_PROP`。

### Q13。Spring Boot 开发工具是用来做什么的？

Spring Boot 开发工具，或称 DevTools，是一套使开发过程更容易的工具。

为了包含这些开发时特性，我们只需要向`pom.xml`文件添加一个依赖项:

```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-devtools</artifactId>
</dependency>
```

如果应用程序在生产环境中运行，则`spring-boot-devtools`模块会自动禁用。默认情况下，归档文件的重新打包也不包括此模块。所以，它不会给我们的最终产品带来任何开销。

默认情况下，DevTools 应用适合开发环境的属性。这些属性禁用模板缓存，启用 web 组的调试日志记录，等等。因此，我们有了这个合理的开发时配置，而无需设置任何属性。

每当类路径上的文件改变时，使用 DevTools 的应用程序就重新启动。这在开发中是一个非常有用的特性，因为它可以对修改给出快速的反馈。

默认情况下，静态资源(包括视图模板)不会触发重启。相反，资源更改会触发浏览器刷新。请注意，只有在浏览器中安装了 LiveReload 扩展以与 DevTools 包含的嵌入式 LiveReload 服务器进行交互时，才会发生这种情况。

有关该主题的更多信息，请参见[Spring Boot 开发工具概述](/web/20220904110305/https://www.baeldung.com/spring-boot-devtools)。

### Q14。如何编写集成测试？

当运行 Spring 应用程序的集成测试时，我们必须有一个`ApplicationContext`。

为了让我们的生活更容易，Spring Boot 为测试提供了一个特殊的注释— `@SpringBootTest`。这个注释从配置类创建一个`ApplicationContext`,配置类由它的`classes`属性指示。

**如果没有设置`classes`属性，Spring Boot 搜索主要配置类。**搜索从包含测试的包开始，直到找到用`@SpringBootApplication`或`@SpringBootConfiguration`标注的类。

如需详细说明，请查看我们在 Spring Boot 的[测试教程。](/web/20220904110305/https://www.baeldung.com/spring-boot-testing)

### Q15 **。Spring Boot 执行器是用来做什么的？**

从本质上讲，Actuator 通过实现生产就绪功能，将 Spring Boot 应用带入生活。这些特性使我们能够在应用程序运行于生产环境时对其进行监控和管理。

将 Spring Boot 执行器集成到项目中非常简单。我们需要做的就是在`pom.xml`文件中包含`spring-boot-starter-actuator`启动器:

```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

Spring Boot 执行器可以使用 HTTP 或 JMX 端点公开操作信息。但是大多数应用程序使用 HTTP，其中端点的身份和前缀构成了一个 URL 路径。

以下是 Actuator 提供的一些最常见的内置端点:

*   `env`暴露环境属性
*   `health`显示应用健康信息
*   `httptrace`显示 HTTP 跟踪信息
*   `info`显示任意应用信息
*   `metrics`显示指标信息
*   `loggers`显示和修改应用程序中记录器的配置
*   `mappings`显示所有`@RequestMapping`路径的列表

请参考我们的 [Spring Boot 致动器教程](/web/20220904110305/https://www.baeldung.com/spring-boot-actuators)了解详细的纲要。

### Q16。配置 Spring Boot 项目，属性和 YAML 哪个更好？

YAML 提供了许多优于属性文件的优势:

*   更加清晰，可读性更好
*   非常适合分层配置数据，这种数据也以更好、更易读的格式表示
*   支持映射、列表和标量类型
*   可以在同一个文件中包含几个[概要文件](/web/20220904110305/https://www.baeldung.com/spring-profiles)(从 Spring Boot 2.4.0 开始，这对于属性文件也是可能的)

然而，由于它的缩进规则，编写它可能有点困难并且容易出错。

有关详细信息和工作示例，请参考我们的[弹簧 YAML vs 属性](/web/20220904110305/https://www.baeldung.com/spring-yaml-vs-properties)教程。

### Q17。Spring Boot 提供了哪些基本注解？

Spring Boot 提供的主要注释存在于它的`org.springframework.boot.autoconfigure`及其子包中。

这里有几个基本的:

*   `@EnableAutoConfiguration`–让 Spring Boot 在其类路径中寻找自动配置 beans 并自动应用它们
*   `@SpringBootApplication`–表示启动应用程序的主类。该注释结合了 `@Configuration`、`@EnableAutoConfiguration`和`@ComponentScan`注释及其默认属性。

Spring Boot 注解为这个主题提供了更多的见解。

### Q18。如何更改 Spring Boot 的默认端口？

我们可以使用以下方法之一[更改嵌入在 Spring Boot](/web/20220904110305/https://www.baeldung.com/spring-boot-change-port) 中的服务器的默认端口:

*   使用属性文件——我们可以使用属性`server.port`在`application.properties`(或`application.yml`)文件中定义它。
*   以编程方式——在我们的主`@SpringBootApplication`类中，我们可以在`SpringApplication`实例上设置`server.port`。
*   使用命令行——当应用程序作为 jar 文件运行时，我们可以将 server.port 设置为 java 命令参数:

    ```java
    java -jar -Dserver.port=8081 myspringproject.jar 
    ```

### Q19。Spring Boot 支持哪些嵌入式服务器，如何更改默认值？

截止到目前， **Spring MVC 支持 Tomcat、Jetty 和 Undertow。** Tomcat 是 Spring Boot 的`web` starter 支持的默认应用服务器。

默认情况下，Spring WebFlux 支持 Reactor Netty、Tomcat、Jetty 和 under flow。

在 Spring MVC 中，要改变默认设置，比如 Jetty，我们需要在依赖项中排除 Tomcat 并包含 Jetty:

```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
    <exclusions>
        <exclusion>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-tomcat</artifactId>
        </exclusion>
    </exclusions>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-jetty</artifactId>
</dependency>
```

类似地，要将 WebFlux 中的默认值更改为 UnderTow，我们需要排除 Reactor Netty，并将 UnderTow 包含在依赖项中。

[比较 Spring Boot 的嵌入式 Servlet 容器](/web/20220904110305/https://www.baeldung.com/spring-boot-servlet-containers)有更多关于我们可以与 Spring MVC 一起使用的不同嵌入式服务器的细节。

### 问题 20。为什么我们需要弹簧剖面？

在为企业开发应用程序时，我们通常会处理多种环境，如开发、QA 和生产。这些环境的配置属性是不同的。

例如，我们可能使用嵌入式 H2 数据库进行开发，但是 Prod 可以使用专有的 Oracle 或 DB2。即使 DBMS 在不同的环境中是相同的，URL 肯定是不同的。

为了使这变得简单明了， **Spring 提供了概要文件来帮助分离每个环境的配置。**因此，属性可以保存在单独的文件中，如`application-dev.properties`和`application-prod.` `properties` ，而不是以编程方式维护。默认的`application.propertie` s 使用`spring.profiles.active`指向当前激活的配置文件，以便获得正确的配置。

[Spring Profiles](/web/20220904110305/https://www.baeldung.com/spring-profiles) 给出了这个话题的综合观点。

## 3。结论

这篇文章讨论了技术面试中可能出现的一些对 Spring Boot 最关键的问题。

我们希望他们能帮助我们找到理想的工作！