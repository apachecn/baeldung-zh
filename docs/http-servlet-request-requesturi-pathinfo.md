# HttpServletRequest 中 getRequestURI 和 getPathInfo 的区别

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/http-servlet-request-requesturi-pathinfo>

## 1。概述

在这个快速教程中，我们将讨论`HttpServletRequest` 类中`getRequestURI()`和`getPathInfo()`的区别。

## 2。`getRequestURI()`与`getPathInfo()`和的区别

函数 **`getRequestURI()`** **返回完整的被请求的 URI。**这包括部署文件夹和 servlet 映射字符串。它还将返回所有额外的路径信息。

函数 **`getPathInfo()`** **只返回传递给 servlet** 的路径。如果没有传递额外的路径信息，这个函数将返回`null`。

换句话说，如果我们在 web 服务器的根目录下部署应用程序，并且**请求映射到“/”的 servlet，那么`getRequestURI()`和`getPathInfo()`将返回相同的 strin** g。否则，我们将得到不同的值。

## 3。请求示例

为了更好地理解`HttpServletRequest`方法，假设我们有一个 [servlet](/web/20220626195427/https://www.baeldung.com/intro-to-servlets) ，可以通过这个 URL 访问:

```java
http://localhost:8080/deploy-folder/servlet-mapping
```

这个请求将命中部署在“deploy-folder”中的 web 应用程序中的“servlet-mapping”servlet。因此，如果我们为这个请求调用`getRequestURI()`和`getPathInfo()` ，它们将返回不同的字符串。

让我们创建一个简单的`doGet()` servlet 方法:

```java
public void doGet(HttpServletRequest request, HttpServletResponse response) throws IOException {
    PrintWriter writer = response.getWriter();
    if ("getPathInfo".equals(request.getParameter("function")) {
        writer.println(request.getPathInfo());
    } else if ("getRequestURI".equals(request.getParameter("function")) {
        writer.println(request.getRequestURI());
    }
    writer.flush();
}
```

首先，让我们看一下 servlet 对由 [curl](/web/20220626195427/https://www.baeldung.com/curl-rest) 命令获取的`getRequestURI`请求的输出:

```java
curl http://localhost:8080/deploy-folder/servlet-mapping/request-path?function=getRequestURI
```

```java
/deploy-folder/servlet-mapping/request-path 
```

同样，让我们来看看`getPathInfo`的 servlet 的输出:

```java
curl http://localhost:8080/deploy-folder/servlet-mapping/request-path?function=getPathInfo
```

```java
/request-path
```

## 4。结论

在本文中，我们解释了`HttpServletRequest` 中`getRequestURI()`和`getPathInfo()`的**区别。我们还用一个示例 servlet 和请求演示了它。**

和往常一样，为测试所有这些功能而实现的 servlet 在 Github 上[可用。](https://web.archive.org/web/20220626195427/https://github.com/eugenp/tutorials/tree/master/javax-servlets)