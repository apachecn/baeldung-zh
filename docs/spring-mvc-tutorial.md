# Spring MVC 教程

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-mvc-tutorial>

## 1。概述

这是一个简单的 Spring MVC 教程，展示了如何建立一个 Spring MVC 项目，包括基于 Java 的配置和 XML 配置。

Spring MVC 项目的 Maven 依赖性在 [Spring MVC 依赖性](/web/20221228013248/https://www.baeldung.com/spring-with-maven#mvc "Spring MVC - the Maven dependencies")一文中有详细描述。

## 2。什么是 Spring MVC？

顾名思义，**它是 Spring 框架中处理模型-视图-控制器或 MVC 模式的一个模块。**它结合了 MVC 模式的所有优点和 Spring 的便利性。

Spring 使用它的`DispatcherServlet` 实现了带有[前端控制器模式的 MVC。](/web/20221228013248/https://www.baeldung.com/spring-controllers#Overview)

简而言之，`DispatcherServlet`充当主控制器，将请求路由到它们预期的目的地。模型只不过是我们应用程序的数据，视图由各种模板引擎中的任何一个来表示。

我们一会儿将看看我们例子中的 JSP。

## 3。使用 Java 配置的 spring MVC

为了通过 Java 配置类启用 Spring MVC 支持，我们只需**添加`@EnableWebMvc`注释**:

```java
@EnableWebMvc
@Configuration
public class WebConfig {

    /// ...
}
```

这将为 MVC 项目建立我们需要的基本支持，例如注册控制器和映射、类型转换器、验证支持、消息转换器和异常处理。

**如果我们想要定制这个配置，我们需要实现`WebMvcConfigurer`接口**:

```java
@EnableWebMvc
@Configuration
public class WebConfig implements WebMvcConfigurer {

   @Override
   public void addViewControllers(ViewControllerRegistry registry) {
      registry.addViewController("/").setViewName("index");
   }

   @Bean
   public ViewResolver viewResolver() {
      InternalResourceViewResolver bean = new InternalResourceViewResolver();

      bean.setViewClass(JstlView.class);
      bean.setPrefix("/WEB-INF/view/");
      bean.setSuffix(".jsp");

      return bean;
   }
}
```

在这个例子中，我们注册了一个从`/WEB-INF/view`目录返回`.jsp`视图的`ViewResolver` bean。

这里非常重要的是，**我们可以注册视图控制器，使用`ViewControllerRegistry`在 URL 和视图名**之间创建直接映射。这样，两者之间就不需要任何控制器了。

如果我们还想定义和扫描控制器类，我们可以在包含控制器的包中添加`@ComponentScan`注释:

```java
@EnableWebMvc
@Configuration
@ComponentScan(basePackages = { "com.baeldung.web.controller" })
public class WebConfig implements WebMvcConfigurer {
    // ...
}
```

为了引导一个应用程序加载这个配置，我们还需要一个初始化器类:

```java
public class MainWebAppInitializer implements WebApplicationInitializer {
    @Override
    public void onStartup(final ServletContext sc) throws ServletException {

        AnnotationConfigWebApplicationContext root = 
          new AnnotationConfigWebApplicationContext();

        root.scan("com.baeldung");
        sc.addListener(new ContextLoaderListener(root));

        ServletRegistration.Dynamic appServlet = 
          sc.addServlet("mvc", new DispatcherServlet(new GenericWebApplicationContext()));
        appServlet.setLoadOnStartup(1);
        appServlet.addMapping("/");
    }
}
```

注意，对于早于 Spring 5 的版本，我们必须使用`WebMvcConfigurerAdapter`类来代替接口。

## 4。 **Spring MVC 使用 XML 配置**

除了上面的 Java 配置，我们还可以使用纯 XML 配置:

```java
<context:component-scan base-package="com.baeldung.web.controller" />
<mvc:annotation-driven />    

<bean id="viewResolver" 
      class="org.springframework.web.servlet.view.InternalResourceViewResolver">
        <property name="prefix" value="/WEB-INF/view/" />
        <property name="suffix" value=".jsp" />
    </bean>

    <mvc:view-controller path="/" view-name="index" />

</beans>
```

如果我们想使用纯 XML 配置，我们还需要添加一个`web.xml`文件来引导应用程序。关于这种方法的更多细节，请查看我们之前的文章。

## 5。控制器和视图

让我们来看一个基本控制器的例子:

```java
@Controller
public class SampleController {
    @GetMapping("/sample")
    public String showForm() {
        return "sample";
    }

}
```

而对应的 JSP 资源是`sample.jsp`文件:

```java
<html>
   <head></head>

   <body>
      <h1>This is the body of the sample view</h1>	
   </body>
</html>
```

基于 JSP 的视图文件位于项目的/ `WEB-INF`文件夹下，因此它们只能被 Spring 基础设施访问，而不能通过直接的 URL 访问。

## 6。带引导的弹簧 MVC

Spring Boot 是对 Spring 平台的一个补充，它使入门和创建独立的生产级应用程序变得非常容易。 **`Boot`并不是为了取代 Spring，而是为了让使用它更快更容易。**

### 6.1。Spring Boot 首发

新框架提供了方便的启动依赖，这是依赖描述符,可以为某个功能引入所有必要的技术。

这样做的好处是，我们不再需要为每个依赖项指定一个版本，而是允许启动者为我们管理依赖项。

最快的开始方式是添加[弹簧-启动-启动-父母](https://web.archive.org/web/20221228013248/https://search.maven.org/classic/#search%7Cga%7C1%7Ca%3A%22spring-boot-starter-parent%22) `pom.xml`:

```java
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.7.2</version>
</parent>
```

这将负责依赖性管理。

### 6.2。Spring Boot 入境点

使用`Spring Boot`构建的每个应用程序只需要定义主入口点。

这通常是一个带有`main`方法的 Java 类，用`@SpringBootApplication`注释:

```java
@SpringBootApplication
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
} 
```

此注释添加了以下其他注释:

*   `@Configuration` 将类标记为 bean 定义的来源。
*   `@EnableAutoConfiguration`告诉框架根据类路径上的依赖关系自动添加 beans。
*   `@ComponentScan`扫描与`Application`类或更低类相同的包中的其他配置和 beans。

有了 Spring Boot，我们可以使用百里香或 JSP 设置前端，而不需要使用第 3 节中定义的 ViewResolver。通过向我们的 pom.xml 添加`spring-boot-starter-thymeleaf`依赖项，百里香叶被启用，并且不需要额外的配置。

引导应用的源代码一如既往地在 GitHub 上[可用。](https://web.archive.org/web/20221228013248/https://github.com/eugenp/tutorials/tree/master/spring-boot-modules/spring-boot-bootstrap)

最后，如果你想从 Spring Boot 开始，看看我们这里的参考介绍。

## 7 .**。结论**

在本文中，我们使用 Java 配置配置了一个简单而实用的 Spring MVC 项目。

这个 Spring MVC 教程的实现可以在[GitHub 项目](https://web.archive.org/web/20221228013248/https://github.com/eugenp/tutorials/tree/master/spring-web-modules/spring-mvc-basics "Spring MVC Tutorial")中找到。

当项目在本地运行时，可以在`[http://localhost:8080/spring-mvc-basics/sample](https://web.archive.org/web/20221228013248/http://localhost:8080/spring-mvc-basics/sample)`访问`sample.jsp `。