# Spring MVC 中的 ViewResolver 指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-mvc-view-resolver-tutorial>

## 1。概述

所有 MVC 框架都提供了一种处理视图的方式。

Spring 通过视图解析器来实现这一点，视图解析器使您能够在浏览器中呈现模型，而无需将实现绑定到特定的视图技术。

`ViewResolver`将视图名称映射到实际视图。

Spring 框架附带了相当多的视图解析器，例如`InternalResourceViewResolver`、`BeanNameViewResolver,`和其他一些。

这是一个简单的教程，展示了如何设置最常见的视图解析器以及如何在同一个配置中使用多个`ViewResolver`。

## 2。弹簧腹板配置

先说 web 配置；我们用`@EnableWebMvc`、`@Configuration`和`@ComponentScan`来标注:

```
@EnableWebMvc
@Configuration
@ComponentScan("com.baeldung.web")
public class WebConfig implements WebMvcConfigurer {
    // All web configuration will go here
}
```

在这里，我们将在配置中设置视图解析器。

## 3。`InternalResourceViewResolver`加一个

这个`ViewResolver`允许我们为视图名设置前缀或后缀等属性，以生成最终的视图页面 URL:

```
@Bean
public ViewResolver internalResourceViewResolver() {
    InternalResourceViewResolver bean = new InternalResourceViewResolver();
    bean.setViewClass(JstlView.class);
    bean.setPrefix("/WEB-INF/view/");
    bean.setSuffix(".jsp");
    return bean;
}
```

对于这样简单的 的例子，我们不需要控制器来处理请求。

我们只需要一个简单的`jsp`页面，放置在配置中定义的`/WEB-INF/view`文件夹中:

```
<html>
    <head></head>
    <body>
        <h1>This is the body of the sample view</h1>
    </body>
</html>
```

## 4。`BeanNameViewResolver`加一个

这是 ViewResovler 的一个实现，它将视图名解释为当前应用程序上下文中的 bean 名。每个这样的`View`都可以被定义为 XML 或 Java 配置中的一个 bean。

首先，我们将`BeanNameViewResolver` 添加到之前的配置中:

```
@Bean
public BeanNameViewResolver beanNameViewResolver(){
    return new BeanNameViewResolver();
}
```

一旦定义了 ViewResolver，我们需要定义类型为`View`的 beans，这样它就可以被`DispatcherServlet `执行来呈现视图:

```
@Bean
public View sample() {
    return new JstlView("/WEB-INF/view/sample.jsp");
}
```

下面是控制器类中相应的处理程序方法:

```
@GetMapping("/sample")
public String showForm() {
    return "sample";
}
```

在控制器方法中，视图名称以"`sample”` 的形式返回，这意味着来自这个处理程序方法的视图解析为 URL 为`/WEB-INF/view/sample.jsp`的 JstlView 类。

## 5。链接`ViewResolvers`并定义订单优先级

Spring MVC 也支持**多视图解析器**。

这允许您在某些情况下覆盖特定视图。我们可以通过在配置中添加多个解析器来简单地链式查看解析器。

完成后，我们需要为这些解析器定义一个顺序。 **`order`属性**用于定义链中调用的顺序。order 属性越高(最大订单号)，视图解析程序在链中的位置就越靠后。

要定义顺序，我们可以将以下代码行添加到我们的视图解析器的配置中:

```
bean.setOrder(0);
```

注意顺序优先级，因为`InternalResourceViewResolver`应该有更高的顺序——因为它旨在表示非常明确的映射。如果其他解析器有更高的顺序，那么`InternalResourceViewResolver`可能永远不会被调用。

## 6.使用 Spring Boot

当使用 Spring Boot 时， **WebMvcAutoConfiguration** 自动配置我们的应用程序上下文`. `中的`**InternalResourceViewResolver** `和**`BeanNameViewResolver`** bean

此外，为模板引擎添加相应的启动器可以减少我们必须进行的大量手动配置。

例如，通过向我们的 pom.xml 添加 [`spring-boot-starter-thymeleaf`](https://web.archive.org/web/20220812064115/https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter-thymeleaf) 依赖项，百里香叶被启用，并且不需要额外的配置:

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-thymeleaf</artifactId>
    <version>${spring-boot-starter-thymeleaf.version}</version>
</dependency>
```

这个 starter 依赖项在我们的应用程序上下文中配置了名为`thymeleafViewResolver`的**百里香视图解析器** bean。我们可以通过提供一个同名的 bean 来覆盖自动配置的 ThymeleafViewResolver。

百里香叶视图解析器的工作原理是用前缀和后缀包围视图名称。前缀和后缀的默认值是' classpath:/templates/'和'。html '，分别为。

Spring Boot 还提供了一个选项，通过分别设置`spring.thymeleaf.prefix`和`spring.thymeleaf.suffix`属性来改变前缀和后缀的默认值。

类似地，我们有对 [groovy 模板](https://web.archive.org/web/20220812064115/https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter-groovy-templates)、 [freemarker](https://web.archive.org/web/20220812064115/https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter-freemarker) 和 [mustache](https://web.archive.org/web/20220812064115/https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter-mustache) 模板引擎的启动依赖，我们可以使用它们来获得使用 Spring Boot 自动配置的相应视图解析器。

使用它在应用程序上下文中找到的所有视图解析器，并尝试每一个，直到它得到一个结果，因此，如果我们计划添加我们自己的视图解析器，这些视图解析器的排序变得非常重要。

## 7 .**。结论**

在本教程中，我们使用 Java 配置配置了一系列视图解析器。通过处理顺序优先级，我们可以设置它们的调用顺序。

这个简单教程的实现可以在 [github 项目](https://web.archive.org/web/20220812064115/https://github.com/eugenp/tutorials/tree/master/spring-web-modules/spring-mvc-basics)中找到。