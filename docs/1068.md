# 安全性和 WebSockets 简介

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-security-websockets>

 ![](img/fc14276f0c992f2bda1dba30766d7d04.png)

It’s just plain hard to get true, **real-time visibility into a running auth flow.**

Parts of the process can be completely hidden from us; if the complete authorization process requires a redirect from a remote OAuth production server, then every debugging effort must go through the production server.

It’s practically unfeasible to debug this locally. There’s no way to reproduce the exact state and no way to inspect what is actually happening under the hood. Not ideal.

Knowing these types of challenges, we built Lightrun - a real-time production debugging tool - to allow you to understand complicated flows with code-level information. Add logs, take snapshots (virtual breakpoints), and instrument metrics without a remote debugger, without stopping the running service, and, most importantly - **in real-time and without side effects**.

**Learn more with this 5-minute tutorial** focused on debugging these kinds of scenarios using Lightrun:

[>> Debugging Authentication and Authorization Using Lightrun](/web/20220524035233/https://www.baeldung.com/lightrun-n-security)

## 1。简介

在之前的一篇文章中，我们展示了如何将 WebSockets 添加到 Spring MVC 项目中。

在这里，我们将描述如何在 Spring MVC 中**给 Spring WebSockets 添加安全性。在继续之前，确保你已经有了基本的 Spring MVC 安全覆盖——如果没有，查看[这篇文章](/web/20220524035233/https://www.baeldung.com/spring-security-basic-authentication)。**

## 2。Maven 依赖关系

我们的 WebSocket 实现需要两组主要的 Maven 依赖关系。

首先，让我们指定我们将使用的 Spring 框架和 Spring 安全的主要版本:

```
<properties>
    <spring.version>5.3.13</spring.version>
    <spring-security.version>5.6.0</spring-security.version>
</properties>
```

其次，让我们添加实现基本身份验证和授权所需的核心 Spring MVC 和 Spring 安全库:

```
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-core</artifactId>
    <version>${spring.version}</version>
</dependency>
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-web</artifactId>
    <version>${spring.version}</version>
</dependency>
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-webmvc</artifactId>
    <version>${spring.version}</version>
</dependency>
<dependency>
    <groupId>org.springframework.security</groupId>
    <artifactId>spring-security-web</artifactId>
    <version>${spring-security.version}</version>
</dependency>
<dependency>
    <groupId>org.springframework.security</groupId>
    <artifactId>spring-security-config</artifactId>
    <version>${spring-security.version}</version>
</dependency> 
```

最新版本的 [spring-core](https://web.archive.org/web/20220524035233/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.springframework%22%20AND%20a%3A%22spring-core%22) 、 [spring-web](https://web.archive.org/web/20220524035233/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22org.springframework%22%20AND%20a%3A%22spring-web%22) 、 [spring-webmvc](https://web.archive.org/web/20220524035233/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22org.springframework%22%20AND%20a%3A%22spring-webmvc%22) 、 [spring-security-web](https://web.archive.org/web/20220524035233/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22%20org.springframework%22%20AND%20a%3A%22spring-security-web%22) 、 [spring-security-config](https://web.archive.org/web/20220524035233/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22%20org.springframework%22%20AND%20a%3A%22spring-security-config%22) 可以在 Maven Central 上找到。

最后，让我们添加所需的依赖项:

```
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-websocket</artifactId>
    <version>${spring.version}</version>
</dependency>
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-messaging</artifactId>
    <version>${spring.version}</version>
</dependency>
<dependency>
    <groupId>org.springframework.security</groupId>
    <artifactId>spring-security-messaging</artifactId>
    <version>${spring-security.version}</version>
</dependency> 
```

你可以在 Maven Central 上找到最新版本的 [spring-websocket](https://web.archive.org/web/20220524035233/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22org.springframework%22%20AND%20a%3A%22spring-websocket%22) 、 [spring-messaging](https://web.archive.org/web/20220524035233/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22org.springframework%22%20AND%20a%3A%22spring-messaging%22) 和[spring-security-messaging](https://web.archive.org/web/20220524035233/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22org.springframework%22%20AND%20a%3A%22spring-security-messaging%22)。

## 3。基本 WebSocket 安全性

使用 `spring-security-messaging` 库的特定于 WebSocket 的安全性以`AbstractSecurityWebSocketMessageBrokerConfigurer`类及其在项目中的实现为中心:

```
@Configuration
public class SocketSecurityConfig 
  extends AbstractSecurityWebSocketMessageBrokerConfigurer {
      //...
}
```

`AbstractSecurityWebSocketMessageBrokerConfigurer` 级**提供由`WebSecurityConfigurerAdapter.`提供的额外安全覆盖**

`spring-security-messaging` 库并不是实现 WebSockets 安全性的唯一方法。如果我们坚持使用普通的`spring-websocket`库，我们可以实现`WebSocketConfigurer`接口并将安全拦截器附加到我们的套接字处理程序上。

因为我们使用的是`spring-security-messaging`库，所以我们将使用`AbstractSecurityWebSocketMessageBrokerConfigurer` 方法。

### 3.1。实施`configureInbound()`

`configureInbound()`的实现是配置`AbstractSecurityWebSocketMessageBrokerConfigurer` 子类中最重要的一步:

```
@Override 
protected void configureInbound(
  MessageSecurityMetadataSourceRegistry messages) { 
    messages
      .simpDestMatchers("/secured/**").authenticated()
      .anyMessage().authenticated(); 
}
```

虽然`WebSecurityConfigurerAdapter` 允许您为不同的路由指定各种应用程序范围的授权需求，但是`AbstractSecurityWebSocketMessageBrokerConfigurer` 允许您为套接字目的地指定特定的授权需求。

### 3.2。类型和目的地匹配

`MessageSecurityMetadataSourceRegistry` 允许我们指定安全约束，如路径、用户角色以及允许哪些消息。

**类型匹配器约束哪些`SimpMessageType` 被允许**以及以何种方式 **:**

```
.simpTypeMatchers(CONNECT, UNSUBSCRIBE, DISCONNECT).permitAll()
```

**目的地匹配器约束哪些端点模式是可访问的**以及以何种方式访问 **:**

```
.simpDestMatchers("/app/**").hasRole("ADMIN")
```

**订阅目的地匹配器映射一个`List`** `of` `SimpDestinationMessageMatcher i`匹配于 `SimpMessageType.SUBSCRIBE:` 的实例

```
.simpSubscribeDestMatchers("/topic/**").authenticated()
```

这里是[类型和目的匹配](https://web.archive.org/web/20220524035233/https://docs.spring.io/spring-security/site/docs/current/api/org/springframework/security/config/annotation/web/messaging/MessageSecurityMetadataSourceRegistry.html)的所有可用方法的完整列表。

## 4。保护套接字路由

现在我们已经了解了基本的套接字安全性和类型匹配配置，我们可以将套接字安全性、视图、STOMP(一种文本消息协议)、消息代理和套接字控制器结合起来，在我们的 Spring MVC 应用程序中实现安全的 WebSockets。

首先，让我们为基本的 Spring 安全覆盖设置我们的套接字视图和控制器:

```
@Configuration
@EnableGlobalMethodSecurity(prePostEnabled = true, securedEnabled = true)
@EnableWebSecurity
@ComponentScan("com.baeldung.springsecuredsockets")
public class SecurityConfig extends WebSecurityConfigurerAdapter { 
    @Override 
    protected void configure(HttpSecurity http) throws Exception { 
        http
          .authorizeRequests()
          .antMatchers("/", "/index", "/authenticate").permitAll()
          .antMatchers(
            "/secured/**/**",
            "/secured/success", 
            "/secured/socket",
            "/secured/success").authenticated()
          .anyRequest().authenticated()
          .and()
          .formLogin()
          .loginPage("/login").permitAll()
          .usernameParameter("username")
          .passwordParameter("password")
          .loginProcessingUrl("/authenticate")
          //...
    }
}
```

其次，让我们设置具有身份验证要求的实际消息目的地:

```
@Configuration
public class SocketSecurityConfig 
  extends AbstractSecurityWebSocketMessageBrokerConfigurer {
    @Override
    protected void configureInbound(MessageSecurityMetadataSourceRegistry messages) {
        messages
          .simpDestMatchers("/secured/**").authenticated()
          .anyMessage().authenticated();
    }   
}
```

现在，在我们的`WebSocketMessageBrokerConfigurer,` 中，我们可以注册实际的消息和 STOMP 端点:

```
@Configuration
@EnableWebSocketMessageBroker
public class SocketBrokerConfig 
  implements WebSocketMessageBrokerConfigurer {

    @Override
    public void configureMessageBroker(MessageBrokerRegistry config) {
        config.enableSimpleBroker("/secured/history");
        config.setApplicationDestinationPrefixes("/spring-security-mvc-socket");
    }

    @Override
    public void registerStompEndpoints(StompEndpointRegistry registry) {
        registry.addEndpoint("/secured/chat")
          .withSockJS();
    }
}
```

让我们定义**一个示例套接字控制器**和端点，我们在上面提供了安全覆盖:

```
@Controller
public class SocketController {

    @MessageMapping("/secured/chat")
    @SendTo("/secured/history")
    public OutputMessage send(Message msg) throws Exception {
        return new OutputMessage(
           msg.getFrom(),
           msg.getText(), 
           new SimpleDateFormat("HH:mm").format(new Date())); 
    }
}
```

## 5。同源策略

**同源策略**要求与端点的所有交互必须来自发起交互的同一个域。

例如，假设您的 WebSockets 实现托管在`foo.com`，并且您正在**执行同源策略**。如果一个用户连接到你在`foo.com`托管的客户端，然后打开另一个浏览器到`bar.com`，那么`bar.com`将不能访问你的 WebSocket 实现。

### 5.1。覆盖同源策略

Spring WebSockets 执行开箱即用的同源策略，而普通的 WebSockets 没有。

事实上， **Spring Security 对于任何有效的 `CONNECT`消息类型都需要一个 CSRF ( `Cross Site Request Forgery`)令牌**:

```
@Controller
public class CsrfTokenController {
    @GetMapping("/csrf")
    public @ResponseBody String getCsrfToken(HttpServletRequest request) {
        CsrfToken csrf = (CsrfToken) request.getAttribute(CsrfToken.class.getName());
        return csrf.getToken();
    }
}
```

通过在`/csrf`调用端点，客户端可以获得令牌并通过 CSRF 安全层进行身份验证。

然而，Spring 的**同源策略可以通过向您的`AbstractSecurityWebSocketMessageBrokerConfigurer`添加以下配置来覆盖**:

```
@Override
protected boolean sameOriginDisabled() {
    return true;
}
```

### 5.2。踏脚、SockJS 支持和框架选项

通常使用 [STOMP](https://web.archive.org/web/20220524035233/http://jmesnil.net/stomp-websocket/doc/) 和 [SockJS](https://web.archive.org/web/20220524035233/https://github.com/sockjs) 来实现 Spring WebSockets 的客户端支持。

默认情况下，SockJS 被配置为不允许通过 HTML `iframe`元素进行传输。这是为了防止点击劫持的威胁。

然而，在某些用例中，允许`iframes`利用 SockJS 传输是有益的。为此，您可以覆盖`WebSecurityConfigurerAdapter`中的默认配置:

```
@Override
protected void configure(HttpSecurity http) 
  throws Exception {
    http
      .csrf()
        //...
        .and()
      .headers()
        .frameOptions().sameOrigin()
      .and()
        .authorizeRequests();
}
```

请注意，在本例中，尽管允许通过`iframes`进行运输，我们仍遵循**同源策略**。

## 6。Oauth2 覆盖率

对 Spring WebSockets 的 Oauth2 特定支持是通过实现 Oauth2 安全覆盖来实现的，这是对标准覆盖`WebSecurityConfigurerAdapter` 的补充`.` [这里有一个如何实现 Oauth2 的例子。](/web/20220524035233/https://www.baeldung.com/rest-api-spring-oauth2-angular)

要对 WebSocket 端点进行身份验证并获得访问权，可以在从客户端连接到后端 WebSocket 时将 Oauth2 `access_token`传递到查询参数中。

下面是一个使用 SockJS 和 STOMP 演示这一概念的示例:

```
var endpoint = '/ws/?access_token=' + auth.access_token;
var socket = new SockJS(endpoint);
var stompClient = Stomp.over(socket);
```

## 7 .**。结论**

在这个简短的教程中，我们展示了如何为 Spring WebSockets 添加安全性。如果你想了解更多关于这种集成的信息，可以看看 Spring 的 WebSocket 和 WebSocket Security 参考文档。

和往常一样，查看我们的 Github 项目中本文使用的例子。**