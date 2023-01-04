# JAX 的 CORS

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/cors-in-jax-rs>

## 1。概述

在这篇简短的文章中，我们将了解如何在基于`JAX-RS`的系统中启用 [CORS](/web/20220628082640/https://www.baeldung.com/cs/cors-preflight-requests) ( `Cross-Origin Resource Sharing`)。我们将在`JAX-RS`之上建立一个应用程序来启用`CORS`机制。

## 2。如何启用 CORS 机制

我们可以通过两种方式在 JAX 启用 CORS-RS。第一种也是最基本的方法是创建一个过滤器，在运行时在每个请求中注入必要的响应头。另一种方法是在每个 URL 端点中手动添加一个适当的头。

理想情况下，应该使用第一种解决方案；然而，当这不是一个选项时，更手动的选项在技术上也是可以的。

### 2.1。使用过滤器

`JAX-RS`有 [`ContainerResponseFilter`](https://web.archive.org/web/20220628082640/https://docs.oracle.com/javaee/7/api/javax/ws/rs/container/ContainerResponseFilter.html) 接口——由容器响应过滤器实现。通常，此过滤器实例会全局应用于任何 HTTP 响应。

我们将实现这个接口来创建一个定制的过滤器，它将向每个传出的请求注入`Access-Control-Allow-*`头，并启用`CORS`机制:

```java
@Provider
public class CorsFilter implements ContainerResponseFilter {

    @Override
    public void filter(ContainerRequestContext requestContext, 
      ContainerResponseContext responseContext) throws IOException {
          responseContext.getHeaders().add(
            "Access-Control-Allow-Origin", "*");
          responseContext.getHeaders().add(
            "Access-Control-Allow-Credentials", "true");
          responseContext.getHeaders().add(
           "Access-Control-Allow-Headers",
           "origin, content-type, accept, authorization");
          responseContext.getHeaders().add(
            "Access-Control-Allow-Methods", 
            "GET, POST, PUT, DELETE, OPTIONS, HEAD");
    }
}
```

这里有几点:

*   实现`ContainerResponseFilter`的过滤器必须用`@Provider`显式注释，以便被 JAX-RS 运行时发现
*   我们在'`Access-Control-Allow-*`'头中插入' * '，这意味着可以通过任何域访问该服务器实例的任何 URL 端点；如果我们想要明确地限制跨域访问，我们必须在这个头中提到这个域

### 2.2。使用头修改成每个端点

如前所述，我们也可以在端点级别显式注入'`Access-Control-Allow-*`'头:

```java
@GET
@Path("/")
@Produces({MediaType.TEXT_PLAIN})
public Response index() {
    return Response
      .status(200)
      .header("Access-Control-Allow-Origin", "*")
      .header("Access-Control-Allow-Credentials", "true")
      .header("Access-Control-Allow-Headers",
        "origin, content-type, accept, authorization")
      .header("Access-Control-Allow-Methods", 
        "GET, POST, PUT, DELETE, OPTIONS, HEAD")
      .entity("")
      .build();
}
```

这里要注意的一点是，如果我们试图在一个大型应用程序中启用`CORS`，我们不应该尝试这种方法，因为在这种情况下，我们必须手动将头注入每个 URL 端点，这将引入额外的开销。

然而，这种技术可以用在应用程序中，我们只需要在一些 URL 端点中启用`CORS`。

## 3。测试

一旦应用程序启动，我们就可以使用 curl 命令测试头文件。示例头输出应该如下所示:

```java
HTTP/1.1 200 OK
Date : Tue, 13 May 2014 12:30:00 GMT
Connection : keep-alive
Access-Control-Allow-Origin : *
Access-Control-Allow-Credentials : true
Access-Control-Allow-Headers : origin, content-type, accept, authorization
Access-Control-Allow-Methods : GET, POST, PUT, DELETE, OPTIONS, HEAD
Transfer-Encoding : chunked
```

此外，我们可以创建一个简单的 AJAX 函数并检查跨域功能:

```java
function call(url, type, data) {
    var request = $.ajax({
      url: url,
      method: "GET",
      data: (data) ? JSON.stringify(data) : "",
      dataType: type
    });

    request.done(function(resp) {
      console.log(resp);
    });

    request.fail(function(jqXHR, textStatus) {
      console.log("Request failed: " + textStatus);
    });
};
```

当然，为了实际执行检查，我们必须在我们使用的 API 之外的另一个源上运行。

您可以通过在一个单独的端口上运行客户端应用程序，在本地非常容易地做到这一点–**,因为端口决定了原点。**

## 4。结论

在本文中，我们展示了在基于 JAX-RS 的应用程序中实现`CORS`机制。

像往常一样，完整的源代码可以在 GitHub 上获得[。](https://web.archive.org/web/20220628082640/https://github.com/eugenp/tutorials/tree/master/resteasy)