# Spring Boot 的 DispatcherServlet 和 web.xml

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-boot-dispatcherservlet-web-xml>

## 1.概观

[`DispatcherServlet`](/web/20221115155356/https://www.baeldung.com/spring-dispatcherservlet) 是 Spring web 应用中的前端控制器。它用于在 Spring MVC 中创建 web 应用程序和 REST 服务。在传统的 Spring web 应用程序中，这个 servlet 定义在`web.xml`文件中。

在本教程中，我们将在 Spring Boot 应用程序中将代码从`web.xml`文件迁移到`DispatcherServlet`。此外，我们将把`web.xml`中的`Filter`、`Servlet`和`Listener`类映射到 Spring Boot 应用程序。

## 2.Maven 依赖性

首先，我们必须将 [`spring-boot-starter-web`](https://web.archive.org/web/20221115155356/https://search.maven.org/artifact/org.springframework.boot/spring-boot-starter-web) Maven 依赖项添加到我们的`pom.xml`文件中:

```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

## 3.`DispatcherServlet`

DispatcherServlet 接收所有的 HTTP 请求，并将它们委派给控制器类。

**在 Servlet 3.x 规范之前，`DispatcherServlet`将被注册到 Spring MVC 应用程序的`web.xml`文件中。**自从 Servlet 3.x 规范以来，我们可以使用`ServletContainerInitializer`以编程方式注册 Servlet。

让我们在 *web.xml* 文件中查看一个 *DispatcherServlet* 示例配置:

```java
<servlet>
    <servlet-name>dispatcher</servlet-name>
    <servlet-class>
        org.springframework.web.servlet.DispatcherServlet
    </servlet-class>
</servlet>

<servlet-mapping>
    <servlet-name>dispatcher</servlet-name>
    <url-pattern>/</url-pattern>
</servlet-mapping>
```

Spring Boot 为使用 Spring MVC 开发 web 应用程序提供了`spring-boot-starter-web`库。Spring Boot 的主要特性之一是自动配置。**Spring Boot 自动配置自动注册并配置`DispatcherServlet`**。因此，我们不需要手动注册`DispatcherServlet`。

默认情况下，`spring-boot-starter-web`启动器将`DispatcherServlet`配置为 URL 模式“/”。因此，我们不需要在 *web.xml* 文件中为上面的 *DispatcherServlet* 示例完成任何额外的配置。然而，我们可以使用`application.properties`文件中的`server.servlet.` *定制 URL 模式:

```java
server.servlet.context-path=/demo
spring.mvc.servlet.path=/baeldung
```

通过这些定制，`DispatcherServlet`被配置为处理 URL 模式`/baeldung`，并且根`contextPath`将是`/demo`。因此，`DispatcherServlet`在`http://localhost:8080/demo/baeldung/.`监听

## 4.应用程序配置

Spring MVC web 应用程序使用`web.xml`文件作为部署描述符文件。此外，它还定义了 URL 路径和`web.xml`文件中的 servlets 之间的映射。

Spring Boot 的情况不再如此。如果我们需要一个特殊的过滤器，我们可以在 Java 类配置中注册它。`web.xml`文件包括过滤器、servlets 和监听器。

**当我们想从传统的 Spring MVC 迁移到现代的 Spring Boot 应用程序时，我们如何将我们的`web.xml`移植到新的 Spring Boot 应用程序？**在 Spring Boot 应用程序中，我们可以用几种方式添加这些概念。

### 4.1.注册一个`Filter`

让我们通过实现 [`Filter`](/web/20221115155356/https://www.baeldung.com/spring-boot-add-filter) 接口来创建一个过滤器:

```java
@Component
public class CustomFilter implements Filter {

    Logger logger = LoggerFactory.getLogger(CustomFilter.class);

    @Override
    public void init(FilterConfig filterConfig) throws ServletException {

    }

    @Override
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain)
      throws IOException, ServletException {
        logger.info("CustomFilter is invoked");
        chain.doFilter(request, response);
    }

    // other methods 
}
```

如果没有 Spring Boot，我们将在 *web.xml* 文件中配置我们的`CustomFilter` :

```java
<filter>
    <filter-name>customFilter</filter-name>
    <filter-class>CustomFilter</filter-class>
</filter>

<filter-mapping>
    <filter-name>customFilter</filter-name>
    <url-pattern>/*</url-pattern>
</filter-mapping>
```

为了让 Spring Boot 能够识别过滤器，我们只需要用`@Component`注释将它定义为一个 bean。

### 4.2.注册一个`Servlet`

让我们通过扩展`HttpServlet`类来定义一个 servlet:

```java
public class CustomServlet extends HttpServlet {

    Logger logger = LoggerFactory.getLogger(CustomServlet.class);

    @Override
    protected void doGet(
        HttpServletRequest req,
        HttpServletResponse resp) throws ServletException, IOException {
            logger.info("CustomServlet doGet() method is invoked");
            super.doGet(req, resp);
    }

    @Override
    protected void doPost(
        HttpServletRequest req,
        HttpServletResponse resp) throws ServletException, IOException {
            logger.info("CustomServlet doPost() method is invoked");
            super.doPost(req, resp);
    }
} 
```

没有 Spring Boot，我们将在`web.xml`文件中配置我们的`CustomServlet`:

```java
<servlet>
    <servlet-name>customServlet</servlet-name>
    <servlet-class>CustomServlet</servlet-class>
</servlet>

<servlet-mapping>
    <servlet-name>customServlet</servlet-name>
    <url-pattern>/servlet</url-pattern>
</servlet-mapping>
```

在 Spring Boot 应用程序中，servlet 要么注册为 Spring `@Bean`，要么通过扫描带有内嵌容器的`@WebServlet`注释类来注册。

使用 Spring `@Bean`方法，我们可以使用`ServletRegistrationBean`类来[注册 servlet](/web/20221115155356/https://www.baeldung.com/register-servlet#registering-servlets-in-spring-boot) 。

因此，我们将把`CustomServlet`定义为具有`ServletRegistrationBean` 类的 bean:

```java
@Bean
public ServletRegistrationBean customServletBean() {
    ServletRegistrationBean bean = new ServletRegistrationBean(new CustomServlet(), "/servlet");
    return bean;
} 
```

### 4.3.注册一个`Listener`

让我们通过扩展`ServletContextListener`类来定义一个监听器:

```java
public class CustomListener implements ServletContextListener {

    Logger logger = LoggerFactory.getLogger(CustomListener.class);

    @Override
    public void contextInitialized(ServletContextEvent sce) {
        logger.info("CustomListener is initialized");
    }

    @Override
    public void contextDestroyed(ServletContextEvent sce) {
        logger.info("CustomListener is destroyed");
    }
} 
```

没有 Spring Boot，我们将在`web.xml`文件中配置我们的`CustomListener`:

```java
<listener>
    <listener-class>CustomListener</listener-class>
</listener>
```

要在 Spring Boot 应用程序中定义监听器，我们可以使用 *@Bean* 或 *@WebListener* 注释。

使用 Spring `@Bean`方法，我们可以使用`ServletListenerRegistrationBean`类来注册`Listener`。

所以，让我们用`ServletListenerRegistrationBean` 类将`CustomListener`定义为一个 bean:

```java
@Bean
public ServletListenerRegistrationBean<ServletContextListener> customListenerBean() {
    ServletListenerRegistrationBean<ServletContextListener> bean = new ServletListenerRegistrationBean();
    bean.setListener(new CustomListener());
    return bean;
}
```

在启动我们的应用程序时，我们可以检查日志输出以确认侦听器已经成功初始化:

```java
2020-09-28 08:50:30.872 INFO 19612 --- [main] c.baeldung.demo.listener.CustomListener: CustomListener is initialized
```

## 5.结论

在这个快速教程中，我们看到了如何在 Spring Boot 应用程序中定义`DispatcherServlet`和`web.xml`元素，包括`filter`、`servlet`和`listener`。和往常一样，上面例子的源代码可以在 GitHub 上找到[。](https://web.archive.org/web/20221115155356/https://github.com/eugenp/tutorials/tree/master/spring-boot-modules/spring-boot-basic-customization-2)