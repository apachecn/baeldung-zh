# 春天和 Spring Boot 的比较

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-vs-spring-boot>

## 1。概述

在本教程中，我们将看看标准 Spring 框架和 Spring Boot 之间的区别。

我们将关注并讨论 Spring 的模块，比如 MVC 和 Security，在 core Spring 和 Boot 中使用时有何不同。

## 延伸阅读:

## [配置 Spring Boot 网络应用](/web/20221115195203/https://www.baeldung.com/spring-boot-application-configuration)

Some of the more useful configs for a Spring Boot application.[Read more](/web/20221115195203/https://www.baeldung.com/spring-boot-application-configuration) →

## 从春天迁徙到 Spring Boot

See how to properly migrate from a Spring to Spring Boot.[Read more](/web/20221115195203/https://www.baeldung.com/spring-boot-migration) →

## 2。春天是什么？

**简单来说，Spring 框架为开发 Java 应用提供了全面的基础设施支持**。

它包含一些很好的特性，如依赖注入，以及一些现成的模块，如:

*   春天的 JDBC
*   Spring MVC
*   春天安全
*   春季 AOP
*   弹簧 ORM
*   弹簧试验

这些模块可以大大减少应用程序的开发时间。

例如，在 Java web 开发的早期，我们需要编写大量样板代码来将记录插入数据源。通过使用 Spring JDBC 模块的`JDBCTemplate`,我们可以将它减少到只有几行代码和几个配置。

## 3。什么是 Spring Boot？

Spring Boot 基本上是 Spring 框架的扩展，它消除了设置 Spring 应用程序所需的样板配置。

它对 Spring 平台有自己的看法，为更快、更高效的开发生态系统铺平了道路。

以下是 Spring Boot 的一些特色:

*   固执己见的“初学者”依赖项，以简化构建和应用程序配置
*   嵌入式服务器可避免应用部署的复杂性
*   指标、健康检查和外部化配置
*   Spring 功能的自动配置——尽可能

让我们一步步熟悉这两个框架。

## 4。Maven 依赖关系

首先，让我们看看使用 Spring 创建 web 应用程序所需的最少依赖项:

```
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-web</artifactId>
    <version>5.3.5</version>
</dependency>
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-webmvc</artifactId>
    <version>5.3.5</version>
</dependency>
```

与 Spring 不同，Spring Boot 只需要一个依赖项就可以启动并运行一个 web 应用程序:

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
    <version>2.4.4</version>
</dependency>
```

在构建期间，所有其他依赖项都会自动添加到最终归档文件中。

另一个很好的例子是测试库。我们通常使用 Spring Test、JUnit、Hamcrest 和 Mockito 库的集合。在 Spring 项目中，我们应该添加所有这些库作为依赖项。

或者，在 Spring Boot，我们只需要测试的 starter 依赖项来自动包含这些库。

Spring Boot 为不同的 Spring 模块提供了大量的启动依赖。一些最常用的是:

*   `spring-boot-starter-data-jpa`
*   `spring-boot-starter-security`
*   `spring-boot-starter-test`
*   `spring-boot-starter-web`
*   `spring-boot-starter-thymeleaf`

关于首发的完整列表，也请查看 [Spring 文档](https://web.archive.org/web/20221115195203/https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#using-boot-starter)。

## 5。MVC 配置

让我们探索使用 Spring 和 Spring Boot 创建 JSP web 应用程序所需的配置。

**Spring 需要定义 dispatcher servlet、映射和其他支持配置。**我们可以使用`web.xml`文件或`Initializer`类来完成这项工作:

```
public class MyWebAppInitializer implements WebApplicationInitializer {

    @Override
    public void onStartup(ServletContext container) {
        AnnotationConfigWebApplicationContext context
          = new AnnotationConfigWebApplicationContext();
        context.setConfigLocation("com.baeldung");

        container.addListener(new ContextLoaderListener(context));

        ServletRegistration.Dynamic dispatcher = container
          .addServlet("dispatcher", new DispatcherServlet(context));

        dispatcher.setLoadOnStartup(1);
        dispatcher.addMapping("/");
    }
}
```

我们还需要将`@EnableWebMvc`注释添加到`@Configuration`类中，并定义一个视图解析器来解析从控制器返回的视图:

```
@EnableWebMvc
@Configuration
public class ClientWebConfig implements WebMvcConfigurer { 
   @Bean
   public ViewResolver viewResolver() {
      InternalResourceViewResolver bean
        = new InternalResourceViewResolver();
      bean.setViewClass(JstlView.class);
      bean.setPrefix("/WEB-INF/view/");
      bean.setSuffix(".jsp");
      return bean;
   }
}
```

相比之下，**一旦我们添加了 web starter，**Spring Boot 只需要几个属性就可以让它工作了

```
spring.mvc.view.prefix=/WEB-INF/jsp/
spring.mvc.view.suffix=.jsp
```

**以上所有的 Spring 配置都是通过一个叫做[自动配置](https://web.archive.org/web/20221115195203/https://docs.spring.io/spring-boot/docs/current/reference/html/using-boot-auto-configuration.html)的过程添加 Boot web starter 自动包含的。**

这意味着 Spring Boot 将查看应用程序中存在的依赖项、属性和 beans，并基于这些启用配置。

当然，如果我们想添加我们自己的定制配置，那么 Spring Boot 自动配置将后退。

### 5.1.配置模板引擎

现在让我们学习如何在 Spring 和 Spring Boot 中配置一个`Thymeleaf`模板引擎。

在 Spring 中，我们需要为视图解析器添加 [`thymeleaf-spring5`](https://web.archive.org/web/20221115195203/https://mvnrepository.com/artifact/org.thymeleaf/thymeleaf-spring5) 依赖和一些配置:

```
@Configuration
@EnableWebMvc
public class MvcWebConfig implements WebMvcConfigurer {

    @Autowired
    private ApplicationContext applicationContext;

    @Bean
    public SpringResourceTemplateResolver templateResolver() {
        SpringResourceTemplateResolver templateResolver = 
          new SpringResourceTemplateResolver();
        templateResolver.setApplicationContext(applicationContext);
        templateResolver.setPrefix("/WEB-INF/views/");
        templateResolver.setSuffix(".html");
        return templateResolver;
    }

    @Bean
    public SpringTemplateEngine templateEngine() {
        SpringTemplateEngine templateEngine = new SpringTemplateEngine();
        templateEngine.setTemplateResolver(templateResolver());
        templateEngine.setEnableSpringELCompiler(true);
        return templateEngine;
    }

    @Override
    public void configureViewResolvers(ViewResolverRegistry registry) {
        ThymeleafViewResolver resolver = new ThymeleafViewResolver();
        resolver.setTemplateEngine(templateEngine());
        registry.viewResolver(resolver);
    }
}
```

Spring Boot 1 只需要依赖`spring-boot-starter-thymeleaf `来启用 web 应用中的`Thymeleaf` 支持。由于`Thymeleaf3.0, `中的新特性，我们还必须添加`thymeleaf-layout-dialect `作为 Spring Boot 2 web 应用程序中的依赖项。或者，我们可以选择添加一个`spring-boot-starter-thymeleaf`依赖项来为我们处理所有这些事情。

一旦依赖项就位，我们可以将模板添加到`src/main/resources/templates`文件夹，Spring Boot 将自动显示它们。

## 6。Spring 安全配置

为了简单起见，我们将看看如何使用这些框架来启用默认的 HTTP 基本身份验证。

让我们从查看使用 Spring 实现安全性所需的依赖项和配置开始。

**Spring 需要标准的`spring-security-web`和`spring-security-config`依赖关系**来设置应用程序的安全性。

接下来**我们需要添加一个扩展`WebSecurityConfigurerAdapter`并使用`@EnableWebSecurity`** 注释的类:

```
@Configuration
@EnableWebSecurity
public class CustomWebSecurityConfigurerAdapter extends WebSecurityConfigurerAdapter {

    @Autowired
    public void configureGlobal(AuthenticationManagerBuilder auth) throws Exception {
        auth.inMemoryAuthentication()
          .withUser("user1")
            .password(passwordEncoder()
            .encode("user1Pass"))
          .authorities("ROLE_USER");
    }

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.authorizeRequests()
          .anyRequest().authenticated()
          .and()
          .httpBasic();
    }

    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }
}
```

这里我们使用`inMemoryAuthentication`来设置认证。

Spring Boot 也需要这些依赖项才能工作，但是我们只需要定义 **`spring-boot-starter-security`的依赖项，因为这会自动将所有相关的依赖项添加到类路径中。**

Spring Boot 的安全配置与上图相同。

要了解 JPA 配置如何在 Spring 和 Spring Boot 中实现，我们可以查看我们的文章[使用 Spring 的 JPA 指南](/web/20221115195203/https://www.baeldung.com/the-persistence-layer-with-spring-and-jpa)。

## 7。应用引导程序

在 Spring 和 Spring Boot 中引导应用程序的基本区别在于 servlet。Spring 使用`web.xml`或`SpringServletContainerInitializer `作为它的引导入口点。

另一方面，Spring Boot 只使用 Servlet 3 特性来引导应用程序。这个就详细说一下吧。

### 7.1。春天是怎样的？

Spring 既支持传统的`web.xml`自举方式，也支持最新的 Servlet 3+方法。

让我们看看`web.xml`方法的步骤:

1.  Servlet 容器(服务器)读取`web.xml.`
2.  在`web.xml`中定义的`DispatcherServlet`由容器实例化。
3.  `DispatcherServlet`通过阅读`WEB-INF/{servletName}-servlet.xml.`创造`WebApplicationContext`
4.  最后，`DispatcherServlet`注册应用程序上下文中定义的 beans。

下面是 Spring 如何使用 Servlet 3+方法启动:

1.  容器搜索实现`ServletContainerInitializer` 的类并执行。
2.  `SpringServletContainerInitializer`找到所有实现`WebApplicationInitializer.`的类
3.  `WebApplicationInitializer` 用 XML 或`@Configuration`类创建上下文。
4.  `WebApplicationInitializer` 用先前创建的上下文创建`DispatcherServlet `。

### 7.2。Spring Boot 是如何白手起家的？

**Spring Boot 应用程序的入口点是用`@SpringBootApplication` :** 标注的类

```
@SpringBootApplication
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```

默认情况下，Spring Boot 使用嵌入式容器来运行应用程序。在这种情况下，Spring Boot 使用`public static void main`入口点来启动一个嵌入式 web 服务器。

它还负责将应用程序上下文中的`Servlet, Filter,`和`ServletContextInitializer`bean 绑定到嵌入式 servlet 容器。

Spring Boot 的另一个特性是，它会自动扫描主类的同一个包或子包中的所有类的组件。

此外，Spring Boot 提供了在外部容器中将其部署为 web 归档的选项。在这种情况下，我们必须扩展`SpringBootServletInitializer`:

```
@SpringBootApplication
public class Application extends SpringBootServletInitializer {
    // ...
}
```

在这里，外部 servlet 容器寻找在 web 档案的 META-INF 文件中定义的主类，`SpringBootServletInitializer`将负责绑定`Servlet, Filter,`和`ServletContextInitializer.`

## 8。打包和部署

最后，让我们看看如何打包和部署应用程序。这两个框架都支持常见的包管理技术，如 Maven 和 Gradle 然而，当涉及到部署时，这些框架有很大的不同。

例如， [Spring Boot Maven 插件](https://web.archive.org/web/20221115195203/https://docs.spring.io/spring-boot/docs/current/maven-plugin/reference/htmlsingle/)在 Maven 中提供 Spring Boot 支持。它还允许打包可执行的 jar 或 war 档案，并“就地”运行应用程序

在部署方面，Spring Boot 相对于 Spring 的一些优势包括:

*   提供嵌入式容器支持
*   使用命令`java -jar`独立运行 jar
*   在外部容器中部署时，排除依赖性以避免潜在 jar 冲突的选项
*   部署时指定活动配置文件的选项
*   集成测试的随机端口生成

## 9。结论

在这篇文章中，我们了解了春天和 Spring Boot 的区别。

简而言之，我们可以说 Spring Boot 只是 Spring 本身的一个扩展，使开发、测试和部署更加方便。