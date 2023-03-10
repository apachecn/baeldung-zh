# Spring Boot 的@ServletComponentScan 注释

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-servletcomponentscan>

## 1.概观

在本文中，我们将浏览`Spring Boot.`中新的`[@ServletComponentScan](https://web.archive.org/web/20221208143917/https://docs.spring.io/spring-boot/docs/current/api/org/springframework/boot/web/servlet/ServletComponentScan.html)`注释

目的是支持以下`Servlet 3.0`注释:

*   `javax.servlet.annotation.WebFilter`
*   `javax.servlet.annotation.WebListener`
*   `javax.servlet.annotation.WebServlet`

通过在`@Configuration`类上标注`@ServletComponentScan`并指定包，`@WebServlet`、`@WebFilter`和`@WebListener`标注的类可以自动注册到嵌入的`Servlet`容器中。

我们已经在[Java servlet](/web/20221208143917/https://www.baeldung.com/intro-to-servlets)介绍中介绍了`@WebServlet`的基本用法，在[Java](/web/20221208143917/https://www.baeldung.com/intercepting-filter-pattern-in-java)拦截过滤模式介绍中介绍了`@WebFilter`的基本用法。对于`@WebListener`，你可以看一下这篇文章，它展示了网络监听器的一个典型用例。

## 2.`Servlets`、`Filters`和`Listeners`

在潜入`@ServletComponentScan`之前，我们先来看看`@WebServlet`、`@WebFilter`、`@WebListener`在`@ServletComponentScan`进场之前是如何使用的。

### 2.1.`@WebServlet`

现在我们将首先定义一个服务于`GET`请求并响应`“hello”`的`Servlet`:

```java
@WebServlet("/hello")
public class HelloServlet extends HttpServlet {

    @Override
    public void doGet(HttpServletRequest request, HttpServletResponse response) {
        try {
            response
              .getOutputStream()
              .write("hello");
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

}
```

### 2.2.`@WebFilter`

然后一个过滤器过滤对目标`“/hello”`的请求，并将`“filtering “`添加到输出中:

```java
@WebFilter("/hello")
public class HelloFilter implements Filter {

    //...
    @Override
    public void doFilter(
      ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain) 
      throws IOException, ServletException {
        servletResponse
          .getOutputStream()
          .print("filtering ");
        filterChain.doFilter(servletRequest, servletResponse);
    }
    //...

}
```

### 2.3.`@WebListener`

最后，在`ServletContext`中设置定制属性的监听器:

```java
@WebListener
public class AttrListener implements ServletContextListener {

    @Override
    public void contextInitialized(ServletContextEvent servletContextEvent) {
        servletContextEvent
          .getServletContext()
          .setAttribute("servlet-context-attr", "test");
    }
    //...
}
```

### 2.4.部署到一个`Servlet`容器

现在我们已经构建了一个简单 web 应用程序的基本组件，我们可以将它打包并部署到一个`Servlet`容器中。通过将打包的 war 文件部署到 [`Jetty`](/web/20221208143917/https://www.baeldung.com/deploy-to-jetty) 、 [`Tomcat`](/web/20221208143917/https://www.baeldung.com/tomcat-deploy-war) 或任何支持`Servlet` 3.0 的`Servlet`容器中，可以很容易地验证每个组件的行为。

## 3.使用`Spring Boot`中的`@ServletComponentScan`

您可能想知道，既然我们可以在大多数`Servlet`容器中使用这些注释而无需任何配置，为什么我们还需要`@ServletComponentScan`？问题在于嵌入式`Servlet`容器。

由于嵌入式容器不支持`@WebServlet`、`@WebFilter`和`@WebListener`注释，`Spring Boot,` 非常依赖嵌入式容器，引入了这个新的注释`@ServletComponentScan`来支持一些使用这三个注释的依赖 jar。

详细的讨论可以在 Github 上的[这一期找到。](https://web.archive.org/web/20221208143917/https://github.com/spring-projects/spring-boot/issues/2290)

### 3.1.Maven 依赖性

要使用`@ServletComponentScan`，我们需要 1.3.0 或以上版本的`Spring Boot`。下面给`pom`加上最新版本的 [`spring-boot-starter-parent`](https://web.archive.org/web/20221208143917/https://search.maven.org/classic/#artifactdetails%7Corg.springframework.boot%7Cspring-boot-starter-parent%7C1.5.1.RELEASE%7Cpom) 和 [`spring-boot-starter-web`](https://web.archive.org/web/20221208143917/https://search.maven.org/classic/#artifactdetails%7Corg.springframework.boot%7Cspring-boot-starter-web%7C1.5.1.RELEASE%7Cjar) :

```java
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.4.0</version>
    <relativePath /> <!-- lookup parent from repository -->
</parent>
```

```java
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
        <version>2.4.0</version>
    </dependency>
</dependencies>
```

### 3.2.使用`@ServletComponentScan`

这个应用程序非常简单。我们添加了`@ServletComponentScan`来启用对`@WebFilter`、`@WebListener`和`@WebServlet:`的扫描

```java
@ServletComponentScan
@SpringBootApplication
public class SpringBootAnnotatedApp {

    public static void main(String[] args) {
        SpringApplication.run(SpringBootAnnotatedApp.class, args);
    }

}
```

无需对之前的 web 应用程序进行任何更改，它就可以正常工作:

```java
@Autowired private TestRestTemplate restTemplate;

@Test
public void givenServletFilter_whenGetHello_thenRequestFiltered() {

    ResponseEntity<String> responseEntity = 
      restTemplate.getForEntity("/hello", String.class);

    assertEquals(HttpStatus.OK, responseEntity.getStatusCode());
    assertEquals("filtering hello", responseEntity.getBody());
}
```

```java
@Autowired private ServletContext servletContext;

@Test
public void givenServletContext_whenAccessAttrs_thenFoundAttrsPutInServletListner() {

    assertNotNull(servletContext);
    assertNotNull(servletContext.getAttribute("servlet-context-attr"));
    assertEquals("test", servletContext.getAttribute("servlet-context-attr"));
}
```

### 3.3.指定要扫描的包

默认情况下，`@ServletComponentScan`将从带注释的类的包中扫描。要指定扫描哪些包，我们可以使用它的属性:

*   `value`
*   `basePackages`
*   `basePackageClasses`

默认的`value`属性是`basePackages`的别名。

假设我们的`SpringBootAnnotatedApp`在包`com.baeldung.annotation`下，我们想要扫描在上面的 web 应用程序中创建的包`com.baeldung.annotation.components`中的类，下面的配置是等价的:

```java
@ServletComponentScan
```

```java
@ServletComponentScan("com.baeldung.annotation.components")
```

```java
@ServletComponentScan(basePackages = "com.baeldung.annotation.components")
```

```java
@ServletComponentScan(
  basePackageClasses = 
    {AttrListener.class, HelloFilter.class, HelloServlet.class})
```

## 4.在后台

`@ServletComponentScan`标注由 [`ServletComponentRegisteringPostProcessor`](https://web.archive.org/web/20221208143917/https://github.com/spring-projects/spring-boot/blob/master/spring-boot-project/spring-boot/src/main/java/org/springframework/boot/web/servlet/ServletComponentRegisteringPostProcessor.java) 处理。在扫描指定的`@WebFilter`、`@WebListener`和`@WebServlet`注释包后，[、`ServletComponentHandlers`、](https://web.archive.org/web/20221208143917/https://github.com/spring-projects/spring-boot/blob/master/spring-boot-project/spring-boot/src/main/java/org/springframework/boot/web/servlet/ServletComponentHandler.java)列表将处理它们的注释属性，并注册扫描的 beans:

```java
class ServletComponentRegisteringPostProcessor
  implements BeanFactoryPostProcessor, ApplicationContextAware {

    private static final List<ServletComponentHandler> HANDLERS;

    static {
        List<ServletComponentHandler> handlers = new ArrayList<>();
        handlers.add(new WebServletHandler());
        handlers.add(new WebFilterHandler());
        handlers.add(new WebListenerHandler());
        HANDLERS = Collections.unmodifiableList(handlers);
    }

    //...

    private void scanPackage(
      ClassPathScanningCandidateComponentProvider componentProvider, 
      String packageToScan){
        //...
        for (ServletComponentHandler handler : HANDLERS) {
            handler.handle(((ScannedGenericBeanDefinition) candidate),
              (BeanDefinitionRegistry) this.applicationContext);
        }
    }
}
```

正如[官方 Javadoc](https://web.archive.org/web/20221208143917/https://docs.spring.io/spring-boot/docs/current/api/org/springframework/boot/web/servlet/ServletComponentScan.html) 中所说， **`@ServletComponentScan`注释只在嵌入式`Servlet`容器**中起作用，这是`Spring Boot`默认自带的。

## 5.结论

在本文中，我们介绍了`@ServletComponentScan`以及如何使用它来支持依赖于任何注释的应用程序:`@WebServlet`、`@WebFilter`、`@WebListener`。

示例和代码的实现可以在 GitHub 项目中的[中找到。](https://web.archive.org/web/20221208143917/https://github.com/eugenp/tutorials/tree/master/spring-boot-modules/spring-boot-mvc)