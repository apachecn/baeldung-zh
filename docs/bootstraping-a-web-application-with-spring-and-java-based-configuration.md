# 用 Spring 5 创建 Web 应用程序

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/bootstraping-a-web-application-with-spring-and-java-based-configuration>

## 1。概述

该教程演示了如何用 Spring 创建一个 Web 应用程序。

我们将研究构建应用程序的 Spring Boot 解决方案，同时也将看到一种非 Spring Boot 的方法。

我们将主要使用 Java 配置，但是也看看它们的等价 XML 配置。

## 延伸阅读:

## [Spring Boot 教程——引导一个简单的应用](/web/20220727020632/https://www.baeldung.com/spring-boot-start)

This is how you start understanding Spring Boot.[Read more](/web/20220727020632/https://www.baeldung.com/spring-boot-start) →

## [配置 Spring Boot 网络应用](/web/20220727020632/https://www.baeldung.com/spring-boot-application-configuration)

Some of the more useful configs for a Spring Boot application.[Read more](/web/20220727020632/https://www.baeldung.com/spring-boot-application-configuration) →

## 从春天迁徙到 Spring Boot

See how to properly migrate from a Spring to Spring Boot.[Read more](/web/20220727020632/https://www.baeldung.com/spring-boot-migration) →

## 2.使用 Spring Boot 进行设置

### 2.1.Maven 依赖性

首先，我们需要 [spring-boot-starter-web](https://web.archive.org/web/20220727020632/https://search.maven.org/search?q=a:spring-boot-starter-web) 依赖关系:

```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
    <version>2.6.1</version>
</dependency>
```

该启动器包括:

*   `spring-web`和我们的 Spring web 应用程序需要的`spring-webmvc`模块
*   一个 Tomcat 启动器，这样我们就可以直接运行我们的 web 应用程序，而不需要显式安装任何服务器

### 2.2.创建 Spring Boot 应用程序

**开始使用 Spring Boot 最直接的方法是创建一个主类并用`@SpringBootApplication`** 对其进行注释:

```java
@SpringBootApplication
public class SpringBootRestApplication {

    public static void main(String[] args) {
        SpringApplication.run(SpringBootRestApplication.class, args);
    }
}
```

这个单独的注释相当于使用`@Configuration`、`@EnableAutoConfiguration`和`@ComponentScan.`

默认情况下，它将扫描同一包或更低包中的所有组件。

**接下来，对于 Spring beans 的基于 Java 的配置，我们需要创建一个 config 类并用`@Configuration`注释**对其进行注释:

```java
@Configuration
public class WebConfig {

}
```

这个注释是基于 Java 的 Spring 配置使用的主要工件；它本身用`@Component`进行了元注释，这使得带注释的类成为标准 beans，因此也是组件扫描的候选对象。

`@Configuration`类的主要目的是作为 Spring IoC 容器的 bean 定义的来源。更详细的描述见[官方文件](https://web.archive.org/web/20220727020632/http://static.springsource.org/spring/docs/current/spring-framework-reference/html/beans.html#beans-java "Java based container configuration - the 3.1 official Spring reference")。

让我们看看使用核心`spring-webmvc`库的解决方案。

## 3.使用 spring-webmvc 进行设置

### 3.1。Maven 依赖关系

首先，我们需要 spring-webmvc 的依赖关系:

```java
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-webmvc</artifactId>
    <version>5.3.3</version>
</dependency>
```

### 3.2。基于 Java 的网络配置

接下来，我们将添加带有`@Configuration`注释的配置类:

```java
@Configuration
@EnableWebMvc
@ComponentScan(basePackages = "com.baeldung.controller")
public class WebConfig {

}
```

**这里，与 Spring Boot 的解决方案不同，我们必须明确定义`@EnableWebMvc`来设置默认的 Spring MVC 配置，以及`@ComponentScan`来指定要扫描组件的包。**

`@EnableWebMvc`注释提供了 Spring Web MVC 配置，比如设置 dispatcher servlet、启用`@Controller`和`@RequestMapping`注释以及设置其他默认设置。

`@ComponentScan`配置组件扫描指令，指定要扫描的软件包。

### 3.3.初始化器类

接下来，我们需要**添加一个实现`WebApplicationInitializer`接口的类:**

```java
public class AppInitializer implements WebApplicationInitializer {

    @Override
    public void onStartup(ServletContext container) throws ServletException {
        AnnotationConfigWebApplicationContext context = new AnnotationConfigWebApplicationContext();
        context.scan("com.baeldung");
        container.addListener(new ContextLoaderListener(context));

        ServletRegistration.Dynamic dispatcher = 
          container.addServlet("mvc", new DispatcherServlet(context));
        dispatcher.setLoadOnStartup(1);
        dispatcher.addMapping("/");   
    }
}
```

这里，我们使用`AnnotationConfigWebApplicationContext`类创建一个 Spring 上下文，这意味着我们只使用基于注释的配置。然后，我们指定包来扫描组件和配置类。

最后，我们定义了 web 应用程序的入口点——`DispatcherServlet.`

这个类可以完全替换来自< 3.0 Servlet 版本的`web.xml`文件。

## 4.XML 配置

让我们快速看一下等效的 XML web 配置:

```java
<context:component-scan base-package="com.baeldung.controller" />
<mvc:annotation-driven />
```

我们可以用上面的`WebConfig`类替换这个 XML 文件。

为了启动应用程序，我们可以使用一个初始化器类来加载 XML 配置或 web.xml 文件。关于这两种方法的更多细节，请查看我们之前的文章。

## 5。结论

在本文中，我们研究了两种用于引导 Spring web 应用程序的流行解决方案，一种使用 Spring Boot web starter，另一种使用 spring-webmvc 核心库。

在下一篇关于 REST 和 Spring 的文章中，我将介绍如何在项目中设置 MVC、HTTP 状态代码的配置、有效负载编组和内容协商。

和往常一样，本文中的代码可以从 Github 上的[处获得。这是一个基于 Maven 的项目，因此应该很容易导入和运行。](https://web.archive.org/web/20220727020632/https://github.com/eugenp/tutorials/tree/master/spring-boot-rest)