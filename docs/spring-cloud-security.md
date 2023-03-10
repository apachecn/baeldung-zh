# Spring 云安全简介

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-cloud-security>

## 1。概述

Spring 云安全模块在 Spring Boot 应用程序中提供了与基于令牌的安全相关的特性。

具体来说，它使基于 OAuth2 的 SSO 变得更容易——支持在资源服务器之间中继令牌，以及使用嵌入式 Zuul 代理配置下游身份验证。

在这篇简短的文章中，我们将了解如何使用 Spring Boot 客户端应用程序、授权服务器和作为资源服务器的 REST API 来配置这些特性。

请注意，对于这个示例，我们只有一个客户端应用程序使用 SSO 来演示云安全特性，但是在典型的场景中，我们至少有两个客户端应用程序来证明单点登录的必要性。

## 2。快速启动云安全 App

让我们从在 Spring Boot 应用程序中配置 SSO 开始。

首先，我们需要添加 [`spring-cloud-starter-oauth2`](https://web.archive.org/web/20220926192518/https://search.maven.org/classic/#search%7Cga%7C1%7Ca%3A%22spring-cloud-starter-oauth2%22) 依赖关系:

```java
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-oauth2</artifactId>
    <version>2.2.2.RELEASE</version>
</dependency>
```

这也会带来`spring-cloud-starter-security`依赖性。

我们可以将任何社交网站配置为我们网站的认证服务器，也可以使用我们自己的服务器。在我们的例子中，我们选择了后一个选项，并配置了一个充当授权服务器的应用程序——它是在`http://localhost:7070/authserver.`本地部署的

我们的授权服务器使用 JWT 令牌。

此外，对于能够检索用户凭证的任何客户机，我们需要配置我们的资源服务器，该服务器运行在端口 9000 上，带有一个可以提供这些凭证的端点。

这里，我们已经配置了一个/ `user`端点，它在`http://localhost:9000/user.`可用

关于如何设置授权服务器和资源服务器的更多细节，请点击这里查看我们之前的文章。

我们现在可以在客户端应用程序的配置类中添加注释:

```java
@Configuration
@EnableOAuth2Sso
public class SiteSecurityConfigurer
  extends WebSecurityConfigurerAdapter {

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        // ...    
    }
}
```

任何需要认证的请求都将被重定向到授权服务器。为此，我们还必须定义服务器属性:

```java
security:
  oauth2:
    client:
      accessTokenUri: http://localhost:7070/authserver/oauth/token
      userAuthorizationUri: http://localhost:7070/authserver/oauth/authorize
      clientId: authserver
      clientSecret: passwordforauthserver
    resource:
      userInfoUri: http://localhost:9000/user
```

注意，我们的类路径中需要有`spring-boot-starter-security` 才能发现上面的配置在工作。

## 3。中继接入令牌

**在中继令牌时，OAuth2 客户端将其收到的 OAuth2 令牌转发给传出的资源请求。**

由于我们已经声明了`@EnableOauth2Sso`注释，Spring Boot 在请求范围中添加了一个`OAuth2ClientContext` bean。基于此，我们可以在我们的客户端应用程序中创建我们自己的`OAuth2RestTemplate`:

```java
@Bean
public OAuth2RestOperations restOperations(
  OAuth2ProtectedResourceDetails resource, OAuth2ClientContext context) {
    return new OAuth2RestTemplate(resource, context);
}
```

一旦我们配置了 bean `,` ,上下文将把访问令牌转发给请求的服务，如果令牌过期，还会刷新令牌。

## 4。使用`RestTemplate` 中继 OAuth 令牌

我们之前在客户端应用程序中定义了一个类型为`OAuth2RestTemplate` 的`restOperations` bean。因此，**我们可以使用`OAuth2RestTemplate`的`getForObject()`方法从我们的客户端向受保护的资源服务器**发送一个带有必要令牌的请求。

首先，让我们在资源服务器中定义一个需要身份验证的端点:

```java
@GetMapping("/person")
@PreAuthorize("hasAnyRole('ADMIN', 'USER')")
public @ResponseBody Person personInfo(){        
    return new Person("abir", "Dhaka", "Bangladesh", 29, "Male");       
 } 
```

这是一个简单的 REST 端点，返回一个`Person`对象的 JSON 表示。

现在，**我们可以使用`getForObject()`方法从客户端应用程序发送一个请求，该方法将令牌中继到资源服务器**:

```java
@Autowired
private RestOperations restOperations;

@GetMapping("/personInfo")
public ModelAndView person() { 
    ModelAndView mav = new ModelAndView("personinfo");
    String personResourceUrl = "http://localhost:9000/person";
    mav.addObject("person", 
      restOperations.getForObject(personResourceUrl, String.class));       

    return mav;
}
```

## 5。为令牌中继配置 Zuul】

如果我们想将令牌向下游传递给代理服务，我们可以使用 Spring Cloud Zuul 嵌入式反向代理。

首先，我们需要添加 Maven 依赖项来使用 Zuul:

```java
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-zuul</artifactId>
</dependency>
```

接下来，我们需要在客户端应用程序的配置类中添加@ `EnableZuulProxy`注释:

```java
@Configuration
@EnableOAuth2Sso
@EnableZuulProxy
public class SiteSecurityConfigurer
  extends WebSecurityConfigurerAdapter {
    //...
}
```

剩下要做的就是将 Zuul 配置属性添加到我们的`application.yml`文件中:

```java
zuul:
  sensitiveHeaders: Cookie,Set-Cookie  
  routes:
    resource:
      path: /api/**
      url: http://localhost:9000
    user: 
      path: /user/**
      url: http://localhost:9000/user
```

任何到达客户端应用程序/ `api`端点的请求都将被重定向到资源服务器 URL。我们还需要提供用户凭证端点的 URL。

## 6。结论

在这篇简短的文章中，我们探讨了如何将 Spring Cloud Security 与 OAuth2 和 Zuul 一起使用来配置安全的授权和资源服务器，以及如何使用`Oauth2RestTemplate`和嵌入式 Zuul 代理在服务器之间中继 OAuth2 令牌。

和往常一样，代码可以在 GitHub 的[上获得。](https://web.archive.org/web/20220926192518/https://github.com/eugenp/tutorials/tree/master/spring-cloud-modules/spring-cloud-security)