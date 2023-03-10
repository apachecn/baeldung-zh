# Apache Tiles 与 Spring MVC 的集成

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-mvc-apache-tiles>

## 1。概述

Apache [Tiles](https://web.archive.org/web/20220525130657/https://tiles.apache.org/) 是一个完全基于复合设计模式的免费开源模板框架。

复合设计模式是一种结构模式，它将对象组合成树形结构来表示整体-部分层次结构，这种模式统一处理单个对象和对象的组合。换句话说，在 Tiles 中，页面是由称为 Tiles 的子视图组合而成的。

该框架相对于其他框架的优势包括:

*   可重用性
*   易于配置
*   低性能开销

在本文中，我们将重点关注将 Apache Tiles 与 Spring MVC 集成在一起。

## 2。依赖配置

这里的第一步是在`pom.xml`中添加必要的[依赖关系](https://web.archive.org/web/20220525130657/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.apache.tiles%22%20AND%20a%3A%22tiles-jsp%22):

```java
<dependency>
    <groupId>org.apache.tiles</groupId>
    <artifactId>tiles-jsp</artifactId>
    <version>3.0.8</version>
</dependency>
```

## 3。平铺布局文件

现在我们需要定义模板定义，特别是对于每个页面，我们将覆盖该特定页面的模板定义:

```java
<tiles-definitions>
    <definition name="template-def" 
           template="/WEB-INF/views/tiles/layouts/defaultLayout.jsp">  
        <put-attribute name="title" value="" />  
        <put-attribute name="header" 
           value="/WEB-INF/views/tiles/templates/defaultHeader.jsp" />  
        <put-attribute name="menu" 
           value="/WEB-INF/views/tiles/templates/defaultMenu.jsp" />  
        <put-attribute name="body" value="" />  
        <put-attribute name="footer" 
           value="/WEB-INF/views/tiles/templates/defaultFooter.jsp" />  
    </definition>  
    <definition name="home" extends="template-def">  
        <put-attribute name="title" value="Welcome" />  
        <put-attribute name="body" 
           value="/WEB-INF/views/pages/home.jsp" />  
    </definition>  
</tiles-definitions>
```

## 4。`ApplicationConfiguration`和其他班级

作为配置的一部分，我们将创建三个名为`ApplicationInitializer`、`ApplicationController`和`ApplicationConfiguration`的特定 java 类:

*   `ApplicationInitializer`初始化并检查`ApplicationConfiguration`类中指定的必要配置
*   该类包含将 Spring MVC 与 Apache Tiles 框架集成的配置
*   `ApplicationController`类与`tiles.xml`文件同步工作，并根据传入的请求重定向到必要的页面

让我们看看每个类的运行情况:

```java
@Controller
@RequestMapping("/")
public class TilesController {
    @RequestMapping(
      value = { "/"}, 
      method = RequestMethod.GET)
    public String homePage(ModelMap model) {
        return "home";
    }
    @RequestMapping(
      value = { "/apachetiles"}, 
      method = RequestMethod.GET)
    public String productsPage(ModelMap model) {
        return "apachetiles";
    }

    @RequestMapping(
      value = { "/springmvc"},
      method = RequestMethod.GET)
    public String contactUsPage(ModelMap model) {
        return "springmvc";
    }
}
```

```java
public class WebInitializer implements WebApplicationInitializer {
 public void onStartup(ServletContext container) throws ServletException {

        AnnotationConfigWebApplicationContext ctx = new AnnotationConfigWebApplicationContext();

        ctx.register(TilesApplicationConfiguration.class);

        container.addListener(new ContextLoaderListener(ctx));

        ServletRegistration.Dynamic servlet = container.addServlet(
          "dispatcher", new DispatcherServlet(ctx));
        servlet.setLoadOnStartup(1);
        servlet.addMapping("/");
    }
}
```

在 Spring MVC 应用程序中，有两个重要的类在配置 tiles 时扮演着关键角色。他们是`TilesConfigurer`和`TilesViewResolver`:

*   `TilesConfigurer`通过提供 tiles 配置文件的路径，帮助链接 Tiles 框架和 Spring 框架
*   `TilesViewResolver`是 Spring API 提供的适配器类之一，用于解析 tiles 视图

最后，在`ApplicationConfiguration`类中，我们使用`TilesConfigurer`和`TilesViewResolver`类来实现集成:

```java
@Configuration
@EnableWebMvc
@ComponentScan(basePackages = "com.baeldung.spring.controller.tiles")
public class TilesApplicationConfiguration implements WebMvcConfigurer {
    @Bean
    public TilesConfigurer tilesConfigurer() {
        TilesConfigurer tilesConfigurer = new TilesConfigurer();
        tilesConfigurer.setDefinitions(
          new String[] { "/WEB-INF/views/**/tiles.xml" });
        tilesConfigurer.setCheckRefresh(true);

        return tilesConfigurer;
    }

    @Override
    public void configureViewResolvers(ViewResolverRegistry registry) {
        TilesViewResolver viewResolver = new TilesViewResolver();
        registry.viewResolver(viewResolver);
    }

    @Override
    public void addResourceHandlers(ResourceHandlerRegistry registry) {
        registry.addResourceHandler("/static/**")
          .addResourceLocations("/static/");
    }
}
```

## 5。瓷砖模板文件

到目前为止，我们已经完成了 Apache Tiles 框架的配置，以及整个应用程序中使用的模板和特定 Tiles 的定义。

在这一步，我们需要创建已经在 `tiles.xml`中定义的特定模板文件。

请查找可用作构建特定页面基础的布局片段:

```java
<html>
    <head>
        <meta 
          http-equiv="Content-Type" content="text/html; charset=ISO-8859-1">
        <title><tiles:getAsString name="title" /></title>
        <link href="<c:url value='/static/css/app.css' />" 
            rel="stylesheet">
        </link>
    </head>
    <body>
        <div class="flex-container">
            <tiles:insertAttribute name="header" />
            <tiles:insertAttribute name="menu" />
        <article class="article">
            <tiles:insertAttribute name="body" />
        </article>
        <tiles:insertAttribute name="footer" />
        </div>
    </body>
</html>
```

## 6。结论

这就结束了 Spring MVC 与 Apache Tiles 的集成。

你可以在 github 项目后面的[中找到完整的实现。](https://web.archive.org/web/20220525130657/https://github.com/eugenp/tutorials/tree/master/spring-web-modules/spring-mvc-views)