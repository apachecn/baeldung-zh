# 在 Zuul 过滤器中修改响应主体

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/zuul-filter-modifying-response-body>

## 1。概述

在本教程中，我们将看看网飞祖尔后过滤器。

网飞·祖尔是一家边缘服务提供商，位于 API 客户端和众多微服务之间。

后置过滤器在最终响应发送到 API 客户端之前运行。这使我们有机会对原始响应体进行操作，并做一些我们想要的事情，如日志记录和其他数据转换。

## 2。依赖性

我们将在 Spring Cloud 环境中与 Zuul 合作。因此，让我们将以下内容添加到我们的`pom.xml:`的依赖性管理部分

```java
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-dependencies</artifactId>
            <version>2020.0.0</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>

<dependencies>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-netflix-zuul</artifactId>
        <version>2.2.2.RELEASE</version>
    </dependency>
</dependencies>
```

最新版本的 [Spring Cloud 依赖](https://web.archive.org/web/20220703155235/https://search.maven.org/search?q=g:org.springframework.cloud%20AND%20a:spring-cloud-dependencies)和`[spring-cloud-starter-netflix-zuul](https://web.archive.org/web/20220703155235/https://search.maven.org/search?q=g:org.springframework.cloud%20a:spring-cloud-starter-netflix-zuul)`可以在 Maven Central 上找到。

## 3。创建后置过滤器

**后置过滤器是扩展抽象类`ZuulFilter`的常规类，过滤器类型为`post`** :

```java
public class ResponseLogFilter extends ZuulFilter {

    @Override
    public String filterType() {
        return POST_TYPE;
    }

    @Override
    public int filterOrder() {
        return 0;
    }

    @Override
    public boolean shouldFilter() {
        return true;
    }

    @Override
    public Object run() throws ZuulException {
        return null;
    }
}
```

请注意，我们在`filterType()`方法中返回了`POST_TYPE`。这就是这种过滤器与其他类型过滤器的真正区别。

另一个需要注意的重要方法是`shouldFilter()`方法。我们在这里返回`true`,因为我们希望过滤器在过滤器链中运行。

在生产就绪的应用程序中，我们可以将这种配置外部化以获得更好的灵活性。

让我们仔细看看`run()`，每当我们的过滤器运行时，它都会被调用。

## 4。修改响应正文

如前所述，Zuul 位于微服务和它们的客户之间。因此，它可以访问响应体，并在传递之前有选择地修改它。

例如，我们可以读取响应正文并记录其内容:

```java
@Override
public Object run() throws ZuulException {

    RequestContext context = RequestContext.getCurrentContext();
    try (final InputStream responseDataStream = context.getResponseDataStream()) {

        if(responseDataStream == null) {
            logger.info("BODY: {}", "");
            return null;
        }

        String responseData = CharStreams.toString(new InputStreamReader(responseDataStream, "UTF-8"));
        logger.info("BODY: {}", responseData);

        context.setResponseBody(responseData);
    }
    catch (Exception e) {
        throw new ZuulException(e, INTERNAL_SERVER_ERROR.value(), e.getMessage());
    }

    return null;
}
```

上面的代码片段展示了我们之前创建的`ResponseLogFilter`中的`run()`方法的完整实现。首先，我们获得了一个`RequestContext`的实例。从这个上下文中，我们能够在尝试使用资源构造时获得响应数据`InputStream` 。

注意，响应输入流可以是`null,` ,这就是我们检查它的原因。这可能是由于服务超时或微服务上的其他意外异常。在我们的例子中，当这种情况发生时，我们只记录一个空的响应体。

接下来，我们将输入流读入到一个`String`中，然后我们可以记录它。

**非常重要的是，我们使用`context.setResponseBody(responseData).`** 将响应体添加回上下文进行处理。如果我们省略这一步，我们将得到一个`IOException`，如下所示:`java.io.IOException: Attempted read on a closed stream`。

## 5。结论

总之，Zuul 中的 post filters 为开发人员提供了一个机会，在将服务响应发送给客户端之前对其进行处理。

但是，我们必须小心不要意外暴露敏感信息，否则会导致违规。

此外，我们应该警惕在 post 过滤器中执行长时间运行的任务，因为这会大大增加响应时间。

和往常一样，源代码可以在 GitHub 的[上找到。](https://web.archive.org/web/20220703155235/https://github.com/eugenp/tutorials/tree/master/spring-cloud-modules/spring-cloud-zuul)