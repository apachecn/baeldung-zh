# Spring Boot CLI 简介

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-boot-cli>

## 1。简介

Spring Boot CLI 是一个**命令行抽象，它允许我们轻松运行用 Groovy 脚本表示的 Spring 微服务**。它还为这些服务提供了简化和增强的依赖性管理。

这篇短文快速介绍了**如何配置 Spring Boot CLI 并执行简单的终端命令来运行预先配置的微服务**。

对于本文，我们将使用 Spring Boot CLI 2.0.0.RELEASE。可以在 [Maven Central](https://web.archive.org/web/20220524111530/https://search.maven.org/classic/#search%7Cga%7C1%7Ca%3A%22spring-boot-cli%22) 找到 Spring Boot CLI 的最新版本。

## 2。设置 Spring Boot CLI

设置 Spring Boot CLI 最简单的方法之一是使用 SDKMAN。SDKMAN 的设置和安装说明可在[这里](https://web.archive.org/web/20220524111530/https://sdkman.io/install)找到。

安装 SDKMAN 后，运行以下命令自动安装和配置 Spring Boot CLI:

```java
$ sdk install springboot
```

要验证安装，请运行命令:

```java
$ spring --version
```

我们也可以通过从源代码编译的方式安装 Spring Boot CLI，Mac 用户可以使用来自[家酿](https://web.archive.org/web/20220524111530/https://brew.sh/)或 [MacPorts](https://web.archive.org/web/20220524111530/https://www.macports.org/) 的预建包。所有安装选项见官方[文档](https://web.archive.org/web/20220524111530/https://docs.spring.io/spring-boot/docs/current/reference/html/getting-started-installing-spring-boot.html#getting-started-installing-the-cli)。

## 3。公共终端命令

Spring Boot CLI 提供了一些现成的有用命令和功能。最有用的特性之一是 Spring Shell，它用必要的`spring`前缀包装命令。

为了**启动嵌入式 shell** ，我们运行:

```java
spring shell
```

从这里，我们可以直接输入想要的命令，而不需要预先挂起关键字`spring`(因为我们现在在 spring shell 中)。

例如，我们可以通过键入以下内容来**显示正在运行的 CLI 的当前版本**:

```java
version
```

**最重要的命令之一是告诉 Spring Boot CLI 运行一个 Groovy 脚本:**

```java
run [SCRIPT_NAME].groovy
```

Spring Boot CLI 将自动推断依赖关系，或者根据正确提供的注释进行推断。此后，它将推出一个嵌入式网络容器和应用程序。

让我们仔细看看如何在 Spring Boot CLI 中使用 Groovy 脚本！

## 4。基本 Groovy 脚本

Groovy 和 Spring 与 Spring Boot CLI 一起**允许在单文件 Groovy 部署中快速编写强大、高性能的微服务脚本**。

支持多脚本应用程序通常需要额外的构建工具，如 [Maven](/web/20220524111530/https://www.baeldung.com/maven) 或 [Gradle](/web/20220524111530/https://www.baeldung.com/gradle) 。

下面我们将介绍 Spring Boot CLI 的一些最常见的使用案例，将更复杂的设置留给其他文章。

关于所有 Spring 支持的 Groovy 注释的列表，请查看官方的[文档](https://web.archive.org/web/20220524111530/https://docs.spring.io/spring-boot/docs/current/reference/html/cli-using-the-cli.html)。

### 4.1。`@Grab`

`@Grab`注释和 Groovy 的 Java-esque `import`子句允许**简单的依赖管理和注入**。

事实上，大多数注释抽象、简化并自动包含必要的导入语句。这允许我们花更多的时间来考虑我们想要部署的服务的架构和底层逻辑。

让我们看看如何使用`@Grab`注释:

```java
package org.test

@Grab("spring-boot-starter-actuator")

@RestController
class ExampleRestController{
  //...
}
```

正如我们所看到的， **`spring-boot-starter-actuator`是预先配置的，允许简洁的脚本部署，而不需要定制的应用程序或环境属性，`XML`或其他编程配置**，尽管这些都可以在必要时指定。

完整的`@Grab`参数列表——每个参数都指定了要下载和导入的库——可以在[这里](https://web.archive.org/web/20220524111530/https://docs.spring.io/spring-boot/docs/current/reference/html/appendix-dependency-versions.html)找到。

### 4.2。`@Controller, @RestController,` 和`@EnableWebMvc`

为了进一步加快部署，我们可以选择**利用 Spring Boot CLI 提供的“抓取提示”来自动推断正确的依赖项以导入**。

下面我们将讨论一些最常见的使用案例。

例如，我们可以使用熟悉的`@Controller`和`@Service`注释来**快速搭建一个标准的 MVC 控制器和服务**:

```java
@RestController
class Example {

    @Autowired
    private MyService myService;

    @GetMapping("/")
    public String helloWorld() {
        return myService.sayWorld();
    }
}

@Service
class MyService {
    public String sayWorld() {
        return "World!";
    }
}
```

Spring Boot CLI 支持 Spring Boot 的所有默认配置。因此，我们可以让 Groovy 应用程序从它们通常的默认位置自动访问静态资源。

### 4.3。@ `EnableWebSecurity`

为了将 Spring Boot 安全选项添加到我们的应用程序中，我们可以使用`@EnableWebSecurity`注释，该注释将由 Spring Boot CLI 自动下载。

下面，我们将使用 `spring-boot-starter-security` 依赖来抽象这个过程的一部分，它利用了幕后的`@EnableWebSecurity`注释:

```java
package bael.security

@Grab("spring-boot-starter-security")

@RestController
class SampleController {

    @RequestMapping("/")
    public def example() {
        [message: "Hello World!"]
    }
} 
```

关于如何保护资源和处理安全性的更多细节，请查看官方的[文档](https://web.archive.org/web/20220524111530/https://spring.io/projects/spring-cloud-security)。

### 4.4。@ `Test`

为了**建立一个简单的 JUnit 测试**，我们可以添加`@Grab(‘junit')`或`@Test`注释:

```java
package bael.test

@Grab('junit')
class Test {
    //...
}
```

这将允许我们轻松地执行 JUnit 测试。

### 4.5。`DataSource` 和`JdbcTemplate`

可以指定持久数据选项，包括`DataSource`或`JdbcTemplate` **，而无需显式使用`@Grab` 注释**:

```java
package bael.data

@Grab('h2')
@Configuration
@EnableWebMvc
@ComponentScan('bael.data')
class DataConfig {

    @Bean
    DataSource dataSource() {
        return new EmbeddedDatabaseBuilder()
          .setType(EmbeddedDatabaseType.H2).build();
    }

}
```

**通过简单地使用熟悉的 Spring bean 配置约定**，我们获得了 H2 嵌入式数据库，并将其设置为`DataSource`。

## 5。自定义配置

使用 Spring Boot CLI 配置 Spring Boot 微服务有两种主要方法:

1.  我们可以在终端命令中添加参数
2.  我们可以使用定制的 YAML 文件来提供应用程序配置

Spring Boot 将自动在`/config`目录中搜索`application.yml`或`application.properties`

```java
├── app
    ├── app.groovy
    ├── config
        ├── application.yml
    ... 
```

我们还可以设置:

```java
├── app
    ├── example.groovy
    ├── example.yml
    ...
```

Spring 上的[这里](https://web.archive.org/web/20220524111530/https://docs.spring.io/spring-boot/docs/current/reference/html/common-application-properties.html)可以找到应用属性的完整列表。

## 6。结论

我们对 Spring Boot CLI 的快速浏览到此结束！更多详情，请查看官方[文档](https://web.archive.org/web/20220524111530/https://docs.spring.io/spring-boot/docs/current/reference/html/cli-using-the-cli.html)。

和往常一样，这篇文章的源代码可以在 GitHub 上找到[。](https://web.archive.org/web/20220524111530/https://github.com/eugenp/tutorials/tree/master/spring-boot-modules/spring-boot-cli)