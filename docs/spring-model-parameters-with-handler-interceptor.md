# 使用处理程序拦截器更改 Spring 模型参数

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-model-parameters-with-handler-interceptor>

## 1。简介

在本教程中，我们将关注 Spring MVC `HandlerInterceptor.` 更具体地说，我们将在处理请求之前和之后改变 Spring MVC 的模型参数。

如果你想阅读`HandlerInterceptor's` 基础知识，看看这篇[文章](/web/20220126111518/https://www.baeldung.com/spring-mvc-handlerinterceptor)。

## 2。Maven 依赖关系

为了使用`Interceptors`，您需要在您的`pom.xml`文件的`dependencies`部分中包含以下部分:

```java
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-web</artifactId>
    <version>5.3.13</version>
</dependency> 
```

最新版本可以在这里找到[。](https://web.archive.org/web/20220126111518/https://search.maven.org/classic/#search%7Cga%7C1%7Ca%3A%22spring-web%22)

这个依赖项只涉及 Spring Web，所以不要忘记为完整的 Web 应用程序添加 s `pring-core`和`spring-context`，以及您选择的日志库。

## 3。定制实施

`HandlerInterceptor`的一个用例是向模型添加公共/用户特定的参数，这些参数将在每个生成的视图上可用。

在我们的示例中，我们将使用定制的拦截器实现将登录用户的用户名添加到模型参数中。在更复杂的系统中，我们可能会添加更具体的信息，如:用户头像路径、用户位置等。

让我们从定义新的`Interceptor`类开始:

```java
public class UserInterceptor extends HandlerInterceptorAdapter {

    private static Logger log = LoggerFactory.getLogger(UserInterceptor.class);

    ...
}
```

我们扩展了`HandlerInterceptorAdapter`，因为我们只想实现`preHandle()` 和`postHandle()`方法。

正如我们之前提到的，我们希望将登录用户的名字添加到模型中。首先，我们需要检查一个用户是否登录。我们可以通过检查`SecurityContextHolder`获得此信息:

```java
public static boolean isUserLogged() {
    try {
        return !SecurityContextHolder.getContext().getAuthentication()
          .getName().equals("anonymousUser");
    } catch (Exception e) {
        return false;
    }
}
```

当建立了一个`HttpSession` ，但是没有人登录时，Spring 安全上下文中的用户名等于`anonymousUser`。接下来，我们继续执行`preHandle():`

### 3.1。`preHandle()`法

在处理请求之前，我们不能访问模型参数。为了添加用户名，我们需要使用`HttpSession` 来设置参数:

```java
@Override
public boolean preHandle(HttpServletRequest request,
  HttpServletResponse response, Object object) throws Exception {
    if (isUserLogged()) {
        addToModelUserDetails(request.getSession());
    }
    return true;
}
```

如果我们在处理请求之前使用这些信息，这是至关重要的。如我们所见，我们正在检查用户是否登录，然后通过获取其会话向我们的请求添加参数:

```java
private void addToModelUserDetails(HttpSession session) {
    log.info("=============== addToModelUserDetails =========================");

    String loggedUsername 
      = SecurityContextHolder.getContext().getAuthentication().getName();
    session.setAttribute("username", loggedUsername);

    log.info("user(" + loggedUsername + ") session : " + session);
    log.info("=============== addToModelUserDetails =========================");
}
```

我们用`SecurityContextHolder` 来获得`loggedUsername`。您可以覆盖 Spring Security `UserDetails`实现来获得电子邮件而不是标准用户名。

### 3.2。法 p `ostHandle()`

处理完请求后，我们的模型参数就可用了，因此我们可以访问它们来更改值或添加新值。为了做到这一点，我们使用被覆盖的`postHandle()` 方法:

```java
@Override
public void postHandle(
  HttpServletRequest req, 
  HttpServletResponse res,
  Object o, 
  ModelAndView model) throws Exception {

    if (model != null && !isRedirectView(model)) {
        if (isUserLogged()) {
        addToModelUserDetails(model);
    }
    }
}
```

我们来看看实现细节。

首先，最好检查一下型号是否不是`null.`这样可以防止我们遇到`NullPointerException`。

此外，我们可以检查一个`View`是否不是重定向`View.`的实例

在请求被处理然后被重定向之后，不需要添加/改变参数，因为新的控制器将立即再次执行处理。为了检查视图是否被重定向，我们引入了以下方法:

```java
public static boolean isRedirectView(ModelAndView mv) {
    String viewName = mv.getViewName();
    if (viewName.startsWith("redirect:/")) {
        return true;
    }
    View view = mv.getView();
    return (view != null && view instanceof SmartView
      && ((SmartView) view).isRedirectView());
}
```

最后，我们再次检查用户是否登录，如果是，我们将向 Spring 模型添加参数:

```java
private void addToModelUserDetails(ModelAndView model) {
    log.info("=============== addToModelUserDetails =========================");

    String loggedUsername = SecurityContextHolder.getContext()
      .getAuthentication().getName();
    model.addObject("loggedUsername", loggedUsername);

    log.trace("session : " + model.getModel());
    log.info("=============== addToModelUserDetails =========================");
}
```

请注意，日志记录非常重要，因为这个逻辑在应用程序的“幕后”工作。很容易忘记我们在没有正确记录的情况下更改了每个`View`的一些模型参数。

## 4。配置

要将新创建的`Interceptor`添加到 Spring 配置中，我们需要覆盖实现`WebMvcConfigurer:`的`WebConfig`类中的`addInterceptors()`方法

```java
@Override
public void addInterceptors(InterceptorRegistry registry) {
    registry.addInterceptor(new UserInterceptor());
}
```

我们可以通过编辑 XML Spring 配置文件来实现相同的配置:

```java
<mvc:interceptors>
    <bean id="userInterceptor" class="com.baeldung.web.interceptor.UserInterceptor"/>
</mvc:interceptors>
```

从这一刻起，我们可以在所有生成的视图上访问所有与用户相关的参数。

请注意，如果配置了多个弹簧`Interceptors`，则按照配置的顺序执行`preHandle()`方法，而按照相反的顺序调用`postHandle()`和`afterCompletion()`方法。

## 5。结论

本教程展示了如何使用 Spring MVC 的 HandlerInterceptor 来拦截 web 请求，以便提供用户信息。

在这个特定的例子中，我们关注于在我们的 web 应用程序中将登录用户的详细信息添加到模型参数中。您可以通过添加更详细的信息来扩展这个`HandlerInterceptor` 实现。

所有示例和配置都可以在 [GitHub](https://web.archive.org/web/20220126111518/https://github.com/eugenp/tutorials/tree/master/spring-security-modules/spring-security-web-mvc-custom) 上找到。

### 5.1。系列文章

该系列的所有文章:

*   [Spring MVC 处理程序拦截器简介](/web/20220126111518/https://www.baeldung.com/spring-mvc-handlerinterceptor)
*   用处理程序拦截器更改 Spring 模型参数(这一个)