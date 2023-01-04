# oauth 2 –@ EnableResourceServer vs @ enableoauth 2 SSO

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-security-oauth2-enable-resource-server-vs-enable-oauth2-sso>

## 1.概观

在本教程中，我们将讨论 Spring Security 中的`@EnableResourceServer`和`@EnableOAuth2Sso`注释。

我们将从解释 OAuth2 `Client`和 OAuth2 `Resource Server` 之间的**差异开始。随后，我们将讨论一下这些注释能为我们做些什么，并用一个使用 [`Zuul`](https://web.archive.org/web/20220627165704/https://cloud.spring.io/spring-cloud-netflix/multi/multi__router_and_filter_zuul.html) 和一个简单 API 的例子来演示它们的用法。**

出于本文的目的，我们将假设一些关于`Zuul`和`OAuth2`的预先存在的经验。

如果你没有，或者觉得回顾一下其中任何一个都会有帮助，请参考我们的[关于`Zuul`](/web/20220627165704/https://www.baeldung.com/spring-rest-with-zuul-proxy) 的快速概述和我们的[关于`OAuth2`](/web/20220627165704/https://www.baeldung.com/spring-security-oauth) 的指南。

## 2.OAuth2 客户端和资源服务器

OAuth2 中有四种不同的 [`roles`](https://web.archive.org/web/20220627165704/https://tools.ietf.org/html/rfc6749#page-6) 我们需要考虑:

*   **`Resource Owner`** —能够授权访问其受保护资源的实体
*   **`Authorization Server`** —成功认证`Resource` `Owners`并获得其授权后，授予`Clients`访问令牌
*   **`Resource Server`** —需要访问令牌来允许或至少考虑访问其资源的组件
*   **`Client`** —能够从授权服务器获得访问令牌的实体

用`@EnableResourceServer`或`@EnableOAuth2Sso`注释我们的配置类，指示 Spring 配置组件，将我们的应用程序转换成上面提到的后两种角色之一。

**`@EnableResourceServer`注释通过配置`[OAuth2AuthenticationProcessingFilter](https://web.archive.org/web/20220627165704/https://docs.spring.io/spring-security/oauth/apidocs/org/springframework/security/oauth2/provider/authentication/OAuth2AuthenticationProcessingFilter.html)`和其他同等重要的组件`.`，使我们的应用程序能够像`Resource Server`** 一样运行

查看 [`ResourceServerSecurityConfigurer`](https://web.archive.org/web/20220627165704/https://github.com/spring-projects/spring-security-oauth/blob/master/spring-security-oauth2/src/main/java/org/springframework/security/oauth2/config/annotation/web/configurers/ResourceServerSecurityConfigurer.java) 类，更好地了解幕后正在配置什么。

相反，**`@EnableOAuth2Sso`注释将我们的应用程序转换成 OAuth2 客户端**。它指示 Spring 配置一个 [`OAuth2ClientAuthenticationProcessingFilter`](https://web.archive.org/web/20220627165704/https://docs.spring.io/spring-security/oauth/apidocs/org/springframework/security/oauth2/client/filter/OAuth2ClientAuthenticationProcessingFilter.html) ，以及我们的应用程序需要能够从授权服务器获得访问令牌的其他组件。

看看 [`SsoSecurityConfigurer`](https://web.archive.org/web/20220627165704/https://github.com/spring-projects/spring-security-oauth2-boot/blob/master/spring-security-oauth2-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/security/oauth2/client/SsoSecurityConfigurer.java) 类，进一步了解 Spring 为我们配置了什么。

将这些注释与一些属性结合起来，使我们能够快速启动和运行。让我们创建两个不同的应用程序来看看它们的运行情况，以及它们如何相互补充:

*   我们的第一个应用程序将是我们的 edge 节点，一个简单的使用`@EnableOAuth2Sso`注释的`Zuul`应用程序。它将负责认证用户(在`Authorization` `Server`的帮助下)，并将传入的请求委派给其他应用程序
*   第二个应用程序将使用`@EnableResourceServer`注释，如果传入的请求包含有效的 OAuth2 访问令牌，它将允许访问受保护的资源

## 3.祖尔语-`@EnableOAuth2Sso`

让我们首先创建一个`Zuul`应用程序，它将充当我们的边缘节点，并负责使用 OAuth2 `Authorization` `Server`对用户进行身份验证:

```java
@Configuration
@EnableZuulProxy
@EnableOAuth2Sso
@Order(value = 0)
public class AppConfiguration extends WebSecurityConfigurerAdapter {

    @Autowired
    private ResourceServerTokenServices 
      resourceServerTokenServices;

    @Override
    public void configure(HttpSecurity http) throws Exception { 
        http.csrf().disable()
            .authorizeRequests()
            .antMatchers("/authorization-server-1/**",
              "/login").permitAll()
            .anyRequest().authenticated().and()
            .logout().permitAll().logoutSuccessUrl("/");
    }
}
```

用`@EnableOAuth2Sso`注释我们的`Zuul`应用程序也通知 Spring 配置一个`[OAuth2TokenRelayFilter](https://web.archive.org/web/20220627165704/https://javadoc.io/doc/org.springframework.cloud/spring-cloud-security/latest/org/springframework/cloud/security/oauth2/proxy/OAuth2TokenRelayFilter.html)`过滤器。该过滤器从用户的 HTTP 会话中检索先前获得的访问令牌，并将它们向下游传播。

注意，我们还在我们的`AppConfiguration`配置类中使用了 [**`@Order`**](https://web.archive.org/web/20220627165704/https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/core/annotation/Order.html) 注释。这是为了确保我们的 [`WebSecurityConfigurerAdapter`](https://web.archive.org/web/20220627165704/https://docs.spring.io/spring-security/site/docs/4.2.5.RELEASE/apidocs/org/springframework/security/config/annotation/web/configuration/WebSecurityConfigurerAdapter.html) 创建的`Filters`优先于其他`WebSecurityConfigurerAdapters`创建的`Filters`。

例如，我们可以用`@EnableResourceServer`注释我们的`Zuul`应用程序，以支持 HTTP 会话标识符和 OAuth2 访问令牌。然而，这样做会创建新的`Filters`，默认情况下，它优先于由`AppConfiguration`类创建的那些。这是因为由 [`@EnableResourceServer`](https://web.archive.org/web/20220627165704/https://github.com/spring-projects/spring-security-oauth/blob/master/spring-security-oauth2/src/main/java/org/springframework/security/oauth2/config/annotation/web/configuration/EnableResourceServer.java) 触发的配置类 [`ResouceServerConfiguration`](https://web.archive.org/web/20220627165704/https://github.com/spring-projects/spring-security-oauth/blob/master/spring-security-oauth2/src/main/java/org/springframework/security/oauth2/config/annotation/web/configuration/ResourceServerConfiguration.java) 指定默认值`order`为 3，而`WebSecurityConfigureAdapter`的默认值`order`为 100。

在我们进入`Resource` `Server,`之前，我们需要配置一些属性:

```java
zuul:
  routes:
    resource-server-mvc-1: /resource-server-mvc-1/**
    authorization-server-1:
      sensitiveHeaders: Authorization
      path: /authorization-server-1/**
      stripPrefix: false
  add-proxy-headers: true

security:
  basic:
    enabled: false
  oauth2:
    sso:
      loginPath: /login
    client:
      accessTokenUri: http://localhost:8769/authorization-server-1/oauth/token
      userAuthorizationUri: /authorization-server-1/oauth/authorize
      clientId: fooClient
      clientSecret: fooSecret
    resource:
      jwt:
        keyValue: "abc"
      id: fooScope
      serviceId: ${PREFIX:}resource
```

不涉及太多细节，使用此配置，我们可以:

*   配置我们的`Zuul`路由，并说明在向下游发送请求之前应该添加/删除哪些报头。
*   为我们的应用程序设置一些 OAuth2 属性，以便能够与我们的`Authorization` `Server`进行通信，并为 [`JWT`](/web/20220627165704/https://www.baeldung.com/spring-security-oauth-jwt) 配置`symmetric`加密。

## 4\. API – `@EnableResourceServer`

现在我们已经有了我们的`Zuul`应用程序，让我们创建我们的`Resource` `Server`:

```java
@SpringBootApplication
@EnableResourceServer
@Controller
@RequestMapping("/")
class ResourceServerApplication {

    public static void main(String[] args) {
        SpringApplication.run(ResourceServerApplication.class, args);
    }

    @RequestMapping(method = RequestMethod.GET)
    @ResponseBody
    public String helloWorld(Principal principal) {
        return "Hello " + principal.getName();
    }
}
```

这是一个简单的应用程序，它公开了一个端点来返回发起请求的`Principal`的`name`。

最后，我们来配置一些属性:

```java
security:
  basic:
    enabled: false
  oauth2:
    resource:
      jwt:
        keyValue: "abc"
      id: fooScope
      service-id: ${PREFIX:}resource
```

请记住，**我们需要一个有效的访问令牌**(存储在我们边缘节点中用户的 HTTP 会话中)**来访问我们`Resource` `Server`** 的端点。

## 5.结论

在本文中，我们解释了 **`@EnableOAuth2Sso`** 和 **`@EnableResourceServer`** 注释的区别。我们还通过一个使用`Zuul`和一个简单 API 的实际例子演示了如何使用它们。

这个例子的完整实现可以在 Github 上找到[。](https://web.archive.org/web/20220627165704/https://github.com/Baeldung/oauth-microservices/tree/master/1x)

在本地运行时，我们可以在`http://192.168.1.67:8765/resource-server-mvc-1`运行和测试应用程序