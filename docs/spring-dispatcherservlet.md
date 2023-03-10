# Spring DispatcherServlet 简介

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-dispatcherservlet>

## 1。简介

简单地说，在`Front Controller`设计模式`,`中，单个控制器**负责将传入的`HttpRequests`导向应用程序的所有其他控制器和处理程序**。

Spring 的`DispatcherServlet`实现了这种模式，因此负责将`HttpRequests`正确地协调到它们正确的处理程序。

在本文中，我们将**研究 Spring `DispatcherServlet's` 请求处理工作流**以及如何实现参与该工作流的几个接口。

## 2。`DispatcherServlet`请求处理

从本质上讲， **`DispatcherServlet`处理传入的`HttpRequest`，委托请求，并根据已配置的`HandlerAdapter`接口**处理请求，这些接口已经在 Spring 应用程序中实现，并附带了指定处理程序、控制器端点和响应对象的注释。

让我们更深入地了解一下`DispatcherServlet` 如何处理组件:

*   搜索与关键字`DispatcherServlet.WEB_APPLICATION_CONTEXT_ATTRIBUTE`下的`DispatcherServlet`相关联的`WebApplicationContext`,并使其可用于流程的所有元素
*   `DispatcherServlet`使用 `getHandler() –` 找到为您的调度程序配置的`HandlerAdapter`接口的所有实现，每个找到并配置的实现在流程的剩余部分通过`handle()`处理请求
*   可选地，将`[LocaleResolver](#localResolver)`绑定到请求，以使流程中的元素能够解析语言环境
*   可选地，将`[ThemeResolver](#themeResolver)`绑定到请求，让元素(比如视图)决定使用哪个主题
*   如果指定了`[MultipartResolver](#multipartResolver)`,则检查请求的`MultipartFile`s——任何找到的请求都被包装在`MultipartHttpServletRequest` 中以供进一步处理
*   `[HandlerExceptionResolver](#handlerExceptionResolver)`在`WebApplicationContext`中声明的实现拾取在请求处理过程中抛出的异常

您可以在此了解更多关于注册和设立`DispatcherServlet` [的方法。](/web/20220628051234/https://www.baeldung.com/register-servlet)

## 3。 `HandlerAdapter`接口

`HandlerAdapter`接口通过几个特定的接口方便了控制器、servlets、`HttpRequests`和 HTTP 路径的使用。**`HandlerAdapter`接口因此在`DispatcherServlet`请求处理工作流**的许多阶段中扮演着重要的角色。

首先，每个`HandlerAdapter`实现都被放入调度程序的 `getHandler()` 方法的`HandlerExecutionChain`中。然后，随着执行链的进行，这些实现中的每一个`handle()`都会调用`HttpServletRequest`对象。

在接下来的章节中，我们将更详细地探讨几个最重要和最常用的`HandlerAdapters`。

### 3.1。映射

为了理解映射，我们需要首先看看如何注释控制器，因为控制器对于`HandlerMapping`接口是如此重要。

`SimpleControllerHandlerAdapter`允许在没有`@Controller`注释的情况下显式实现控制器。

`RequestMappingHandlerAdapter` 支持用`@RequestMapping` 注释`.`注释的方法

我们在这里将重点关注`@Controller`注释，但是也有一些有用的参考资料，包括几个使用`SimpleControllerHandlerAdapter` 的[示例。](/web/20220628051234/https://www.baeldung.com/spring-mvc-handler-adapters)

**`@RequestMapping`注释设置了一个特定的端点，在这个端点上，一个处理程序将在与之关联的`WebApplicationContext`内**可用。

让我们看一个公开和处理`‘/user/example'`端点的`Controller`的例子:

```java
@Controller
@RequestMapping("/user")
@ResponseBody
public class UserController {

    @GetMapping("/example")
    public User fetchUserExample() {
        // ...
    }
}
```

由`@RequestMapping`注释指定的路径通过`HandlerMapping`接口在内部管理。

**URL 结构自然与`DispatcherServlet`本身相关——由 servlet 映射决定。**

因此，如果`DispatcherServlet`被映射到“/”，那么所有映射都将被该映射覆盖。

但是，如果 servlet 映射是'`/dispatcher`'，那么任何@ `RequestMapping`注释都将与这个根 URL 相关。

**记住，对于 servlet 映射来说，/'与'/* '【T1]是不同的！/'是默认映射，将所有 URL 暴露给调度员的责任范围。**

/* '让很多 Spring 新手感到困惑。它没有指定具有相同 URL 上下文的所有路径都在调度程序的责任范围内。相反，它会覆盖并忽略其他调度程序映射。因此，'/example '将作为 404 出现！

因此， **'/* '不应该使用，除非在非常有限的情况下**(比如配置过滤器)。

### 3.2。HTTP 请求处理

**一个`DispatcherServlet`的核心职责是将传入的`HttpRequests`分派给正确的处理程序**，这些处理程序由`@Controller`或`@RestController`注释指定。

顺便提一下，`@Controller`和`@RestController`的主要区别在于响应是如何生成的——`@RestController`也默认定义了`@ResponseBody`。

我们对 Spring 控制器进行更深入研究的文章可以在这里找到。

### 3.3。`ViewResolver`界面

一个`ViewResolver`被附加到一个`DispatcherServlet`上，作为一个`ApplicationContext`对象上的配置设置。

**A `ViewResolver`决定调度员提供什么样的视图以及从哪里提供这些视图**。

下面是一个示例配置，我们将把它放入我们的 *AppConfig* 中，用于呈现 JSP 页面:

```java
@Configuration
@EnableWebMvc
@ComponentScan("com.baeldung.springdispatcherservlet")
public class AppConfig implements WebMvcConfigurer {

    @Bean
    public UrlBasedViewResolver viewResolver() {
        UrlBasedViewResolver resolver
          = new UrlBasedViewResolver();
        resolver.setPrefix("/WEB-INF/view/");
        resolver.setSuffix(".jsp");
        resolver.setViewClass(JstlView.class);
        return resolver;
    }
}
```

非常直接！这有三个主要部分:

1.  设置前缀，这将设置默认的 URL 路径，以便在其中查找集合视图
2.  通过后缀设置的默认视图类型
3.  在解析器上设置一个视图类，它允许像 JSTL 或瓦片这样的技术与呈现的视图相关联

**一个常见的问题是调度员的`ViewResolver`** **和整个项目目录结构**的关联有多精确。让我们来看看基本的。

下面是一个使用 Spring 的 XML 配置为`InternalViewResolver`配置路径的例子:

```java
<property name="prefix" value="/jsp/"/>
```

出于示例的目的，我们假设我们的应用程序托管在:

```java
http://localhost:8080/
```

这是本地托管的 Apache Tomcat 服务器的默认地址和端口。

假设我们的应用程序名为`dispatcherexample-1.0.0`，我们的 JSP 视图可以从以下位置访问:

```java
http://localhost:8080/dispatcherexample-1.0.0/jsp/
```

这些视图在 Maven 的普通 Spring 项目中的路径是这样的:

```java
src -|
     main -|
            java
            resources
            webapp -|
                    jsp
                    WEB-INF
```

视图的默认位置在 WEB-INF 中。在上面的代码片段中为我们的`InternalViewResolver`指定的路径决定了‘src/main/web app’的子目录，您的视图将在其中可用。

### 3.4。`LocaleResolver`界面

**为我们的调度程序定制会话、请求或 cookie 信息的主要方式是通过`LocaleResolver`接口**。

`CookieLocaleResolver` 是一个允许使用 cookies 配置无状态应用程序属性的实现。再加到`AppConfig`吧。

```java
@Bean
public CookieLocaleResolver cookieLocaleResolverExample() {
    CookieLocaleResolver localeResolver 
      = new CookieLocaleResolver();
    localeResolver.setDefaultLocale(Locale.ENGLISH);
    localeResolver.setCookieName("locale-cookie-resolver-example");
    localeResolver.setCookieMaxAge(3600);
    return localeResolver;
}

@Bean 
public LocaleResolver sessionLocaleResolver() { 
    SessionLocaleResolver localeResolver = new SessionLocaleResolver(); 
    localeResolver.setDefaultLocale(Locale.US); 
    localResolver.setDefaultTimeZone(TimeZone.getTimeZone("UTC"));
    return localeResolver; 
} 
```

`SessionLocaleResolver` 允许在有状态应用程序中进行特定于会话的配置。

`setDefaultLocale`()方法代表一个地理、政治或文化区域，而`setDefaultTimeZone` ( )确定相关应用`Bean`的相关`TimeZone`对象。

这两种方法在上述`LocaleResolver`的每个实现中都可用。

### 3.5。`ThemeResolver`界面

Spring 为我们的视图提供了风格化的主题。

让我们看看如何配置我们的 dispatcher 来处理主题。

首先，**让我们设置所有必要的配置来找到并使用我们的静态主题文件**。我们需要为我们的`ThemeSource`设置一个静态资源位置来配置实际的`Themes`本身(`Theme`对象包含那些文件中规定的所有配置信息)。将此添加到`AppConfig`:

```java
@Override
public void addResourceHandlers(ResourceHandlerRegistry registry) {
    registry.addResourceHandler("/resources/**")
      .addResourceLocations("/", "/resources/")
      .setCachePeriod(3600)
      .resourceChain(true)
      .addResolver(new PathResourceResolver());
}

@Bean
public ResourceBundleThemeSource themeSource() {
    ResourceBundleThemeSource themeSource
      = new ResourceBundleThemeSource();
    themeSource.setDefaultEncoding("UTF-8");
    themeSource.setBasenamePrefix("themes.");
    return themeSource;
} 
```

由`DispatcherServlet` 管理的请求可以通过一个指定的参数来修改主题，该参数被传递给`ThemeChangeInterceptor` 对象`.` 上可用的`setParamName`()添加到`AppConfig:` 

```java
@Bean
public CookieThemeResolver themeResolver() {
    CookieThemeResolver resolver = new CookieThemeResolver();
    resolver.setDefaultThemeName("example");
    resolver.setCookieName("example-theme-cookie");
    return resolver;
}

@Bean
public ThemeChangeInterceptor themeChangeInterceptor() {
   ThemeChangeInterceptor interceptor
     = new ThemeChangeInterceptor();
   interceptor.setParamName("theme");
   return interceptor;
}

@Override
public void addInterceptors(InterceptorRegistry registry) {
    registry.addInterceptor(themeChangeInterceptor());
} 
```

下面的 JSP 标记被添加到我们的视图中，以显示正确的样式:

```java
<link rel="stylesheet" href="${ctx}/<spring:theme code='styleSheet'/>" type="text/css"/>
```

下面的 URL 请求使用传递到我们配置的`ThemeChangeIntercepter:`中的‘theme’参数来呈现`example`主题

```java
http://localhost:8080/dispatcherexample-1.0.0/?theme=example
```

### 3.6。`MultipartResolver`界面

一个`MultipartResolver` 实现检查对多部分的请求，如果至少找到一个多部分，就将它们包装在一个`MultipartHttpServletRequest`中，供流程中的其他元素进一步处理。添加到`AppConfig`:

```java
@Bean
public CommonsMultipartResolver multipartResolver() 
  throws IOException {
    CommonsMultipartResolver resolver
      = new CommonsMultipartResolver();
    resolver.setMaxUploadSize(10000000);
    return resolver;
} 
```

现在我们已经配置了我们的`MultipartResolver` bean，让我们设置一个控制器来处理`MultipartFile`请求:

```java
@Controller
public class MultipartController {

    @Autowired
    ServletContext context;

    @PostMapping("/upload")
    public ModelAndView FileuploadController(
      @RequestParam("file") MultipartFile file) 
      throws IOException {
        ModelAndView modelAndView = new ModelAndView("index");
        InputStream in = file.getInputStream();
        String path = new File(".").getAbsolutePath();
        FileOutputStream f = new FileOutputStream(
          path.substring(0, path.length()-1)
          + "/uploads/" + file.getOriginalFilename());
        int ch;
        while ((ch = in.read()) != -1) {
            f.write(ch);
        }
        f.flush();
        f.close();
        in.close();
        modelAndView.getModel()
          .put("message", "File uploaded successfully!");
        return modelAndView;
    }
}
```

我们可以使用一个普通的表单向指定的端点提交一个文件。上传的文件将在“CATALINA_HOME/bin/uploads”中提供。

### 3.7。`HandlerExceptionResolver`界面

Spring 的`HandlerExceptionResolver`为整个 web 应用程序、单个控制器或一组控制器提供统一的错误处理。

**为了提供应用程序范围的自定义异常处理，创建一个用`@ControllerAdvice`** 注释的类:

```java
@ControllerAdvice
public class ExampleGlobalExceptionHandler {

    @ExceptionHandler
    @ResponseBody 
    public String handleExampleException(Exception e) {
        // ...
    }
}
```

该类中用`@ExceptionHandler`标注的任何方法都可以在 dispatcher 职责范围内的每个控制器上使用。

每当`@ExceptionHandler`被用作注释并且正确的类作为参数被传入时，`DispatcherServlet's ApplicationContext`中的`HandlerExceptionResolver`接口的实现可用于**拦截该调度程序负责区域**下的特定控制器**:**

```java
@Controller
public class FooController{

    @ExceptionHandler({ CustomException1.class, CustomException2.class })
    public void handleException() {
        // ...
    }
    // ...
}
```

如果异常`CustomException1`或`CustomException2`发生，`handleException()`方法现在将作为上面例子中`FooController` 的异常处理程序。

这里有一篇关于 Spring web 应用中异常处理的更深入的文章。

## 4。结论

在本教程中，我们回顾了 Spring 的`DispatcherServlet` 和几种配置它的方法。

和往常一样，本教程中使用的源代码可以从 Github 上的[处获得。](https://web.archive.org/web/20220628051234/https://github.com/eugenp/tutorials/tree/master/spring-web-modules/spring-mvc-basics)