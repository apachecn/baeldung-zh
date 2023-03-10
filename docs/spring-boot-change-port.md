# 如何更改 Spring Boot 的默认端口

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-boot-change-port>

## 1。概述

Spring Boot 为许多配置属性提供了合理的默认值。但是我们有时需要用特定于案例的值来定制它们。

一个常见的用例是**改变嵌入式服务器的默认端口。**

在这个快速教程中，我们将介绍实现这一点的几种方法。

## 延伸阅读:

## [弹簧和 Spring Boot 的特性](/web/20221004040433/https://www.baeldung.com/properties-with-spring)

Tutorial for how to work with properties files and property values in Spring.[Read more](/web/20221004040433/https://www.baeldung.com/properties-with-spring) →

## [Spring Boot 改变上下文路径](/web/20221004040433/https://www.baeldung.com/spring-boot-context-path)

Learn various ways of changing the context path in your Spring Boot application[Read more](/web/20221004040433/https://www.baeldung.com/spring-boot-context-path) →

## 2。使用属性文件

自定义 Spring Boot 最快速、最简单的方法是覆盖默认属性的值。

**对于服务器端口，我们要改变的属性是`server.port`。**

默认情况下，嵌入式服务器在端口 8080 上启动。

**那么，让我们看看如何在`application.properties`文件**中提供不同的值:

```java
server.port=8081
```

现在，服务器将在端口 8081 上启动。

如果我们使用的是`application.yml`文件，我们也可以这样做:

```java
server:
  port : 8081
```

如果将这两个文件放在 Maven 应用程序的`src/main/resources`目录中，Spring Boot 会自动加载它们。

### 2.1.特定于环境的端口

如果我们有一个部署在不同环境中的应用程序，我们可能希望它在每个系统的不同端口上运行。

通过将属性文件方法与 Spring 概要文件结合起来，我们可以很容易地实现这一点。具体来说，我们可以为每个环境创建一个属性文件。

例如，我们将有一个包含以下内容的`application-dev.properties`文件:

```java
server.port=8081
```

然后我们将添加另一个带有不同端口的`application-qa.properties`文件:

```java
server.port=8082
```

现在，属性文件配置对于大多数情况应该足够了。然而，这个目标还有其他的选择，所以让我们也来探索一下。

## 3.程序配置

我们可以通过在启动应用程序时设置特定的属性或者定制嵌入式服务器配置来以编程方式配置端口。

首先，让我们看看如何在主`@SpringBootApplication`类中设置属性:

```java
@SpringBootApplication
public class CustomApplication {
    public static void main(String[] args) {
        SpringApplication app = new SpringApplication(CustomApplication.class);
        app.setDefaultProperties(Collections
          .singletonMap("server.port", "8083"));
        app.run(args);
    }
}
```

接下来，为了定制服务器配置，我们必须实现`WebServerFactoryCustomizer`接口:

```java
@Component
public class ServerPortCustomizer 
  implements WebServerFactoryCustomizer<ConfigurableWebServerFactory> {

    @Override
    public void customize(ConfigurableWebServerFactory factory) {
        factory.setPort(8086);
    }
}
```

请注意，这适用于 Spring Boot 2.x 版本。

对于 Spring Boot 1.x，我们可以类似地实现`EmbeddedServletContainerCustomizer`接口。

## 4。使用命令行参数

当将我们的应用程序打包成 jar 并运行时，我们可以用`java`命令设置`server.port`参数:

```java
java -jar spring-5.jar --server.port=8083
```

或者使用等效的语法:

```java
java -jar -Dserver.port=8083 spring-5.jar
```

## 5.评估顺序

最后，让我们看看 Spring Boot 对这些方法的评估顺序。

基本上，配置优先级是

*   嵌入式服务器配置
*   命令行参数
*   属性文件
*   主`@SpringBootApplication`配置

## 6.结论

在本文中，我们看到了如何在 Spring Boot 应用程序中配置服务器端口。

与往常一样，GitHub 上的[提供了示例的源代码。](https://web.archive.org/web/20221004040433/https://github.com/eugenp/tutorials/tree/master/spring-boot-modules/spring-boot-basic-customization)