# 自定义 Spring 安全配置器

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-security-custom-configurer>

## 1。概述

Spring Security Java 配置支持为我们提供了强大的流畅 APIs 为应用程序定义安全映射和规则。

在这篇简短的文章中，我们将看到如何向前迈出这一步，并实际定义一个定制的配置器；这是一种将定制逻辑引入标准安全配置的先进而灵活的方法。

对于我们这里的快速示例，我们将添加根据给定的错误状态代码列表为经过身份验证的用户记录错误的功能。

## 2。`SecurityConfigurer` 一种风俗

为了开始定义我们的配置器，首先**我们需要扩展`AbstractHttpConfigurer`类**:

```
public class ClientErrorLoggingConfigurer 
  extends AbstractHttpConfigurer<ClientErrorLoggingConfigurer, HttpSecurity> {

    private List<HttpStatus> errorCodes;

    // standard constructors

    @Override
    public void init(HttpSecurity http) throws Exception {
        // initialization code
    }

    @Override
    public void configure(HttpSecurity http) throws Exception {
       http.addFilterAfter(
         new ClientErrorLoggingFilter(errorCodes), 
         FilterSecurityInterceptor.class);
    }
}
```

这里，**我们需要覆盖的主要方法是`configure()`方法**——它包含这个配置器将应用的安全配置。

在我们的例子中，我们在最后一个 Spring 安全过滤器之后注册了一个新的过滤器。此外，因为我们打算记录响应状态错误代码，所以我们添加了一个`errorCodes List`属性，我们可以用它来控制我们将记录的错误代码。

我们还可以选择在`init()`方法中添加额外的配置，它在`configure()`方法之前执行。

接下来，让我们定义在自定义实现中注册的 Spring 安全过滤器类:

```
public class ClientErrorLoggingFilter extends GenericFilterBean {

    private static final Logger logger = LogManager.getLogger(
      ClientErrorLoggingFilter.class);
    private List<HttpStatus> errorCodes;

    // standard constructor

    @Override
    public void doFilter(
      ServletRequest request, 
      ServletResponse response, 
      FilterChain chain) 
      throws IOException, ServletException {
        //...

        chain.doFilter(request, response);
    }
}
```

这是一个标准的 Spring filter 类，它扩展了`GenericFilterBean`并覆盖了`doFilter()`方法。它有两个属性，分别代表我们将用来显示消息的记录器和`errorCodes.`的`List`

让我们仔细看看`doFilter()`方法:

```
Authentication auth = SecurityContextHolder.getContext().getAuthentication();
if (auth == null) {
    chain.doFilter(request, response);
    return;
}
int status = ((HttpServletResponse) response).getStatus();
if (status < 400 || status >= 500) {
    chain.doFilter(request, response);
    return;
}
if (errorCodes == null) {
    logger.debug("User " + auth.getName() + " encountered error " + status);
} else {
    if (errorCodes.stream().anyMatch(s -> s.value() == status)) {
        logger.debug("User " + auth.getName() + " encountered error " + status);
    }
}
```

如果状态代码是客户端错误状态代码，意思是在 400 和 500 之间，那么我们将检查`errorCodes`列表。

如果这是空的，那么我们将显示任何客户端错误状态代码。否则，我们将首先检查错误代码是否是给定的状态代码`List`的一部分。

## 3。使用自定义配置器

现在我们有了自己的自定义 API，**我们可以通过定义 bean，然后使用`HttpSecurity:`的`apply()`方法**将它添加到 Spring 安全配置中

```
@Configuration
public class SecurityConfig extends WebSecurityConfigurerAdapter {

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.authorizeRequests()
          //...
          .and()
          .apply(clientErrorLogging());
    }

    @Bean
    public ClientErrorLoggingConfigurer clientErrorLogging() {
        return new ClientErrorLoggingConfigurer() ;
    }
}
```

我们还可以用我们想要记录的特定错误代码列表来定义 bean:

```
@Bean
public ClientErrorLoggingConfigurer clientErrorLogging() {
    return new ClientErrorLoggingConfigurer(Arrays.asList(HttpStatus.NOT_FOUND)) ;
}
```

仅此而已！现在，我们的安全配置将包括自定义过滤器并显示日志消息。

如果我们希望默认添加定制配置器，我们可以使用`META-INF/spring.factories`文件:

```
org.springframework.security.config.annotation.web.configurers.AbstractHttpConfigurer = com.baeldung.dsl.ClientErrorLoggingConfigurer
```

要手动禁用它，我们可以使用`disable()`方法:

```
//...
.apply(clientErrorLogging()).disable();
```

## 4。结论

在这个快速教程中，我们关注了 Spring 安全配置支持的一个高级特性——**我们已经看到了如何定义我们自己的自定义`SecurityConfigurer`** 。

和往常一样，这个例子的完整源代码可以在 GitHub 上找到[。](https://web.archive.org/web/20220628105252/https://github.com/eugenp/tutorials/tree/master/spring-security-modules/spring-5-security)