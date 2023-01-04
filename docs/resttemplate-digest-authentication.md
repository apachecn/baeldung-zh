# 带有摘要式身份验证的 RestTemplate

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/resttemplate-digest-authentication>

 ![](img/9a8c1219885d9f39d3a5969197cc5c1f.png)

It’s just plain hard to get true, **real-time visibility into a running auth flow.**

Parts of the process can be completely hidden from us; if the complete authorization process requires a redirect from a remote OAuth production server, then every debugging effort must go through the production server.

It’s practically unfeasible to debug this locally. There’s no way to reproduce the exact state and no way to inspect what is actually happening under the hood. Not ideal.

Knowing these types of challenges, we built Lightrun - a real-time production debugging tool - to allow you to understand complicated flows with code-level information. Add logs, take snapshots (virtual breakpoints), and instrument metrics without a remote debugger, without stopping the running service, and, most importantly - **in real-time and without side effects**.

**Learn more with this 5-minute tutorial** focused on debugging these kinds of scenarios using Lightrun:

[>> Debugging Authentication and Authorization Using Lightrun](/web/20220521224623/https://www.baeldung.com/lightrun-n-security)

## 1。概述

本文将展示如何**配置 Spring `RestTemplate`来消费一个使用摘要认证**保护的服务。

与基本认证类似，一旦在模板中设置了摘要认证，客户端将能够通过必要的安全步骤并获得`Authorization`报头所需的信息:

```java
Authorization: Digest 
    username="user1",
    realm="Custom Realm Name",
    nonce="MTM3NTYwOTA5NjU3OTo5YmIyMjgwNTFlMjdhMTA1MWM3OTMyMWYyNDY2MGFlZA==",
    uri="/spring-security-rest-digest-auth/api/foos/1", 
    ....
```

有了这些数据，服务器可以正确地验证请求并返回 200 OK 响应。

## 2。设置`RestTemplate`

在 Spring 上下文中需要将`RestTemplate`声明为**一个 bean**——这在 XML 或普通 Java 中都很简单，使用`@Bean`注释:

```java
import org.apache.http.HttpHost;
import com.baeldung.client.HttpComponentsClientHttpRequestFactoryDigestAuth;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.client.RestTemplate;

@Configuration
public class ClientConfig {

    @Bean
    public RestTemplate restTemplate() {
        HttpHost host = new HttpHost("localhost", 8080, "http");
        CloseableHttpClient client = HttpClientBuilder.create().
          setDefaultCredentialsProvider(provider()).useSystemProperties().build();
        HttpComponentsClientHttpRequestFactory requestFactory = 
          new HttpComponentsClientHttpRequestFactoryDigestAuth(host, client);

        return new RestTemplate(requestFactory);;
    }

    private CredentialsProvider provider() {
        CredentialsProvider provider = new BasicCredentialsProvider();
        UsernamePasswordCredentials credentials = 
          new UsernamePasswordCredentials("user1", "user1Pass");
        provider.setCredentials(AuthScope.ANY, credentials);
        return provider;
    }
}
```

摘要访问机制的大部分配置是在注入模板的客户机 http 请求工厂的定制实现中完成的。

请注意，我们现在使用可以访问安全 API 的凭证来预配置模板。

## 3。配置摘要式认证

我们将通过扩展和配置来利用 Spring 3.1 中引入的对当前 HttpClient 4.x(即`HttpComponentsClientHttpRequestFactory`)的支持。

我们主要是要配置`HttpContext`并为摘要式认证连接我们的定制逻辑:

```java
import java.net.URI;
import org.apache.http.HttpHost;
import org.apache.http.client.AuthCache;
import org.apache.http.client.protocol.ClientContext;
import org.apache.http.impl.auth.DigestScheme;
import org.apache.http.impl.client.BasicAuthCache;
import org.apache.http.protocol.BasicHttpContext;
import org.apache.http.protocol.HttpContext;
import org.springframework.http.HttpMethod;
import org.springframework.http.client.HttpComponentsClientHttpRequestFactory;

public class HttpComponentsClientHttpRequestFactoryDigestAuth 
  extends HttpComponentsClientHttpRequestFactory {

    HttpHost host;

    public HttpComponentsClientHttpRequestFactoryDigestAuth(HttpHost host, HttpClient httpClient) {
        super(httpClient);
        this.host = host;
    }

    @Override
    protected HttpContext createHttpContext(HttpMethod httpMethod, URI uri) {
        return createHttpContext();
    }

    private HttpContext createHttpContext() {
        // Create AuthCache instance
        AuthCache authCache = new BasicAuthCache();
        // Generate DIGEST scheme object, initialize it and add it to the local auth cache
        DigestScheme digestAuth = new DigestScheme();
        // If we already know the realm name
        digestAuth.overrideParamter("realm", "Custom Realm Name");
        authCache.put(host, digestAuth);

        // Add AuthCache to the execution context
        BasicHttpContext localcontext = new BasicHttpContext();
        localcontext.setAttribute(ClientContext.AUTH_CACHE, authCache);
        return localcontext;
    }
}
```

现在，`RestTemplate`可以简单地注射并用于测试:

```java
@Test
public void whenSecuredRestApiIsConsumed_then200OK() {
    String uri = "http://localhost:8080/spring-security-rest-digest-auth/api/foos/1";
    ResponseEntity<Foo> entity = restTemplate.exchange(uri, HttpMethod.GET, null, Foo.class);
    System.out.println(entity.getStatusCode());
}
```

为了说明完整的配置过程，这个测试还设置了用户凭证–`user1`和`user1Pass`。当然，这部分应该只做一次**，并且在测试本身之外**。

## 4。Maven 依赖关系

`RestTemplate`和 HttpClient 库所需的 Maven 依赖关系是:

```java
<dependency>
   <groupId>org.springframework</groupId>
   <artifactId>spring-webmvc</artifactId>
   <version>5.2.8.RELEASE</version>
</dependency>

<dependency>
   <groupId>org.apache.httpcomponents</groupId>
   <artifactId>httpclient</artifactId>
   <version>4.3.5</version>
</dependency>
```

## 5。结论

本教程展示了如何设置和**配置 Rest 模板，以便它可以使用使用摘要认证**保护的应用程序。REST API 本身需要用摘要安全机制进行[配置。](/web/20220521224623/https://www.baeldung.com/spring-security-digest-authentication "Spring Security Digest Authentication")

这个实现可以在 GitHub 项目示例中找到——这是一个基于 Maven 的项目，所以它应该很容易导入和运行。