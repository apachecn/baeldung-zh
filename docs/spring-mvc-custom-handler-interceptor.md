# 使用自定义 Spring MVC 的处理程序拦截器来管理会话

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-mvc-custom-handler-interceptor>

## 1。简介

在本教程中，我们将关注 Spring MVC `HandlerInterceptor.`

更具体地说，我们将展示使用拦截器的一个更高级的用例—**我们将通过设置自定义计数器和手动跟踪会话来模拟会话超时逻辑**。

如果你想了解春天的基本知识，看看这篇文章。

## 2。Maven 依赖关系

为了使用`Interceptors`，您需要在您的`pom.xml`文件的`dependencies`部分中包含以下部分:

```java
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-web</artifactId>
    <version>5.3.13</version>
</dependency>
```

最新版本可以在[这里](https://web.archive.org/web/20220828134825/https://search.maven.org/classic/#search%7Cga%7C1%7Ca%3A%22spring-web%22)找到。这个依赖项只涉及 Spring Web，所以不要忘记为一个完整(最小)的 Web 应用程序添加 s `pring-core`和`spring-context`。

## 3。会话超时的自定义实现

在本例中，我们将为系统中的用户配置最长非活动时间。此后，他们将自动从应用程序中注销。

这个逻辑**只是一个概念证明**——我们当然可以使用会话超时轻松地获得相同的结果——但是结果不是这里的重点，重点是拦截器的使用。

因此，我们希望确保如果用户不活动，会话将无效。例如，如果用户忘记注销，非活动时间计数器将防止未经授权的用户访问该帐户。为了做到这一点，我们需要为非活动时间设置常数:

```java
private static final long MAX_INACTIVE_SESSION_TIME = 5 * 10000;
```

出于测试目的，我们将其设置为 50 秒；别忘了，是以 ms 计算的。

现在，我们需要跟踪应用程序中的每个会话，所以我们需要包含这个 Spring 接口:

```java
@Autowired
private HttpSession session;
```

让我们继续使用`preHandle()`方法。

### 3.1。`preHandle()`

在该方法中，我们将包括以下操作:

*   设置计时器以检查请求的处理时间
*   检查用户是否登录(使用本文[文章](/web/20220828134825/https://www.baeldung.com/spring-model-parameters-with-handler-interceptor)中的`UserInterceptor`方法)
*   如果用户的非活动会话时间超过最大允许值，则自动注销

让我们看一下实现:

```java
@Override
public boolean preHandle(
  HttpServletRequest req, HttpServletResponse res, Object handler) throws Exception {
    log.info("Pre handle method - check handling start time");
    long startTime = System.currentTimeMillis();
    request.setAttribute("executionTime", startTime);
} 
```

在这部分代码中，我们设置了处理执行的`startTime`。从这一刻起，我们将数秒来完成每个请求的处理。在下一部分中，我们将提供会话时间的逻辑，前提是有人在 HTTP 会话期间登录:

```java
if (UserInterceptor.isUserLogged()) {
    session = request.getSession();
    log.info("Time since last request in this session: {} ms",
      System.currentTimeMillis() - request.getSession().getLastAccessedTime());
    if (System.currentTimeMillis() - session.getLastAccessedTime()
      > MAX_INACTIVE_SESSION_TIME) {
        log.warn("Logging out, due to inactive session");
        SecurityContextHolder.clearContext();
        request.logout();
        response.sendRedirect("/spring-rest-full/logout");
    }
}
return true; 
```

首先，我们需要从请求中获取会话。

接下来，我们做一些控制台日志记录，记录自从用户在我们的应用程序中执行任何操作以来，谁登录了，过了多长时间。我们可以使用`session.getLastAccessedTime()` 获得该信息，从当前时间中减去该信息，并与我们的`MAX_INACTIVE_SESSION_TIME.` 进行比较

如果时间比我们允许的长，我们清除上下文，注销请求，然后(可选地)发送一个重定向作为对默认注销视图的响应，这是在 Spring 安全配置文件中声明的。

为了完成处理时间示例的计数器，我们还实现了`postHandle()`方法，这将在下一小节中描述。

### 3.2。`postHandle()`

这个方法的实现只是为了获取信息，处理当前请求花了多长时间。正如您在前面的代码片段中看到的，我们在 Spring 模型中设置了`executionTime`。现在是时候使用它了:

```java
@Override
public void postHandle(
  HttpServletRequest request, 
  HttpServletResponse response,
  Object handler, 
  ModelAndView model) throws Exception {
    log.info("Post handle method - check execution time of handling");
    long startTime = (Long) request.getAttribute("executionTime");
    log.info("Execution time for handling the request was: {} ms",
      System.currentTimeMillis() - startTime);
}
```

实现很简单——我们检查一个执行时间，然后从当前系统时间中减去它。记得把模型的值强制转换成`long`就行了。

现在我们可以正确地记录执行时间了。

## 4。拦截器的配置

要将新创建的`Interceptor`添加到 Spring 配置中，我们需要覆盖实现`WebMvcConfigurer:`的`WebConfig`类中的`addInterceptors()`方法

```java
@Override
public void addInterceptors(InterceptorRegistry registry) {
    registry.addInterceptor(new SessionTimerInterceptor());
}
```

我们可以通过编辑 XML Spring 配置文件来实现相同的配置:

```java
<mvc:interceptors>
    <bean id="sessionTimerInterceptor" class="com.baeldung.web.interceptor.SessionTimerInterceptor"/>
</mvc:interceptors>
```

此外，我们需要添加监听器，以便自动创建`ApplicationContext`:

```java
public class ListenerConfig implements WebApplicationInitializer {
    @Override
    public void onStartup(ServletContext sc) throws ServletException {
        sc.addListener(new RequestContextListener());
    }
}
```

## 5。结论

本教程展示了如何使用 Spring MVC 的`HandlerInterceptor`拦截 web 请求，以便手动进行会话管理/超时。

像往常一样，所有的例子和配置都可以在 [GitHub](https://web.archive.org/web/20220828134825/https://github.com/eugenp/tutorials/tree/master/spring-security-modules/spring-security-web-mvc-custom) 上找到。

### 5.1。系列文章

该系列的所有文章:

*   [Spring MVC 处理程序拦截器简介](/web/20220828134825/https://www.baeldung.com/spring-mvc-handlerinterceptor)
*   [用处理器拦截器改变弹簧模型参数](/web/20220828134825/https://www.baeldung.com/spring-model-parameters-with-handler-interceptor)
*   +当前文章