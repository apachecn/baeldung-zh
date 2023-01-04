# Servlet 3 异步支持 Spring MVC 和 Spring Security

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-mvc-async-security>

## 1。简介

在这个快速教程中，我们将重点关注 Servlet 3 对异步请求的支持**，以及 Spring MVC 和 Spring Security 如何处理这些**。

web 应用程序中异步的最基本动机是处理长时间运行的请求。在大多数用例中，我们需要确保 Spring 安全原则被传播到这些线程。

当然，Spring Security [与 MVC 范围之外的`@Async`](/web/20220815045631/https://www.baeldung.com/spring-security-async-principal-propagation) 集成，并处理 HTTP 请求。

## 2。Maven 依赖关系

为了在 Spring MVC 中使用异步集成，我们需要在我们的`pom.xml`中包含以下依赖项:

```
<dependency>
    <groupId>org.springframework.security</groupId>
    <artifactId>spring-security-web</artifactId>
    <version>5.6.0</version>
</dependency>
<dependency>
    <groupId>org.springframework.security</groupId>
    <artifactId>spring-security-config</artifactId>
    <version>5.6.0</version>
</dependency> 
```

Spring 安全依赖的最新版本可以在[这里](https://web.archive.org/web/20220815045631/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22org.springframework.security%22)找到。

## 3。春天 MVC`@Async`和

根据官方[文件，春安与](https://web.archive.org/web/20220815045631/https://spring.io/blog/2012/12/17/spring-security-3-2-m1-highlights-servlet-3-api-support/#servlet3-async)`[WebAsyncManager](https://web.archive.org/web/20220815045631/http://static.springsource.org/spring/docs/current/javadoc-api/org/springframework/web/context/request/async/WebAsyncManager.html)`整合。

第一步是确保我们的`springSecurityFilterChain`被设置为处理异步请求。我们可以在 Java 配置中这样做，在我们的 *Servlet* 配置类中添加下面一行:

```
dispatcher.setAsyncSupported(true);
```

或者在 XML 配置中:

```
<filter>
    <filter-name>springSecurityFilterChain</filter-name>
    <filter-class>org.springframework.web.filter.DelegatingFilterProxy</filter-class>
    <async-supported>true</async-supported>
</filter>
<filter-mapping>
    <filter-name>springSecurityFilterChain</filter-name>
    <url-pattern>/*</url-pattern>
    <dispatcher>REQUEST</dispatcher>
    <dispatcher>ASYNC</dispatcher>
</filter-mapping>
```

我们还需要在 servlet 配置中启用`async-supported`参数:

```
<servlet>
    ...
    <async-supported>true</async-supported>
    ...
</servlet>
```

现在我们已经准备好发送异步请求，并传播了`SecurityContext`。

Spring Security 中的内部机制将确保当在另一个`Thread` 中提交响应导致用户注销时，我们的`SecurityContext`不再被清除。

## 4。用例

让我们通过一个简单的例子来看看这一点:

```
@Override
public Callable<Boolean> checkIfPrincipalPropagated() {
    Object before 
      = SecurityContextHolder.getContext().getAuthentication().getPrincipal();
    log.info("Before new thread: " + before);

    return new Callable<Boolean>() {
        public Boolean call() throws Exception {
            Object after 
              = SecurityContextHolder.getContext().getAuthentication().getPrincipal();
            log.info("New thread: " + after);
            return before == after;
        }
    };
}
```

**我们要检查弹簧`SecurityContext`是否传播到新线程。**

上面介绍的方法将自动执行它的`Callable`,包括`SecurityContext`,如日志所示:

```
web - 2017-01-02 10:42:19,011 [http-nio-8081-exec-3] INFO
  o.baeldung.web.service.AsyncService - Before new thread:
  [[email protected]](/web/20220815045631/https://www.baeldung.com/cdn-cgi/l/email-protection):
  Username: temporary; Password: [PROTECTED]; Enabled: true;
  AccountNonExpired: true; credentialsNonExpired: true;
  AccountNonLocked: true; Granted Authorities: ROLE_ADMIN

web - 2017-01-02 10:42:19,020 [MvcAsync1] INFO
  o.baeldung.web.service.AsyncService - New thread:
  [[email protected]](/web/20220815045631/https://www.baeldung.com/cdn-cgi/l/email-protection):
  Username: temporary; Password: [PROTECTED]; Enabled: true;
  AccountNonExpired: true; credentialsNonExpired: true;
  AccountNonLocked: true; Granted Authorities: ROLE_ADMIN
```

如果没有设置要传播的`SecurityContext`，第二个请求将以`null`值结束。

还有其他重要的使用案例来使用带有传播`SecurityContext`的异步请求:

*   我们希望发出多个外部请求，这些请求可以并行运行，并且可能需要很长时间来执行
*   我们在本地有一些重要的处理要做，我们的外部请求可以并行执行
*   其他的代表“一发了之”的场景，比如发送电子邮件

请注意，如果我们的多个方法调用以前以同步方式链接在一起，那么将这些方法转换为异步方法可能需要同步结果。

## 5。结论

在这个简短的教程中，我们展示了 Spring 对在认证上下文中处理异步请求的支持`.`

从编程模型的角度来看，新功能看似简单。但是确实有一些方面需要更深入的了解。

这个例子也可以在 Github 上作为 Maven 项目[获得。](https://web.archive.org/web/20220815045631/https://github.com/eugenp/tutorials/tree/master/spring-security-modules/spring-security-web-rest)