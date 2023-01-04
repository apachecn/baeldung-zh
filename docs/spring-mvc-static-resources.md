# 用 Spring 服务静态资源

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-mvc-static-resources>

## 1。概述

本教程将探索如何使用 XML 和 Java 配置通过 Spring 服务静态资源。

## 延伸阅读:

## [使用 Spring MVC 的可缓存静态资产](/web/20221130175218/https://www.baeldung.com/cachable-static-assets-with-spring-mvc)

This article shows how to cache your static assets such as Javascript and CSS files when serving them with Spring MVC.[Read more](/web/20221130175218/https://www.baeldung.com/cachable-static-assets-with-spring-mvc) →

## [web jars 简介](/web/20221130175218/https://www.baeldung.com/maven-webjars)

A quick and practical guide to using WebJars with Spring.[Read more](/web/20221130175218/https://www.baeldung.com/maven-webjars) →

## [用 Maven 缩小 JS 和 CSS 资产](/web/20221130175218/https://www.baeldung.com/maven-minification-of-js-and-css-assets)

A quick guide to using Maven to minify Javascript and CSS files in a Java web project.[Read more](/web/20221130175218/https://www.baeldung.com/maven-minification-of-js-and-css-assets) →

## 2.使用 Spring Boot

Spring Boot 附带了一个预配置的`[ResourceHttpRequestHandler](https://web.archive.org/web/20221130175218/https://github.com/spring-projects/spring-framework/blob/master/spring-webmvc/src/main/java/org/springframework/web/servlet/resource/ResourceHttpRequestHandler.java) `实现，以便于服务静态资源。

**默认情况下，这个处理程序提供来自类路径**上的`/static, /public, /resources, `和`/META-INF/resources `目录的静态内容。由于默认情况下`src/main/resources` 通常在类路径上，所以我们可以将这些目录中的任何一个放在那里。

例如，如果我们将一个`about.html `文件放在类路径的`/static `目录中，那么我们可以通过`http://localhost:8080/about.html`访问该文件。类似地，我们可以通过将该文件添加到其他提到的目录中来获得相同的结果。

### 2.1.自定义路径模式

**默认情况下，Spring Boot 服务于请求的根部分下的所有静态内容，** `**/****` `. `尽管这似乎是一个很好的默认配置，**我们可以通过`[spring.mvc.static-path-pattern](https://web.archive.org/web/20221130175218/https://github.com/spring-projects/spring-boot/blob/master/spring-boot-project/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/web/servlet/WebMvcProperties.java#L105) `** **配置属性来更改它。**

例如，如果我们想通过`http://localhost:8080/content/about.html, `访问同一个文件，我们可以在`application.properties:`中这样说

```java
spring.mvc.static-path-pattern=/content/**
```

在 WebFlux 环境中，我们应该使用`[spring.webflux.static-path-pattern](https://web.archive.org/web/20221130175218/https://github.com/spring-projects/spring-boot/blob/c71ed407e43e561333719573d63464ae9db388c1/spring-boot-project/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/web/reactive/WebFluxProperties.java#L38)`属性。

### 2.2.自定义目录

与路径模式类似，**也可以通过`[spring.web.resources.static-locations](https://web.archive.org/web/20221130175218/https://github.com/spring-projects/spring-boot/blob/main/spring-boot-project/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/web/WebProperties.java#L86)` 配置属性更改默认资源位置。**该属性可以接受多个逗号分隔的资源位置:

```java
spring.web.resources.static-locations=classpath:/files/,classpath:/static-files
```

在这里，我们提供来自类路径中的`/files` 和`/static-files` 目录的静态内容。此外， **Spring Boot 可以从类路径之外提供静态文件**:

```java
spring.web.resources.static-locations=file:/opt/files 
```

这里我们使用[文件资源签名](https://web.archive.org/web/20221130175218/https://docs.spring.io/spring/docs/5.2.4.RELEASE/spring-framework-reference/core.html#resources-resourceloader)，`file:/`，从本地磁盘提供文件。

## 3。XML 配置

如果我们需要走基于 XML 的配置的老路，我们可以很好地利用`mvc:resources`元素，用特定的公共 URL 模式指向资源的位置。

例如，下面一行将通过搜索我们应用程序中根文件夹下的"/ `resources/`"目录，为所有使用公共 URL 模式(如"`/resources/**”,`)的资源请求提供服务:

```java
<mvc:resources mapping="/resources/**" location="/resources/" />
```

现在我们可以访问一个 CSS 文件，如下面的 HTML 页面所示:

```java
<%@ taglib uri="http://java.sun.com/jsp/jstl/core" prefix="c" %>
<html>
<head>
    <link href="<c:url value="/resources/myCss.css" />" rel="stylesheet">
    <title>Home</title>
</head>
<body>
    <h1>Hello world!</h1>
</body>
</html>
```

## 4。`ResourceHttpRequestHandler`

春天 3.1。引入了`ResourceHand` `lerRegistry`来配置`ResourceHttpRequestHandler`的,用于从类路径、WAR 或文件系统提供静态资源。我们可以在我们的 web 上下文配置类中以编程方式配置`ResourceHandlerRegistry` 。

### 4.1。为存储在 WAR 中的资源提供服务

为了说明这一点，我们将使用与之前相同的 URL 指向`myCss.css`，但是现在实际的文件将位于 WAR 的`webapp/resources`文件夹中，这是我们在部署 Spring 3.1+应用程序时应该放置静态资源的位置:

```java
@Configuration
@EnableWebMvc
public class MvcConfig implements WebMvcConfigurer {
    @Override
    public void addResourceHandlers(ResourceHandlerRegistry registry) {
        registry
          .addResourceHandler("/resources/**")
          .addResourceLocations("/resources/");	
    }
}
```

让我们来分析一下这个例子。首先，我们通过定义一个资源处理器来配置面向外部的 URI 路径。然后，我们将面向外部的 URI 路径内部映射到资源实际所在的物理路径。

当然，我们可以使用这个简单而灵活的 API 定义多个资源处理程序。

现在在一个`html`页面中的下面一行将为我们获取在`webapp/resources`目录中的`myCss.css`资源:

```java
<link href="<c:url value="/resources/myCss.css" />" rel="stylesheet">
```

### 4.2。为存储在文件系统中的资源提供服务

比方说，每当有请求进入与`/files/**` 模式匹配的公共 URL 时，我们希望提供存储在`/opt/files/` 目录中的资源。我们只需配置 URL 模式，并将其映射到磁盘上的特定位置:

```java
@Override
public void addResourceHandlers(ResourceHandlerRegistry registry) {
    registry
      .addResourceHandler("/files/**")
      .addResourceLocations("file:/opt/files/");
 }
```

对于 Windows 用户，在本例中传递给`addResourceLocations`的参数是“`file:///C:/opt/files/`”。

一旦我们配置了资源位置，我们就可以使用我们的`home.html`中映射的 URL 模式来**加载存储在文件系统中的图像:**

```java
<%@ taglib uri="http://java.sun.com/jsp/jstl/core" prefix="c" %>
<html>
<head>
    <link href="<c:url value="/resources/myCss.css" />" rel="stylesheet">
    <title>Home</title>
</head>
<body>
    <h1>Hello world!</h1>
    <img alt="image"  src="<c:url value="files/myImage.png" />">
</body>
</html>
```

### 4.3。为资源配置多个位置

如果我们想在一个以上的位置寻找一个资源呢？

我们可以用`addResourceLocations`方法包含多个位置。将按顺序搜索位置列表，直到找到资源:

```java
@Override
public void addResourceHandlers(ResourceHandlerRegistry registry) {
    registry
      .addResourceHandler("/resources/**")
      .addResourceLocations("/resources/","classpath:/other-resources/");
}
```

下面的 curl 请求将显示存储在类路径中应用程序的`webappp/resources`或`other-resources`文件夹中的`Hello.html`页面:

```java
curl -i http://localhost:8080/handling-spring-static-resources/resources/Hello.html
```

## 5。新`ResourceResolvers`

春天 4.1。为新的 **`ResourcesResolvers,`** 提供了不同类型的资源解析器，可用于在加载静态资源时优化浏览器性能。这些解析器可以链接并缓存在浏览器中，以优化请求处理。

### 5.1。`PathResourceResolver`

这是最简单的解析器，其目的是在给定公共 URL 模式的情况下找到资源。事实上，如果我们不在`ResourceChainRegistration`上添加一个`ResourceResolver`，这就是默认的解析器。

让我们看一个例子:

```java
@Override
public void addResourceHandlers(ResourceHandlerRegistry registry) {
    registry
      .addResourceHandler("/resources/**")
      .addResourceLocations("/resources/","/other-resources/")
      .setCachePeriod(3600)
      .resourceChain(true)
      .addResolver(new PathResourceResolver());
}
```

注意事项:

*   我们将资源链中的`PathResourceResolver`注册为唯一的`ResourceResolver`。我们可以参考 4.3 节。来看看如何连锁多一个`ResourceResolver`。
*   所提供的资源将在浏览器中缓存 3600 秒。
*   最后用方法`resourceChain(true)`配置链。

现在，对于与`PathResourceResolver`一起在`webapp/resources`或`webapp/other-resources`文件夹中定位`foo.js` 脚本的 HTML 代码:

```java
<script type="text/javascript" src="<c:url value="/resources/foo.js" />">
```

### 5.2。`EncodedResourceResolver`

这个解析器试图根据`Accept-Encoding`请求头值找到一个编码的资源。

例如，我们可能需要通过使用`gzip`内容编码提供静态资源的压缩版本来优化带宽。

要配置一个`EncodedResourceResolver,`，我们需要在 `ResourceChain,`中配置它，就像我们配置`PathResourceResolver`一样:

```java
registry
  .addResourceHandler("/other-files/**")
  .addResourceLocations("file:/Users/Me/")
  .setCachePeriod(3600)
  .resourceChain(true)
  .addResolver(new EncodedResourceResolver());
```

默认情况下，`EncodedResourceResolver`配置为支持`br`和`gzip`编码。

因此，下面的`curl`请求将获得位于文件系统的`Users/Me/`目录中的`Home.html`文件的压缩版本:

```java
curl -H  "Accept-Encoding:gzip" 
  http://localhost:8080/handling-spring-static-resources/other-files/Hello.html
```

注意**我们是如何将头的`Accept-Encoding`值设置为`gzip.`** 的，这很重要，因为只有当 gzip 内容对响应有效时，这个特定的解析器才会生效。

最后，请注意，与之前一样，压缩版本将在浏览器中缓存的时间段内保持可用，在本例中是 3600 秒。

### 5.3。链接`ResourceResolvers`

为了优化资源查找， *ResourceResolvers* 可以将资源的处理委托给其他解析器。唯一不能委托给链的解析器是我们应该添加在链末端的`PathResourceResolver,`。

事实上，如果`resourceChain`没有被设置为 `true`，那么默认情况下只有一个 `PathResourceResolver`将被用来提供资源。在这里，如果*gzipsourceresolver*不成功，我们将链接`PathResourceResolver`来解析资源:

```java
@Override
public void addResourceHandlers(ResourceHandlerRegistry registry) {
    registry
      .addResourceHandler("/js/**")
      .addResourceLocations("/js/")
      .setCachePeriod(3600)
      .resourceChain(true)
      .addResolver(new GzipResourceResolver())
      .addResolver(new PathResourceResolver());
}
```

现在我们已经将`/js/**`模式添加到了`ResourceHandler`中，让我们包含`foo.js`资源，它位于我们的`home.html`页面的`webapp/js/`目录中:

```java
<%@ taglib uri="http://java.sun.com/jsp/jstl/core" prefix="c" %>
<html>
<head>
    <link href="<c:url value="/resources/bootstrap.css" />" rel="stylesheet" />
    <script type="text/javascript"  src="<c:url value="/js/foo.js" />"></script>
    <title>Home</title>
</head>
<body>
    <h1>This is Home!</h1>
    <img alt="bunny hop image"  src="<c:url value="files/myImage.png" />" />
    <input type = "button" value="Click to Test Js File" onclick = "testing();" />
</body>
</html>
```

值得一提的是，从 Spring Framework 5.1 开始，`[GzipResourceResolver](https://web.archive.org/web/20221130175218/https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/web/servlet/resource/GzipResourceResolver.html) `已经被弃用，取而代之的是`EncodedResourceResolver`。所以以后要避免使用。

## 6。附加安全配置

如果使用 Spring Security，允许访问静态资源是很重要的。我们需要添加相应的权限来访问资源 URL:

```java
<intercept-url pattern="/files/**" access="permitAll" />
<intercept-url pattern="/other-files/**/" access="permitAll" />
<intercept-url pattern="/resources/**" access="permitAll" />
<intercept-url pattern="/js/**" access="permitAll" />
```

## 7。结论

在本文中，我们展示了 Spring 应用程序服务静态资源的各种方式。

基于 XML 的资源配置**是一个遗留选项**，如果我们还不能走 Java 配置路线，我们可以使用它。

**春天 3.1。通过它的`ResourceHandlerRegistry`对象，**产生了一个基本的程序选择。

最后，**随弹簧 4.1** 发货的新开箱`ResourceResolvers` 和`ResourceChainRegistration`对象。提供资源加载优化功能，如缓存和资源处理程序链接，以提高服务静态资源的效率。

和往常一样，完整的例子可以在 Github 的[上找到。此外，Spring Boot 相关的源代码也可以在这个项目](https://web.archive.org/web/20221130175218/https://github.com/eugenp/tutorials/tree/master/spring-static-resources)中[获得。](https://web.archive.org/web/20221130175218/https://github.com/eugenp/tutorials/tree/master/spring-boot-modules/spring-boot-mvc-2)