# 为所有 Spring Boot 控制器添加前缀

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-boot-controllers-add-prefix>

## 1.介绍

在 Spring Boot 应用程序中，每个控制器都可以有自己的 URL 映射。这使得单个应用程序很容易在多个位置提供 web 端点。例如，我们可以将我们的 API 端点分成逻辑组，比如内部和外部。

然而，有时我们可能希望所有的端点都在同一个前缀下。在本教程中，我们将看看为所有 Spring Boot 控制器使用一个公共前缀的不同方法。

## 2.Servlet 上下文

Spring 应用程序中负责处理 web 请求的主要组件是`[DispatcherServlet](/web/20220815034912/https://www.baeldung.com/spring-dispatcherservlet)`。通过定制这个组件，我们可以很好地控制请求的路由方式。

让我们看一下定制`DispatcherServlet`的两种不同方式，这将使我们所有的应用程序端点在一个公共的 URL 前缀上可用。

### 2.1.春豆

第一种方法是引入新的 Spring bean:

```java
@Configuration
public class DispatcherServletCustomConfiguration {

    @Bean
    public DispatcherServlet dispatcherServlet() {
        return new DispatcherServlet();
    }

    @Bean
    public ServletRegistrationBean dispatcherServletRegistration() {
        ServletRegistrationBean registration = new ServletRegistrationBean(dispatcherServlet(), "/api/");
        registration.setName(DispatcherServletAutoConfiguration.DEFAULT_DISPATCHER_SERVLET_REGISTRATION_BEAN_NAME);
        return registration;
    }
}
```

这里，我们创建了一个包装了`DispatcherServlet` bean 的`ServletRegistrationBean`。注意，我们提供了一个显式的基本 URL`/api/`。**这意味着我们所有的端点都必须通过基本 URL 前缀**来访问。

### 2.2.应用程序属性

我们也可以通过使用应用程序属性来获得相同的结果。在 Spring Boot 2 . 0 . 0 之后的版本中，我们会将以下内容添加到我们的`application.properties`文件中:

```java
server.servlet.contextPath=/api
```

在该版本之前，属性名称略有不同:

```java
server.contextPath=/api
```

这种方法的一个好处是它只使用普通的弹簧属性。**这意味着我们可以使用标准机制，如概要文件或外部属性绑定，轻松地更改或覆盖我们的公共前缀**。

### 2.3.利弊

这两种方法的主要优点也是主要缺点:它们影响应用程序中的每个端点。

对于某些应用程序来说，这可能非常好。然而，一些应用程序可能需要使用标准端点映射来与第三方服务进行交互，例如 OAuth 交换。在这些情况下，像这样的全球解决方案可能并不适合。

## 3.释文

我们可以在 Spring 应用程序中为所有控制器添加前缀的另一种方法是使用注释。下面，我们来看看两种不同的方法。

### 3.1.斯佩尔

第一种方式包括使用标准的`@RequestMapping annotation`和 [Spring 表达式语言](/web/20220815034912/https://www.baeldung.com/spring-expression-language) (SpEL)。使用这种方法，我们只需向每个想要添加前缀的控制器添加一个属性:

```java
@Controller
@RequestMapping(path = "${apiPrefix}/users")
public class UserController {

} 
```

然后，我们只需在我们的`application.properties`中指定属性值:

```java
apiPrefix=/api
```

### 3.2.自定义注释

另一种方法是创建我们自己的注释:

```java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Component
@RequestMapping("/api/")
public @interface ApiPrefixController {
    @AliasFor(annotation = Component.class)
    String value() default "";
}
```

然后，我们只需要将注释应用到我们想要添加前缀的每个控制器:

```java
@Controller
@ApiPrefixController
public class SomeController {
    @RequestMapping("/users")
    @ReponseBody
    public String getAll(){
        // ...
    }
}
```

### 3.3.利弊

这两种方法解决了前一种方法的主要问题:**它们都提供了对哪些控制器获得前缀**的细粒度控制。我们可以仅将注释应用于特定的控制器，而不是影响应用程序中的所有端点。

## 4.服务器端转发

我们要看的最后一种方法是使用服务器端转发。**与重定向不同，转发不涉及对客户端的响应**。这意味着我们的应用程序可以在端点之间传递请求，而不会影响客户端。

首先，让我们编写一个具有两个端点的简单控制器:

```java
@Controller
class EndpointController {
    @GetMapping("/endpoint1")
    @ResponseBody
    public String endpoint1() {
        return "Hello from endpoint 1";
    }

    @GetMapping("/endpoint2")
    @ResponseBody
    public String endpoint2() {
        return "Hello from endpoint 2";
    }
}
```

接下来，我们根据我们想要的前缀创建一个新的控制器:

```java
@Controller
@RequestMapping("/api/endpoint")
public class ApiPrefixController {

    @GetMapping
    public ModelAndView route(ModelMap model) {
        if(new Random().nextBoolean()) {
            return new ModelAndView("forward:/endpoint1", model);
        } 
        else {
            return new ModelAndView("forward:/endpoint2", model);
        }
    }
}
```

该控制器有一个充当路由器的端点。在这种情况下，它实质上是通过抛硬币将原始请求转发到我们的另外两个端点之一。

我们可以通过发送几个连续的请求来验证它是否工作:

```java
> curl http://localhost:8080/api/endpoint
Hello from endpoint 2
> curl http://localhost:8080/api/endpoint
Hello from endpoint 1
> curl http://localhost:8080/api/endpoint
Hello from endpoint 1
> curl http://localhost:8080/api/endpoint
Hello from endpoint 2
> curl http://localhost:8080/api/endpoint
Hello from endpoint 2
```

这种方法的主要好处是它非常强大。我们可以应用任何我们想要的逻辑来确定如何转发请求:URL 路径、HTTP 方法、HTTP 头等等。

## 5.结论

在本文中，我们学习了在 Spring 应用程序中为每个控制器应用一个公共前缀的几种方法。与大多数决策一样，每种方法都有利弊，在实施之前应该仔细考虑。

一如既往，本教程中的代码示例可以在 GitHub 上找到[。](https://web.archive.org/web/20220815034912/https://github.com/eugenp/tutorials/tree/master/spring-boot-modules/spring-boot-mvc-3/src/main/java/com/baeldung/controllerprefix)