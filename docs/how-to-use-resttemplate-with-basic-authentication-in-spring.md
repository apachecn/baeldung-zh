# 使用 RestTemplate 的基本身份验证

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/how-to-use-resttemplate-with-basic-authentication-in-spring>

**目录**

*   [**1。**概述](#overview)
*   [**2。**立春`RestTemplate`](#resttemplate)
*   [**3。**手工管理授权 HTTP 头](#manual_auth)
*   [**4。**授权 HTTP 头的自动管理](#automatic_auth)
*   [**5。**美芬依存](#maven)
*   [**6。**结论](#conclusion)

## 1。概述

在本教程中，我们将学习如何使用 Spring 的`RestTemplate`来**消费一个受基本认证**保护的 RESTful 服务。

一旦我们为模板设置了基本的身份验证，每个请求将被发送**，包含执行身份验证过程所必需的完整凭证**。根据基本认证方案的规范，凭证将被编码，并使用`Authorization` HTTP 头。一个例子是这样的:

```java
Authorization: Basic QWxhZGRpbjpvcGVuIHNlc2FtZQ==
```

## 延伸阅读:

## [Spring RestTemplate 错误处理](/web/20221125232917/https://www.baeldung.com/spring-rest-template-error-handling)

Learn how to handle errors with Spring's RestTemplate[Read more](/web/20221125232917/https://www.baeldung.com/spring-rest-template-error-handling) →

## [使用 Spring RestTemplate 拦截器](/web/20221125232917/https://www.baeldung.com/spring-rest-template-interceptor)

Learn about using interceptors in your Spring application with the RestTemplate.[Read more](/web/20221125232917/https://www.baeldung.com/spring-rest-template-interceptor) →

## [探索 Spring Boot TestRestTemplate](/web/20221125232917/https://www.baeldung.com/spring-boot-testresttemplate)

Learn how to use the new TestRestTemplate in Spring Boot to test a simple API.[Read more](/web/20221125232917/https://www.baeldung.com/spring-boot-testresttemplate) →

## 2。`RestTemplate`设置

我们可以简单地通过为它声明一个 bean 来将`RestTemplate`引导到 Spring 上下文中；然而，设置带有**基本认证**的`RestTemplate`将需要手动干预，因此为了更大的灵活性，我们将使用 Spring `FactoryBean`而不是直接声明 bean。此`FactoryBean`将在初始化时创建和配置模板:

```java
@Component
public class RestTemplateFactory
  implements FactoryBean<RestTemplate>, InitializingBean {

    private RestTemplate restTemplate;

    public RestTemplate getObject() {
        return restTemplate;
    }
    public Class<RestTemplate> getObjectType() {
        return RestTemplate.class;
    }
    public boolean isSingleton() {
        return true;
    }

    public void afterPropertiesSet() {
        HttpHost host = new HttpHost("localhost", 8082, "http");
        restTemplate = new RestTemplate(
          new HttpComponentsClientHttpRequestFactoryBasicAuth(host));
    }
}
```

`host`和`port`值应该取决于环境，允许客户灵活地为集成测试定义一组值，为生产使用定义另一组值。这些值可以由属性文件的[一级 Spring 支持来管理。](/web/20221125232917/https://www.baeldung.com/project-configuration-with-spring "Using Properties to configure a Spring Project")

## 3。授权 HTTP 头的手动管理

对于我们来说，为基本认证创建`Authorization`头相当简单，所以我们可以用几行代码手动完成:

```java
HttpHeaders createHeaders(String username, String password){
   return new HttpHeaders() {{
         String auth = username + ":" + password;
         byte[] encodedAuth = Base64.encodeBase64( 
            auth.getBytes(Charset.forName("US-ASCII")) );
         String authHeader = "Basic " + new String( encodedAuth );
         set( "Authorization", authHeader );
      }};
}
```

此外，发送请求也很简单:

```java
restTemplate.exchange
 (uri, HttpMethod.POST, new HttpEntity<T>(createHeaders(username, password)), clazz);
```

## 4。授权 HTTP 头的自动管理

Spring 3.0 和 3.1，以及现在的 4.x，对 Apache HTTP 库有很好的支持:

*   在 Spring 3.0 中，`CommonsClientHttpRequestFactory`与现在的**整合成了寿终正寝的** `HttpClient 3.x.`
*   Spring 3.1 通过`HttpComponentsClientHttpRequestFactory`引入了对当前 **HttpClient 4.x** 的支持(在 JIRA [SPR-6180](https://web.archive.org/web/20221125232917/https://jira.springsource.org/browse/SPR-6180 "Upgrade Apache HttpClient to version 4.0") 中增加了支持)。
*   Spring 4.0 通过`HttpComponentsAsyncClientHttpRequestFactory.`引入了异步支持

让我们开始用 HttpClient 4 和 Spring 4 进行设置。

`RestTemplate`将需要一个支持基本认证的 HTTP 请求工厂。然而，直接使用现有的`HttpComponentsClientHttpRequestFactory`将被证明是困难的，因为`RestTemplate`的架构是在没有很好地支持`HttpContext,`的[的情况下设计的。因此，我们需要子类化`HttpComponentsClientHttpRequestFactory`并覆盖`createHttpContext`方法:](https://web.archive.org/web/20221125232917/https://jira.springsource.org/browse/SPR-8367 "JIRA issue on RestTemplate support for Basic Authentication")

```java
public class HttpComponentsClientHttpRequestFactoryBasicAuth 
  extends HttpComponentsClientHttpRequestFactory {

    HttpHost host;

    public HttpComponentsClientHttpRequestFactoryBasicAuth(HttpHost host) {
        super();
        this.host = host;
    }

    protected HttpContext createHttpContext(HttpMethod httpMethod, URI uri) {
        return createHttpContext();
    }

    private HttpContext createHttpContext() {
        AuthCache authCache = new BasicAuthCache();

        BasicScheme basicAuth = new BasicScheme();
        authCache.put(host, basicAuth);

        BasicHttpContext localcontext = new BasicHttpContext();
        localcontext.setAttribute(HttpClientContext.AUTH_CACHE, authCache);
        return localcontext;
    }
}
```

我们在这里构建了基本的认证支持，创建了`HttpContext`。正如我们所看到的，使用 HttpClient 4.x 进行抢先式基本身份验证对我们来说是一种负担。身份验证信息是缓存的，对我们来说，设置这种身份验证缓存非常手动且不直观。

现在一切就绪，`RestTemplate`将能够只通过添加一个`BasicAuthorizationInterceptor:`来支持基本的认证方案

```java
restTemplate.getInterceptors().add(
  new BasicAuthorizationInterceptor("username", "password"));
```

然后请求:

```java
restTemplate.exchange(
  "http://localhost:8082/spring-security-rest-basic-auth/api/foos/1", 
  HttpMethod.GET, null, Foo.class);
```

要深入讨论如何保护 REST 服务本身，[请查看本文](/web/20221125232917/https://www.baeldung.com/basic-and-digest-authentication-for-a-rest-api-with-spring-security "Authentication for a REST API")。

## 5。Maven 依赖关系

对于`RestTemplate`本身和 HttpClient 库，我们需要以下 Maven 依赖项:

```java
<dependency>
   <groupId>org.springframework</groupId>
   <artifactId>spring-webmvc</artifactId>
   <version>5.0.6.RELEASE</version>
</dependency>

<dependency>
   <groupId>org.apache.httpcomponents</groupId>
   <artifactId>httpclient</artifactId>
   <version>4.5.3</version>
</dependency>
```

可选地，如果我们手动构造 HTTP `Authorization`头，那么我们将需要一个额外的库来支持编码:

```java
<dependency>
   <groupId>commons-codec</groupId>
   <artifactId>commons-codec</artifactId>
   <version>1.10</version>
</dependency>
```

我们可以在 [Maven 资源库](https://web.archive.org/web/20221125232917/https://search.maven.org/classic/#search%7Cga%7C1%7C(g%3A%22org.springframework%22%20AND%20a%3A%22spring-webmvc%22)%20OR%20(g%3A%22org.apache.httpcomponents%22%20AND%20a%3A%22httpclient%22)%20OR%20(g%3A%22commons-codec%22%20AND%20a%3A%22commons-codec%22))中找到最新版本。

## 6。结论

尽管 3.x 分支已经寿终正寝，Spring 对该版本的支持也完全被否决，但是在`RestTemplate`和 security 上可以找到的许多信息仍然没有考虑到当前的 HttpClient 4.x 版本。在本文中，我们试图通过一步一步的详细讨论来改变这种情况，讨论如何使用`RestTemplate`设置基本认证，并使用它来消费安全的 REST API。

为了超越本文中的代码示例，实现消费端和实际的 RESTful 服务，看看 Github 上的项目[。](https://web.archive.org/web/20221125232917/https://github.com/eugenp/tutorials/tree/master/spring-security-modules/spring-security-web-rest-basic-auth)

这是一个基于 Maven 的项目，因此应该很容易导入和运行。