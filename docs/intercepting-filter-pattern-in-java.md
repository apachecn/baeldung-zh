# Java 拦截过滤器模式简介

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/intercepting-filter-pattern-in-java>

## 1。概述

在本教程中，我们将介绍`Intercepting Filter Pattern`表示层核心 J2EE 模式。

这是我们`Pattern Series`的第二个教程，也是`Front Controller Pattern`指南的后续，可以在`[here](/web/20221128123645/https://www.baeldung.com/java-front-controller-pattern).`找到

`Intercepting Filters` 是在处理程序处理传入请求之前或之后触发操作的过滤器。

拦截过滤器代表了 web 应用程序中的集中式组件，对所有请求都是通用的，并且是可扩展的，不会影响现有的处理程序。

## 2。用例

让我们扩展前面指南[中的例子](/web/20221128123645/https://www.baeldung.com/java-front-controller-pattern)并实现**一个认证机制、请求日志和访问者计数器**。此外，我们希望能够以各种不同的**编码交付我们的页面**。****

 **所有这些都是拦截过滤器的用例，因为它们对所有请求都是通用的，并且应该独立于处理程序。

## 3。过滤策略

让我们介绍不同的过滤策略和示例性用例。要使用 Jetty Servlet 容器运行代码，只需执行:

```java
$> mvn install jetty:run
```

### 3.1。自定义过滤策略

定制过滤器策略用于每个需要有序处理请求的用例，在**的意义上，一个过滤器基于执行链**中前一个过滤器的结果。

这些链将通过实现 [`FilterChain`](https://web.archive.org/web/20221128123645/https://tomcat.apache.org/tomcat-7.0-doc/servletapi/javax/servlet/FilterChain.html) 接口并向其注册各种`Filter`类来创建。

当使用具有不同关注点的多个过滤器链时，您可以在过滤器管理器中将它们连接在一起:

[![intercepting filter-custom strategy](img/9abf3327bca0fe972430463b0558d5cb.png)](/web/20221128123645/https://www.baeldung.com/wp-content/uploads/2016/11/intercepting_filter-custom_strategy.png)

在我们的示例中，访问者计数器通过对登录用户的唯一用户名进行计数来工作，这意味着它基于身份验证过滤器的结果，因此，两个过滤器必须链接在一起。

让我们实现这个过滤器链。

首先，我们将创建一个身份验证过滤器，它检查会话是否存在，如果不存在，则发出一个登录过程:

```java
public class AuthenticationFilter implements Filter {
    ...
    @Override
    public void doFilter(
      ServletRequest request,
      ServletResponse response, 
      FilterChain chain) {
        HttpServletRequest httpServletRequest = (HttpServletRequest) request;
        HttpServletResponse httpServletResponse = (HttpServletResponse) response;

        HttpSession session = httpServletRequest.getSession(false);
        if (session == null || session.getAttribute("username") == null) {
            FrontCommand command = new LoginCommand();
            command.init(httpServletRequest, httpServletResponse);
            command.process();
        } else {
            chain.doFilter(request, response);
        }
    }

    ...
}
```

现在让我们创建访问者计数器。该过滤器维护一个唯一用户名的`HashSet`,并向请求添加一个“计数器”属性:

```java
public class VisitorCounterFilter implements Filter {
    private static Set<String> users = new HashSet<>();

    ...
    @Override
    public void doFilter(ServletRequest request, ServletResponse response,
      FilterChain chain) {
        HttpSession session = ((HttpServletRequest) request).getSession(false);
        Optional.ofNullable(session.getAttribute("username"))
          .map(Object::toString)
          .ifPresent(users::add);
        request.setAttribute("counter", users.size());
        chain.doFilter(request, response);
    }

    ...
}
```

接下来，我们将实现一个`FilterChain`，它迭代注册的过滤器并执行`doFilter` 方法:

```java
public class FilterChainImpl implements FilterChain {
    private Iterator<Filter> filters;

    public FilterChainImpl(Filter... filters) {
        this.filters = Arrays.asList(filters).iterator();
    }

    @Override
    public void doFilter(ServletRequest request, ServletResponse response) {
        if (filters.hasNext()) {
            Filter filter = filters.next();
            filter.doFilter(request, response, this);
        }
    }
}
```

为了将我们的组件连接在一起，让我们创建一个简单的静态管理器，它负责实例化过滤器链，注册过滤器，并初始化它:

```java
public class FilterManager {
    public static void process(HttpServletRequest request,
      HttpServletResponse response, OnIntercept callback) {
        FilterChain filterChain = new FilterChainImpl(
          new AuthenticationFilter(callback), new VisitorCounterFilter());
        filterChain.doFilter(request, response);
    }
}
```

作为最后一步，我们必须从我们的`FrontCommand`内部调用我们的`FilterManager`作为请求处理序列的公共部分:

```java
public abstract class FrontCommand {
    ...

    public void process() {
        FilterManager.process(request, response);
    }

    ...
}
```

### 3.2。基本过滤策略

在这一节中，我们将介绍`Base Filter Strategy,` ，所有实现的过滤器都使用一个公共超类。

这个策略与上一节中的定制策略或我们将在下一节中介绍的`Standard Filter Strategy`配合得很好。

抽象基类可用于应用属于过滤器链的自定义行为。我们将在示例中使用它来减少与过滤器配置和调试日志记录相关的样板代码:

```java
public abstract class BaseFilter implements Filter {
    private Logger log = LoggerFactory.getLogger(BaseFilter.class);

    protected FilterConfig filterConfig;

    @Override
    public void init(FilterConfig filterConfig) throws ServletException {
        log.info("Initialize filter: {}", getClass().getSimpleName());
        this.filterConfig = filterConfig;
    }

    @Override
    public void destroy() {
        log.info("Destroy filter: {}", getClass().getSimpleName());
    }
}
```

让我们扩展这个基类来创建一个请求日志过滤器，它将被集成到下一节:

```java
public class LoggingFilter extends BaseFilter {
    private static final Logger log = LoggerFactory.getLogger(LoggingFilter.class);

    @Override
    public void doFilter(
      ServletRequest request, 
      ServletResponse response,
      FilterChain chain) {
        chain.doFilter(request, response);
        HttpServletRequest httpServletRequest = (HttpServletRequest) request;

        String username = Optional
          .ofNullable(httpServletRequest.getAttribute("username"))
          .map(Object::toString)
          .orElse("guest");

        log.info(
          "Request from '{}@{}': {}?{}", 
          username, 
          request.getRemoteAddr(),
          httpServletRequest.getRequestURI(), 
          request.getParameterMap());
    }
}
```

### 3.3。标准过滤策略

一种更灵活的应用过滤器的方式是实现`Standard Filter Strategy`。这可以通过在部署描述符中声明过滤器来实现，或者从 Servlet 规范 3.0 开始，通过注释来实现。

标准过滤器策略允许将新过滤器插入默认链，而无需明确定义过滤器管理器:

[![intercepting filter-standard strategy](img/15fa07d166ad027bfe97397fe7e61568.png)](/web/20221128123645/https://www.baeldung.com/wp-content/uploads/2016/11/intercepting_filter-standard_strategy.png)

请注意，过滤器的应用顺序不能通过注释来指定。如果您需要有序执行，您必须坚持使用部署描述符或实现定制的过滤器策略。

让我们实现一个注释驱动的编码过滤器，它也使用基本过滤器策略:

```java
@WebFilter(servletNames = {"intercepting-filter"}, 
  initParams = {@WebInitParam(name = "encoding", value = "UTF-8")})
public class EncodingFilter extends BaseFilter {
    private String encoding;

    @Override
    public void init(FilterConfig filterConfig) throws ServletException {
        super.init(filterConfig);
        this.encoding = filterConfig.getInitParameter("encoding");
    }

    @Override
    public void doFilter(ServletRequest request,
      ServletResponse response, FilterChain chain) {
        String encoding = Optional
          .ofNullable(request.getParameter("encoding"))
          .orElse(this.encoding);
        response.setCharacterEncoding(encoding); 

        chain.doFilter(request, response);
    }
}
```

在具有部署描述符的 Servlet 场景中，我们的`web.xml`将包含这些额外的声明:

```java
<filter>
    <filter-name>encoding-filter</filter-name>
    <filter-class>
      com.baeldung.patterns.intercepting.filter.filters.EncodingFilter
    </filter-class>
</filter>
<filter-mapping>
    <filter-name>encoding-filter</filter-name>
    <servlet-name>intercepting-filter</servlet-name>
</filter-mapping>
```

让我们拿起日志过滤器并对其进行注释，以便 Servlet 使用:

```java
@WebFilter(servletNames = "intercepting-filter")
public class LoggingFilter extends BaseFilter {
    ...
}
```

### 3.4。模板过滤策略

`Template Filter Strategy`与基本过滤器策略非常相似，除了它使用基类中声明的模板方法，这些方法必须在实现中被覆盖:

[![intercepting filter-template strategy](img/3f8b5e5d625c014f1a282ef606c8b1f4.png)](/web/20221128123645/https://www.baeldung.com/wp-content/uploads/2016/11/intercepting_filter-template_strategy.png)

让我们用两个抽象过滤器方法创建一个基本过滤器类，在进一步处理之前和之后调用这两个方法。

由于这种策略不太常见，我们在示例中没有使用它，所以具体的实现和用例取决于您的想象:

```java
public abstract class TemplateFilter extends BaseFilter {
    protected abstract void preFilter(HttpServletRequest request,
      HttpServletResponse response);

    protected abstract void postFilter(HttpServletRequest request,
      HttpServletResponse response);

    @Override
    public void doFilter(ServletRequest request,
      ServletResponse response, FilterChain chain) {
        HttpServletRequest httpServletRequest = (HttpServletRequest) request;
        HttpServletResponse httpServletResponse = (HttpServletResponse) response;

        preFilter(httpServletRequest, httpServletResponse);
        chain.doFilter(request, response);
        postFilter(httpServletRequest, httpServletResponse);
    }
}
```

## 4。结论

截取过滤器模式捕获可以独立于业务逻辑发展的横切关注点。从业务运营的角度来看，过滤器是作为一系列前置或后置操作来执行的。

正如我们到目前为止所看到的，可以使用不同的策略来实现`Intercepting Filter Pattern`。在“真实世界”的应用中，这些不同的方法可以结合起来。

像往常一样，你会发现来源 [`on GitHub`](https://web.archive.org/web/20221128123645/https://github.com/eugenp/tutorials/tree/master/patterns-modules/intercepting-filter) 。**