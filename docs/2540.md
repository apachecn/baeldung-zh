# Spring MVC HandlerInterceptor 简介

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-mvc-handlerinterceptor>

## 1.概观

在本教程中，我们将重点了解 Spring MVC `HandlerInterceptor`以及如何正确使用它。

## 2.Spring MVC 处理程序

为了理解 Spring 拦截器是如何工作的，让我们后退一步，看看 `HandlerMapping`。

`HandlerMapping`的目的是将一个处理器方法映射到一个 URL。这样，`DispatcherServlet`将能够在处理请求时调用它。

事实上，`DispatcherServlet`使用`HandlerAdapter`来实际调用该方法。

简而言之，拦截器拦截请求并处理它们。它们有助于避免重复的处理程序代码，如日志记录和授权检查。

现在我们已经了解了整体的上下文，让我们看看如何使用一个`HandlerInterceptor`来执行一些预处理和后处理动作。

## 3.Maven 依赖性

为了使用拦截器，我们需要在我们的`pom.xml`中包含 [`spring-web`](https://web.archive.org/web/20220812020546/https://search.maven.org/classic/#search%7Cga%7C1%7Ca%3A%22spring-web%22) 依赖项:

```
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-web</artifactId>
    <version>5.3.13</version>
</dependency> 
```

## 4.弹簧处理器拦截器

简单地说， **Spring 拦截器是一个扩展了`HandlerInterceptorAdapter`类或者实现了`HandlerInterceptor`接口的类。**

`HandlerInterceptor`包含三个主要方法:

*   `prehandle()`–在实际处理程序执行之前调用
*   `postHandle()`–在处理程序执行后调用
*   `afterCompletion()`——在完成完整请求并生成视图后调用

这三种方法为各种预处理和后处理提供了灵活性。

在我们继续之前，有一个小提示:要跳过理论直接跳到例子，直接跳到第 5 部分。

下面是一个简单的`preHandle()`实现:

```
@Override
public boolean preHandle(
  HttpServletRequest request,
  HttpServletResponse response, 
  Object handler) throws Exception {
    // your code
    return true;
}
```

注意，该方法返回一个`boolean`值。它告诉 Spring 进一步处理请求(`true`)或不处理请求(`false`)。

接下来，我们有一个`postHandle()`的实现:

```
@Override
public void postHandle(
  HttpServletRequest request, 
  HttpServletResponse response,
  Object handler, 
  ModelAndView modelAndView) throws Exception {
    // your code
}
```

**拦截器在处理请求之后，但在生成视图之前，立即调用这个方法。**

例如，我们可以使用这种方法将登录用户的头像添加到模型中。

我们需要实现的最后一个方法是`afterCompletion()`:

```
@Override
public void afterCompletion(
  HttpServletRequest request, 
  HttpServletResponse response,
  Object handler, Exception ex) {
    // your code
}
```

这个方法允许我们在请求处理完成后执行定制逻辑。

此外，值得一提的是，我们可以注册多个自定义拦截器。为此，我们可以使用 [`DefaultAnnotationHandlerMapping`](https://web.archive.org/web/20220812020546/https://docs.spring.io/spring-framework/docs/4.3.7.RELEASE_to_4.3.8.RELEASE/Spring%20Framework%204.3.8.RELEASE/org/springframework/web/servlet/mvc/annotation/DefaultAnnotationHandlerMapping.html) 。

## 5.自定义记录器拦截器

在本例中，我们将重点关注登录我们的 web 应用程序。

首先，我们的类需要实现`HandlerInterceptor`:

```
public class LoggerInterceptor implements HandlerInterceptor {
    ...
}
```

我们还需要在拦截器中启用日志记录:

```
private static Logger log = LoggerFactory.getLogger(LoggerInterceptor.class);
```

这允许`Log4J`显示日志，并指示哪个类当前正在将信息记录到指定的输出中。

接下来，让我们关注我们的定制拦截器实现。

### 5.1.`preHandle()` 方法

顾名思义，拦截器在处理请求之前调用`preHandle()`。

默认情况下，该方法返回`true` 将请求进一步发送到处理程序方法。但是，我们可以通过返回`false`来告诉 Spring 停止执行。

我们可以使用钩子来记录关于请求参数的信息，比如请求来自哪里。

在我们的例子中，我们使用一个简单的`Log4J`记录器记录这些信息:

```
@Override
public boolean preHandle(
  HttpServletRequest request,
  HttpServletResponse response, 
  Object handler) throws Exception {

    log.info("[preHandle][" + request + "]" + "[" + request.getMethod()
      + "]" + request.getRequestURI() + getParameters(request));

    return true;
} 
```

如我们所见，我们正在记录关于请求的一些基本信息。

当然，如果我们在这里遇到密码，我们需要确保不要记录下来。一个简单的选择是用星号代替密码和任何其他敏感类型的数据。

下面是如何做到这一点的快速实现:

```
private String getParameters(HttpServletRequest request) {
    StringBuffer posted = new StringBuffer();
    Enumeration<?> e = request.getParameterNames();
    if (e != null) {
        posted.append("?");
    }
    while (e.hasMoreElements()) {
        if (posted.length() > 1) {
            posted.append("&");
        }
        String curr = (String) e.nextElement();
        posted.append(curr + "=");
        if (curr.contains("password") 
          || curr.contains("pass")
          || curr.contains("pwd")) {
            posted.append("*****");
        } else {
            posted.append(request.getParameter(curr));
        }
    }
    String ip = request.getHeader("X-FORWARDED-FOR");
    String ipAddr = (ip == null) ? getRemoteAddr(request) : ip;
    if (ipAddr!=null && !ipAddr.equals("")) {
        posted.append("&_psip=" + ipAddr); 
    }
    return posted.toString();
}
```

最后，我们的目标是获取 HTTP 请求的源 IP 地址。

下面是一个简单的实现:

```
private String getRemoteAddr(HttpServletRequest request) {
    String ipFromHeader = request.getHeader("X-FORWARDED-FOR");
    if (ipFromHeader != null && ipFromHeader.length() > 0) {
        log.debug("ip from proxy - X-FORWARDED-FOR : " + ipFromHeader);
        return ipFromHeader;
    }
    return request.getRemoteAddr();
}
```

### 5.2.`postHandle()` 方法

拦截器在处理程序执行之后，但在`DispatcherServlet`呈现视图之前调用这个方法。

我们可以用它来给`ModelAndView`添加额外的属性。另一个用例是计算请求的处理时间。

在我们的例子中，我们将在`DispatcherServlet`呈现视图之前简单地记录我们的请求:

```
@Override
public void postHandle(
  HttpServletRequest request, 
  HttpServletResponse response,
  Object handler, 
  ModelAndView modelAndView) throws Exception {

    log.info("[postHandle][" + request + "]");
}
```

### 5.3.`afterCompletion()` 方法

我们可以使用这个方法在视图呈现后获得请求和响应数据:

```
@Override
public void afterCompletion(
  HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) 
  throws Exception {
    if (ex != null){
        ex.printStackTrace();
    }
    log.info("[afterCompletion][" + request + "][exception: " + ex + "]");
}
```

## 6.配置

现在我们已经把所有的部分放在一起，让我们添加我们的自定义拦截器。

为此，我们需要覆盖`addInterceptors()`方法:

```
@Override
public void addInterceptors(InterceptorRegistry registry) {
    registry.addInterceptor(new LoggerInterceptor());
}
```

我们可以通过编辑 XML Spring 配置文件来实现相同的配置:

```
<mvc:interceptors>
    <bean id="loggerInterceptor" class="com.baeldung.web.interceptor.LoggerInterceptor"/>
</mvc:interceptors>
```

这个配置激活后，拦截器也将激活，应用程序中的所有请求都将被正确记录。

请注意，如果配置了多个 Spring 拦截器，`preHandle()`方法按照配置顺序执行，而`postHandle()`和`afterCompletion()`方法按照相反的顺序调用。

**请记住，如果我们使用 Spring Boot 而不是 vanilla Spring，我们不需要用`@EnableWebMvc` 来注释我们的配置类。**

## 7.结论

本文简要介绍了如何使用 Spring MVC 处理程序拦截器来拦截 HTTP 请求。

所有的例子和配置都可以在 GitHub 的[上找到。](https://web.archive.org/web/20220812020546/https://github.com/eugenp/tutorials/tree/master/spring-security-modules/spring-security-web-mvc-custom)