# 新泽西过滤器和拦截器

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/jersey-filters-interceptors>

## 1。简介

在本文中，我们将解释过滤器和拦截器如何在 Jersey 框架中工作，以及它们之间的主要区别。

这里我们将使用 Jersey 2，并且我们将使用 Tomcat 9 服务器测试我们的应用程序。

## 2。应用程序设置

让我们首先在服务器上创建一个简单的资源:

```java
@Path("/greetings")
public class Greetings {

    @GET
    public String getHelloGreeting() {
        return "hello";
    }
}
```

同样，让我们为我们的应用程序创建相应的服务器配置:

```java
@ApplicationPath("/*")
public class ServerConfig extends ResourceConfig {

    public ServerConfig() {
        packages("com.baeldung.jersey.server");
    }
}
```

如果你想更深入地了解如何用 Jersey 创建一个 API，你可以看看这篇文章。

你也可以看看我们的客户端文章，学习如何用 Jersey 创建一个 Java 客户端。

## 3。过滤器

现在，让我们从过滤器开始。

简单地说，**过滤器让我们修改请求和响应的属性**——例如，HTTP 头。过滤器可以应用于服务器端和客户端。

请记住，无论是否找到资源，总是会执行**过滤器。**

### 3.1。实现请求服务器过滤器

让我们从服务器端的过滤器开始，创建一个请求过滤器。

**我们将通过实现`ContainerRequestFilter`接口并在我们的服务器**中将它注册为`Provider`来实现

```java
@Provider
public class RestrictedOperationsRequestFilter implements ContainerRequestFilter {

    @Override
    public void filter(ContainerRequestContext ctx) throws IOException {
        if (ctx.getLanguage() != null && "EN".equals(ctx.getLanguage()
          .getLanguage())) {

            ctx.abortWith(Response.status(Response.Status.FORBIDDEN)
              .entity("Cannot access")
              .build());
        }
    }
}
```

这个简单的过滤器只是通过调用`abortWith()`方法来拒绝请求中带有语言`“EN”`的请求。

如示例所示，我们只需要实现一个接收请求上下文的方法，我们可以根据需要修改它。

让我们记住这个过滤器是在资源匹配后执行的。

如果我们想在资源匹配之前执行一个过滤器，**我们可以通过用`@PreMatching`注释**注释我们的过滤器来使用预匹配过滤器:

```java
@Provider
@PreMatching
public class PrematchingRequestFilter implements ContainerRequestFilter {

    @Override
    public void filter(ContainerRequestContext ctx) throws IOException {
        if (ctx.getMethod().equals("DELETE")) {
            LOG.info("\"Deleting request");
        }
    }
}
```

如果我们现在尝试访问我们的资源，我们可以检查我们的预匹配过滤器是否首先被执行:

```java
2018-02-25 16:07:27,800 [http-nio-8080-exec-3] INFO  c.b.j.s.f.PrematchingRequestFilter - prematching filter
2018-02-25 16:07:27,816 [http-nio-8080-exec-3] INFO  c.b.j.s.f.RestrictedOperationsRequestFilter - Restricted operations filter
```

### 3.2。实施响应服务器过滤器

现在，我们将在服务器端实现一个响应过滤器，它只会向响应添加一个新的头。

为此，**我们的过滤器必须实现`ContainerResponseFilter`接口**，并实现它唯一的方法:

```java
@Provider
public class ResponseServerFilter implements ContainerResponseFilter {

    @Override
    public void filter(ContainerRequestContext requestContext, 
      ContainerResponseContext responseContext) throws IOException {
        responseContext.getHeaders().add("X-Test", "Filter test");
    }
}
```

请注意，`ContainerRequestContext`参数只是作为只读参数使用——因为我们已经在处理响应了。

### 2.3。实施客户端过滤器

我们现在将在客户端使用过滤器。这些过滤器的工作方式与服务器过滤器相同，我们必须实现的接口与服务器端的接口非常相似。

让我们看看它在过滤器中的运行情况，该过滤器为请求添加了一个属性:

```java
@Provider
public class RequestClientFilter implements ClientRequestFilter {

    @Override
    public void filter(ClientRequestContext requestContext) throws IOException {
        requestContext.setProperty("test", "test client request filter");
    }
}
```

让我们也创建一个 Jersey 客户机来测试这个过滤器:

```java
public class JerseyClient {

    private static String URI_GREETINGS = "http://localhost:8080/jersey/greetings";

    public static String getHelloGreeting() {
        return createClient().target(URI_GREETINGS)
          .request()
          .get(String.class);
    }

    private static Client createClient() {
        ClientConfig config = new ClientConfig();
        config.register(RequestClientFilter.class);

        return ClientBuilder.newClient(config);
    }
}
```

注意，我们必须将过滤器添加到客户机配置中来注册它。

最后，我们还将为客户机中的响应创建一个过滤器。

这与服务器中的工作方式非常相似，但是实现了`ClientResponseFilter`接口:

```java
@Provider
public class ResponseClientFilter implements ClientResponseFilter {

    @Override
    public void filter(ClientRequestContext requestContext, 
      ClientResponseContext responseContext) throws IOException {
        responseContext.getHeaders()
          .add("X-Test-Client", "Test response client filter");
    }

}
```

同样，`ClientRequestContext`是只读的。

## 4。截击机

拦截器更多地与包含在请求和响应中的 HTTP 消息体的编组和解组联系在一起。它们既可以在服务器端使用，也可以在客户端使用。

请记住，它们是在过滤器之后执行的，并且只有当消息体存在时才执行。

拦截器有两种类型:`ReaderInterceptor`和`WriterInterceptor`，对于服务器端和客户端都是一样的。

接下来，我们将在我们的服务器上创建另一个资源——通过 POST 访问该资源，并在主体中接收一个参数，因此拦截器将在访问它时执行:

```java
@POST
@Path("/custom")
public Response getCustomGreeting(String name) {
    return Response.status(Status.OK.getStatusCode())
      .build();
}
```

我们还将向 Jersey 客户端添加一个新方法，以测试这个新资源:

```java
public static Response getCustomGreeting() {
    return createClient().target(URI_GREETINGS + "/custom")
      .request()
      .post(Entity.text("custom"));
}
```

### 4.1。实施一个`ReaderInterceptor`

读取器拦截器允许我们操作入站流，因此我们可以使用它们来修改服务器端的请求或客户端的响应。

让我们在服务器端创建一个拦截器，在被拦截的请求体中编写一条自定义消息:

```java
@Provider
public class RequestServerReaderInterceptor implements ReaderInterceptor {

    @Override
    public Object aroundReadFrom(ReaderInterceptorContext context) 
      throws IOException, WebApplicationException {
        InputStream is = context.getInputStream();
        String body = new BufferedReader(new InputStreamReader(is)).lines()
          .collect(Collectors.joining("\n"));

        context.setInputStream(new ByteArrayInputStream(
          (body + " message added in server reader interceptor").getBytes()));

        return context.proceed();
    }
}
```

注意**我们必须调用`proceed()` 方法** **来调用链中的下一个拦截器**。一旦执行了所有的拦截器，就会调用适当的消息体读取器。

### 3.2。实施一个`WriterInterceptor`

写拦截器的工作方式与读拦截器非常相似，但是它们操纵出站流——因此我们可以在客户端的请求或服务器端的响应中使用它们。

让我们创建一个 writer 拦截器，在客户端向请求添加一条消息:

```java
@Provider
public class RequestClientWriterInterceptor implements WriterInterceptor {

    @Override
    public void aroundWriteTo(WriterInterceptorContext context) 
      throws IOException, WebApplicationException {
        context.getOutputStream()
          .write(("Message added in the writer interceptor in the client side").getBytes());

        context.proceed();
    }
}
```

同样，我们必须调用方法`proceed()` 来调用下一个拦截器。

当所有的拦截器都被执行时，适当的消息体编写器将被调用。

**不要忘记，您必须在客户端配置**中注册这个拦截器，就像我们之前在客户端过滤器中所做的那样:

```java
private static Client createClient() {
    ClientConfig config = new ClientConfig();
    config.register(RequestClientFilter.class);
    config.register(RequestWriterInterceptor.class);

    return ClientBuilder.newClient(config);
}
```

## 5。执行顺序

让我们用一个图表来总结一下我们到目前为止看到的所有内容，该图表显示了在客户端向服务器发出请求的过程中，何时执行过滤器和拦截器:

[![](img/349945b35adaf04989be79ad3540ca26.png)](/web/20220526040716/https://www.baeldung.com/wp-content/uploads/2018/03/Jersey2.png)

正如我们所见，**过滤器总是首先执行，拦截器就在调用适当的消息体读取器或写入器**之前执行。

如果我们看一下我们创建的过滤器和拦截器，它们将按以下顺序执行:

1.  `RequestClientFilter`
2.  `RequestClientWriterInterceptor`
3.  `PrematchingRequestFilter`
4.  `RestrictedOperationsRequestFilter`
5.  RequestServerReaderInterceptor
6.  `ResponseServerFilter`
7.  `ResponseClientFilter`

此外，当我们有几个过滤器或拦截器时，我们可以通过用`@Priority`注释来指定确切的执行顺序。

优先级是用一个`Integer`指定的，它按照请求的升序和响应的降序对过滤器和拦截器进行排序。

让我们给我们的`RestrictedOperationsRequestFilter`添加一个优先级:

```java
@Provider
@Priority(Priorities.AUTHORIZATION)
public class RestrictedOperationsRequestFilter implements ContainerRequestFilter {
    // ...
}
```

请注意，我们使用了预定义的优先级进行授权。

## 6。名称绑定

到目前为止，我们看到的过滤器和拦截器被称为全局的，因为它们是为每个请求和响应而执行的。

但是，**它们也可以被定义为只为特定的资源方法**执行，这就是所谓的名称绑定。

### 6.1。静态绑定

进行名称绑定的一种方法是静态地创建一个将在所需资源中使用的特定注释。这个注释必须包含`@NameBinding`元注释。

让我们在应用程序中创建一个:

```java
@NameBinding
@Retention(RetentionPolicy.RUNTIME)
public @interface HelloBinding {
}
```

之后，我们可以用这个`@HelloBinding`注释来注释一些资源:

```java
@GET
@HelloBinding
public String getHelloGreeting() {
    return "hello";
}
```

最后，我们也要用这个注释来注释我们的一个过滤器，所以这个过滤器将只对访问`getHelloGreeting()`方法的请求和响应执行:

```java
@Provider
@Priority(Priorities.AUTHORIZATION)
@HelloBinding
public class RestrictedOperationsRequestFilter implements ContainerRequestFilter {
    // ...
}
```

请记住，我们的`RestrictedOperationsRequestFilter`不会再被其余的资源触发。

### 6.2。动态绑定

另一种方法是使用动态绑定，它在启动时加载到配置中。

让我们首先为这个部分向我们的服务器添加另一个资源:

```java
@GET
@Path("/hi")
public String getHiGreeting() {
    return "hi";
}
```

现在，让我们通过实现`DynamicFeature`接口为这个资源创建一个绑定:

```java
@Provider
public class HelloDynamicBinding implements DynamicFeature {

    @Override
    public void configure(ResourceInfo resourceInfo, FeatureContext context) {
        if (Greetings.class.equals(resourceInfo.getResourceClass()) 
          && resourceInfo.getResourceMethod().getName().contains("HiGreeting")) {
            context.register(ResponseServerFilter.class);
        }
    }
}
```

在本例中，我们将`getHiGreeting()`方法与我们之前创建的`ResponseServerFilter`相关联。

重要的是要记住，我们必须从这个过滤器中删除`@Provider`注释，因为我们现在通过`DynamicFeature`来配置它。

如果我们不这样做，过滤器将被执行两次:一次作为全局过滤器，另一次作为绑定到`getHiGreeting()`方法的过滤器。

## 7。结论

在本教程中，我们重点了解了过滤器和拦截器在 Jersey 2 中是如何工作的，以及如何在 web 应用程序中使用它们。

与往常一样，GitHub 上的[提供了示例的完整源代码。](https://web.archive.org/web/20220526040716/https://github.com/eugenp/tutorials/tree/master/jersey)