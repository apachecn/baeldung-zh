# Spring MVC 中的 HandlerAdapters

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-mvc-handler-adapters>

## 1。概述

在本文中，我们将关注 Spring 框架中可用的各种处理程序适配器实现。

## 2。什么是 Handleradapter？

`HandlerAdapter` 基本上是一个接口，它在 Spring MVC 中以非常灵活的方式促进了 HTTP 请求的处理。

它与`HandlerMapping`一起使用，将一个方法映射到一个特定的 URL。

然后，`DispatcherServlet` 使用一个`HandlerAdapter` 来调用这个方法。servlet 不直接调用该方法——它基本上充当了自身和处理程序对象之间的桥梁，导致了松散耦合的设计。

让我们来看看这个界面中可用的各种方法:

```java
public interface HandlerAdapter {
    boolean supports(Object handler);

    ModelAndView handle(
      HttpServletRequest request,
      HttpServletResponse response, 
      Object handler) throws Exception;

    long getLastModified(HttpServletRequest request, Object handler);
}
```

`supports` API 用于检查是否支持特定的处理程序实例。在调用该接口的`handle()` 方法之前，应该先调用该方法，以确定是否支持 handler 实例。

`handle` API 用于处理特定的 HTTP 请求。该方法负责通过将`HttpServletRequest`和`HttpServletResponse`对象作为参数传递来调用处理程序。处理程序然后执行应用程序逻辑并返回一个`ModelAndView`对象，然后由`DispatcherServlet`处理。

## 3。Maven 依赖关系

让我们从需要添加到`pom.xml`的 Maven 依赖项开始:

```java
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-webmvc</artifactId>
    <version>5.2.8.RELEASE</version>
</dependency>
```

最新版本的`spring-webmvc`神器可以在[这里](https://web.archive.org/web/20220628070510/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.springframework%22%20AND%20a%3A%22spring-webmvc%22)找到。

## 4。`HandlerAdapter`种类

### 4.1。`SimpleControllerHandlerAdapter`

这是 Spring MVC 注册的默认处理程序适配器。它处理实现`Controller`接口的类，并用于将请求转发给控制器对象。

如果 web 应用程序只使用控制器，那么我们不需要配置任何`HandlerAdapter`，因为框架使用这个类作为处理请求的默认适配器。

让我们定义一个简单的控制器类，使用旧风格的控制器(实现`Controller`接口):

```java
public class SimpleController implements Controller {
    @Override
    public ModelAndView handleRequest(
      HttpServletRequest request, 
      HttpServletResponse response) throws Exception {

        ModelAndView model = new ModelAndView("Greeting");
        model.addObject("message", "Dinesh Madhwal");
        return model;
    }
}
```

类似的 XML 配置:

```java
<beans ...>
    <bean name="/greeting.html"
      class="com.baeldung.spring.controller.SimpleControllerHandlerAdapterExample"/>
    <bean id="viewResolver"
      class="org.springframework.web.servlet.view.InternalResourceViewResolver">
        <property name="prefix" value="/WEB-INF/" />
        <property name="suffix" value=".jsp" />
    </bean>
</beans>
```

`BeanNameUrlHandlerMapping`类是这个处理程序适配器的映射类。

**注意**:如果在`BeanFactory,` 中定义了自定义处理器适配器，那么该适配器不会自动注册。因此，我们需要在上下文中明确地定义它。如果它没有被定义，而我们已经定义了一个自定义的处理程序适配器，那么我们将得到一个异常，表明没有为处理程序指定适配器。

### 4.2。`SimpleServletHandlerAdapter`

这个处理程序适配器允许使用任何`Servlet`与`DispatcherServlet`一起处理请求。它通过调用它的`service()` 方法`.`将来自`DispatcherServlet` 的请求转发给适当的`Servlet`类

实现`Servlet` 接口的 beans 由这个适配器自动处理。默认情况下它没有注册，我们需要像其他普通 bean 一样在`DispatcherServlet`的配置文件中注册它:

```java
<bean name="simpleServletHandlerAdapter" 
  class="org.springframework.web.servlet.handler.SimpleServletHandlerAdapter" />
```

### 4.3。`AnnotationMethodHandlerAdapter`

这个适配器类用于执行用`@RequestMapping` 注释标注的方法。它用于映射基于 HTTP 方法和 HTTP 路径的方法。

这个适配器的映射类是`DefaultAnnotationHandlerMapping,` ，用于在类型级别处理`@RequestMapping`注释，而`AnnotationMethodHandlerAdaptor` 用于在方法级别处理。

当初始化`DispatcherServlet`时，这两个类已经被框架注册了。但是，如果已经定义了其他处理程序适配器，那么我们需要在配置文件中也定义它。

让我们定义一个控制器类:

```java
@Controller
public class AnnotationHandler {
    @RequestMapping("/annotedName")
    public ModelAndView getEmployeeName() {
        ModelAndView model = new ModelAndView("Greeting");        
        model.addObject("message", "Dinesh");       
        return model;  
    }  
}
```

`@Controller`注释表明这个类充当`controller.`的角色

`@RequestMapping` 注释将`getEmployeeName()`方法映射到 URL `/name.`

根据应用程序是使用基于 Java 的配置还是基于 XML 的配置，有两种不同的方法来配置这个适配器。让我们看看使用 Java 配置的第一种方式:

```java
@ComponentScan("com.baeldung.spring.controller")
@Configuration
@EnableWebMvc
public class ApplicationConfiguration implements WebMvcConfigurer {
    @Bean
    public InternalResourceViewResolver jspViewResolver() {
        InternalResourceViewResolver bean = new InternalResourceViewResolver();
        bean.setPrefix("/WEB-INF/");
        bean.setSuffix(".jsp");
        return bean;
    }
}
```

如果应用程序使用 XML 配置，那么有两种不同的方法在 web 应用程序上下文 XML 中配置这个处理程序适配器。让我们看看文件`spring-servlet_AnnotationMethodHandlerAdapter.xml`中定义的第一种方法:

```java
<beans ...>
    <context:component-scan base-package="com.baeldung.spring.controller" />
    <bean 
      class="org.springframework.web.servlet.mvc.annotation.DefaultAnnotationHandlerMapping"/>
    <bean 
      class="org.springframework.web.servlet.mvc.annotation.AnnotationMethodHandlerAdapter"/>
    <bean id="viewResolver"
      class="org.springframework.web.servlet.view.InternalResourceViewResolver">
        <property name="prefix" value="/WEB-INF/" />
        <property name="suffix" value=".jsp" />
    </bean>
</beans>
```

`<context:component-scan />`标签用于指定要扫描的包的`controller`类。

让我们来看看第二种方法:

```java
<beans ...>
    <mvc:annotation-driven/>
    <context:component-scan base-package="com.baeldung.spring.controller" />
    <bean id="viewResolver"
      class="org.springframework.web.servlet.view.InternalResourceViewResolver">
        <property name="prefix" value="/WEB-INF/" />
        <property name="suffix" value=".jsp" />
    </bean>
</beans>
```

`<mvc:annotation-driven>`标签会自动向 spring MVC 注册这两个类。这个适配器在 Spring 3.2 中被弃用，Spring 3.1 引入了一个名为`RequestMappingHandlerAdapter` 的新处理程序适配器。

### 4.4。`RequestMappingHandlerAdapter`

这个适配器类是在 Spring 3.1 中引入的，取代了 Spring 3.2 中的`AnnotationMethodHandlerAdaptor` 处理程序适配器。

与`RequestMappingHandlerMapping` 类一起使用，由**执行用`@RequestMapping`** 标注的方法。

`RequestMappingHandlerMapping`用于维护请求 URI 到处理程序的映射。一旦获得了处理程序，`DispatcherServlet` 将请求分派给适当的处理程序适配器，然后该适配器调用`handlerMethod().`

在 3.1 之前的 Spring 版本中，类型级和方法级映射是在两个不同的阶段中处理的。

第一阶段是通过`DefaultAnnotationHandlerMapping`选择控制器，第二阶段是通过`AnnotationMethodHandlerAdapter`调用实际方法。

从 Spring 版本开始，只有一个阶段，涉及到识别控制器以及需要调用哪个方法来处理请求。

让我们定义一个简单的控制器类:

```java
@Controller
public class RequestMappingHandler {

    @RequestMapping("/requestName")
    public ModelAndView getEmployeeName() {
        ModelAndView model = new ModelAndView("Greeting");        
        model.addObject("message", "Madhwal");        
        return model;  
    }  
}
```

根据应用程序是使用基于 Java 的配置还是基于 XML 的配置，有两种不同的方法来配置这个适配器。

让我们看看使用 Java 配置的第一种方式:

```java
@ComponentScan("com.baeldung.spring.controller")
@Configuration
@EnableWebMvc
public class ServletConfig implements WebMvcConfigurer {
    @Bean
    public InternalResourceViewResolver jspViewResolver() {
        InternalResourceViewResolver bean = new InternalResourceViewResolver();
        bean.setPrefix("/WEB-INF/");
        bean.setSuffix(".jsp");
        return bean;
    }
}
```

如果应用程序使用 XML 配置，那么有两种不同的方法在 web 应用程序上下文 XML 中配置这个处理程序适配器。让我们看看文件`spring-servlet_RequestMappingHandlerAdapter.xml`中定义的第一种方法:

```java
<beans ...>
    <context:component-scan base-package="com.baeldung.spring.controller" />

    <bean 
      class="org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerMapping"/>

    <bean
      class="org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerAdapter"/>

    <bean id="viewResolver"
      class="org.springframework.web.servlet.view.InternalResourceViewResolver">
        <property name="prefix" value="/WEB-INF/" />
        <property name="suffix" value=".jsp" />
    </bean>
</beans>
```

这是第二种方法:

```java
<beans ...>
    <mvc:annotation-driven />

    <context:component-scan base-package="com.baeldung.spring.controller" />

    <bean id="viewResolver"
      class="org.springframework.web.servlet.view.InternalResourceViewResolver">
        <property name="prefix" value="/WEB-INF/" />
        <property name="suffix" value=".jsp" />
    </bean>
</beans>
```

这个标签会自动向 Spring MVC 注册这两个类。

如果我们需要定制`RequestMappingHandlerMapping,` ，那么我们需要从应用程序上下文 XML 中移除这个标签，并在应用程序上下文 XML 中手动配置它。

### 4.5。`HttpRequestHandlerAdapter`

这个处理程序适配器用于处理`HttpRequest`的处理程序。它实现了`HttpRequestHandler` 接口，该接口包含一个处理请求和生成响应的`handleRequest()` 方法。

此方法的返回类型是 void，它不会像其他处理程序适配器那样生成`ModelAndView`返回类型。它基本上用于生成二进制响应，并且不生成要呈现的视图。

## 5。运行应用程序

如果应用程序部署在端口号为`8082`的`localhost`上，并且上下文根为`spring-mvc-handlers`:

```java
http://localhost:8082/spring-mvc-handlers/
```

## 6。结论

在本文中，我们讨论了 Spring 框架中可用的各种类型的处理程序适配器。

大多数开发人员可能会坚持默认设置，但是当我们需要超越基础时，理解框架的灵活性是非常值得的。

本教程的源代码可以在 [GitHub 项目](https://web.archive.org/web/20220628070510/https://github.com/eugenp/tutorials/tree/master/spring-web-modules/spring-mvc-basics-2)中找到。