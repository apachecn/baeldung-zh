# 配置 Spring Boot Web 应用程序

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-boot-application-configuration>

## 1。概述

Spring Boot 可以做很多事情；在本教程中，我们将讨论一些更有趣的启动配置选项。

## 延伸阅读:

## [从春天迁徙到 Spring Boot](/web/20220812053707/https://www.baeldung.com/spring-boot-migration)

看看如何恰当地从春天迁徙到 Spring Boot。[阅读更多](/web/20220812053707/https://www.baeldung.com/spring-boot-migration) →

## [用 Spring Boot](/web/20220812053707/https://www.baeldung.com/spring-boot-custom-starter)

创建自定义 Spring Boot 启动器的快速实用指南。[了解更多](/web/20220812053707/https://www.baeldung.com/spring-boot-custom-starter)→

## [Spring Boot 测试](/web/20220812053707/https://www.baeldung.com/spring-boot-testing)

了解 Spring Boot 如何支持测试，以高效地编写单元测试。[阅读更多](/web/20220812053707/https://www.baeldung.com/spring-boot-testing) →

## 2。端口号

在主独立应用中，主 HTTP 端口默认为 8080；**我们可以轻松地配置 Boot 来使用不同的端口**:

```
server.port=8083
```

对于基于 YAML 的配置:

```
server:
    port: 8083
```

我们还可以通过编程定制服务器端口:

```
@Component
public class CustomizationBean implements
  WebServerFactoryCustomizer<ConfigurableServletWebServerFactory> {

    @Override
    public void customize(ConfigurableServletWebServerFactory container) {
        container.setPort(8083);
    }
}
```

## 3。上下文路径

默认情况下，上下文路径是“/”。如果这不理想，您需要将其更改为/ `app_name`之类的内容，下面是通过属性进行更改的快速而简单的方法:

```
server.servlet.contextPath=/springbootapp
```

对于基于 YAML 的配置:

```
server:
    servlet:
        contextPath:/springbootapp
```

最后，更改也可以通过编程来完成:

```
@Component
public class CustomizationBean
  implements WebServerFactoryCustomizer<ConfigurableServletWebServerFactory> {

    @Override
    public void customize(ConfigurableServletWebServerFactorycontainer) {
        container.setContextPath("/springbootapp");
    }
}
```

## 4。白色标签错误页面

如果您没有在配置中指定任何自定义实现，Spring Boot 会自动注册一个``BasicErrorController`` bean。

然而，这个默认控制器当然可以配置为:

```
public class MyCustomErrorController implements ErrorController {

    private static final String PATH = "/error";

    @GetMapping(value=PATH)
    public String error() {
        return "Error haven";
    }
}
```

## 5。自定义错误消息

默认情况下，Boot 提供了`/error` 映射，以合理的方式处理错误。

如果您想配置更具体的错误页面，可以使用统一的 Java DSL 来定制错误处理:

```
@Component
public class CustomizationBean
  implements WebServerFactoryCustomizer<ConfigurableServletWebServerFactory> {

    @Override
    public void customize(ConfigurableServletWebServerFactorycontainer) {        
        container.addErrorPages(new ErrorPage(HttpStatus.BAD_REQUEST, "/400"));
        container.addErrorPages(new ErrorPage("/errorHaven"));
    }
}
```

这里，我们专门处理了`Bad Request`来匹配`/400`路径，并处理了所有其他的来匹配公共路径。

一个非常简单的`/errorHaven`实现:

```
@GetMapping("/errorHaven")
String errorHeaven() {
    return "You have reached the haven of errors!!!";
}
```

输出:

```
You have reached the haven of errors!!!
```

## 6。以编程方式关闭启动应用程序

你可以在`SpringApplication.` 的帮助下以编程的方式关闭一个引导应用程序。这有一个静态的`exit()` 方法，它有两个参数:`ApplicationContext` 和`ExitCodeGenerator`:

```
@Autowired
public void shutDown(ExecutorServiceExitCodeGenerator exitCodeGenerator) {
    SpringApplication.exit(applicationContext, exitCodeGenerator);
}
```

就是通过这个实用方法，我们可以关闭 app。

## 7。配置日志记录级别

您可以轻松地**调整引导应用程序**中的日志记录级别；从版本 1.2.0 开始，您可以在主属性文件中配置日志级别:

```
logging.level.org.springframework.web: DEBUG
logging.level.org.hibernate: ERROR
```

就像标准的 Spring 应用程序一样，您可以通过在类路径中添加定制的 XML 或属性文件并在 pom 中定义库来激活不同的日志系统，如`Logback`、`log4j`、`log4j2`等。

## 8。注册一个新的 Servlet

如果您在嵌入式服务器的帮助下部署应用程序，您可以在引导应用程序**中注册新的 Servlets，方法是将它们作为 bean**从传统配置中公开:

```
@Bean
public HelloWorldServlet helloWorld() {
    return new HelloWorldServlet();
}
```

或者，您可以使用`ServletRegistrationBean**:**` 

```
@Bean
public SpringHelloServletRegistrationBean servletRegistrationBean() {

    SpringHelloServletRegistrationBean bean = new SpringHelloServletRegistrationBean(
      new SpringHelloWorldServlet(), "/springHelloWorld/*");
    bean.setLoadOnStartup(1);
    bean.addInitParameter("message", "SpringHelloWorldServlet special message");
    return bean;
}
```

## 9。在启动应用程序中配置 Jetty 或 under flow

Spring Boot 初学者通常使用 **Tomcat 作为默认的嵌入式服务器**。如果需要更改，您可以排除 Tomcat 依赖项，而包含 Jetty 或 Undertow:

**配置码头**

```
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

```
@Bean
public JettyEmbeddedServletContainerFactory  jettyEmbeddedServletContainerFactory() {
    JettyEmbeddedServletContainerFactory jettyContainer = 
      new JettyEmbeddedServletContainerFactory();

    jettyContainer.setPort(9000);
    jettyContainer.setContextPath("/springbootapp");
    return jettyContainer;
}
```

**配置逆流**

```
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
    <artifactId>spring-boot-starter-undertow</artifactId>
</dependency>
```

```
@Bean
public UndertowEmbeddedServletContainerFactory embeddedServletContainerFactory() {
    UndertowEmbeddedServletContainerFactory factory = 
      new UndertowEmbeddedServletContainerFactory();

    factory.addBuilderCustomizers(new UndertowBuilderCustomizer() {
        @Override
        public void customize(io.undertow.Undertow.Builder builder) {
            builder.addHttpListener(8080, "0.0.0.0");
        }
    });

    return factory;
}
```

## 10。结论

在这篇简短的文章中，我们回顾了一些更加有趣和有用的 Spring Boot 配置选项。

当然，在参考文档中有很多很多选项可以根据您的需要配置和调整引导应用程序——这些只是我发现的一些更有用的选项。

本文中使用的代码可以在我们的 Github 知识库中找到。