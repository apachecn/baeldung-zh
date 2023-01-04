# 没有网络服务器的 Spring Boot

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-boot-no-web-server>

## 1.介绍

Spring Boot 是为各种用例快速创建新 Java 应用程序的一个很好的框架。最受欢迎的用途之一是作为 web 服务器，使用许多受支持的嵌入式 servlet 容器和模板引擎之一。

然而， **Spring Boot 有许多不需要网络服务器**的用途:[控制台应用](/web/20221128042212/https://www.baeldung.com/spring-boot-console-app)，作业调度，[批处理](/web/20221128042212/https://www.baeldung.com/introduction-to-spring-batch)或流处理，[无服务器](/web/20221128042212/https://www.baeldung.com/spring-cloud-function)应用，等等。

在本教程中，我们将看看在没有 web 服务器的情况下使用 Spring Boot 的几种不同方式。

## 2.使用依赖关系

防止 Spring Boot 应用程序启动嵌入式 web 服务器的最简单的方法是**不在我们的依赖关系**中包含 web 服务器启动器。

这意味着不要在 Maven POM 或 Gradle 构建文件中包含`spring-boot-starter-web`依赖项。相反，我们希望用更基本的`spring-boot-starter`依赖来代替它。

请记住**Tomcat 依赖关系有可能作为可传递依赖关系**包含在我们的应用程序中。在这种情况下，我们可能需要从包含 Tomcat 库的依赖项中排除它。

## 3.修改 Spring 应用程序

在 Spring Boot 禁用嵌入式 web 服务器的另一种方法是使用代码。我们可以使用`SpringApplicationBuilder`:

```
new SpringApplicationBuilder(MainApplication.class)
  .web(WebApplicationType.NONE)
  .run(args);
```

或者我们可以参考`SpringApplication`:

```
SpringApplication application = new SpringApplication(MainApplication.class);
application.setWebApplicationType(WebApplicationType.NONE);
application.run(args);
```

在这两种情况下，**我们都有在类路径**上保持 servlet 和容器 API 可用的好处。这意味着我们仍然可以使用 web 服务器库，而无需启动 web 服务器。这可能是有用的，例如，如果我们想使用它们来编写测试或在我们自己的代码中使用它们的 API。

## 4.使用应用程序属性

使用代码禁用 web 服务器是一个静态选项——无论我们将它部署在哪里，它都会影响我们的应用程序。但是如果我们想在特定的环境下创建 web 服务器呢？

在这种情况下，我们可以使用 Spring 应用程序属性:

```
spring.main.web-application-type=none
```

或者使用等效的 YAML:

```
spring:
  main:
    web-application-type: none
```

这种方法的好处是我们可以有条件地启用 web 服务器。使用 [Spring profiles](/web/20221128042212/https://www.baeldung.com/spring-profiles) 或[条件](/web/20221128042212/https://www.baeldung.com/spring-conditionalonproperty)，我们可以控制不同部署中的 web 服务器行为。

例如，我们可以让 web 服务器在开发中运行，只是为了公开指标或其他 Spring 端点，而出于安全原因，在生产中保持禁用。

请注意，Spring Boot 的一些**早期版本使用名为`web-environment`** 的`boolean`属性来启用和禁用 web 服务器。随着传统和反应式容器在 Spring Boot 的采用，**酒店已经更名，现在使用 enum** 。

## 5.结论

不使用 web 服务器创建 Spring Boot 应用程序有很多原因。在本教程中，我们已经看到了多种方法来做到这一点。每种方法都有自己的优缺点，所以我们应该选择最符合我们需求的方法。