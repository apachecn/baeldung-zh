# 向 Reddit OAuth2 和 Spring Security 认证

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-security-oauth2-authentication-with-reddit>

## 1。概述

在本教程中，我们将使用 Spring Security OAuth 通过 Reddit API 进行身份验证。

## 2。Maven 配置

首先，为了使用 Spring Security OAuth——我们需要将以下依赖项添加到我们的`pom.xml` 中(当然还有您可能使用的任何其他 Spring 依赖项):

```java
<dependency>
    <groupId>org.springframework.security.oauth</groupId>
    <artifactId>spring-security-oauth2</artifactId>
    <version>2.0.6.RELEASE</version>
</dependency>
```

## 3。配置 OAuth2 客户端

接下来——让我们配置 OAuth2 客户端——`OAuth2RestTemplate`——和一个用于所有认证相关属性的`reddit.properties`文件:

```java
@Configuration
@EnableOAuth2Client
@PropertySource("classpath:reddit.properties")
protected static class ResourceConfiguration {

    @Value("${accessTokenUri}")
    private String accessTokenUri;

    @Value("${userAuthorizationUri}")
    private String userAuthorizationUri;

    @Value("${clientID}")
    private String clientID;

    @Value("${clientSecret}")
    private String clientSecret;

    @Bean
    public OAuth2ProtectedResourceDetails reddit() {
        AuthorizationCodeResourceDetails details = new AuthorizationCodeResourceDetails();
        details.setId("reddit");
        details.setClientId(clientID);
        details.setClientSecret(clientSecret);
        details.setAccessTokenUri(accessTokenUri);
        details.setUserAuthorizationUri(userAuthorizationUri);
        details.setTokenName("oauth_token");
        details.setScope(Arrays.asList("identity"));
        details.setPreEstablishedRedirectUri("http://localhost/login");
        details.setUseCurrentUri(false);
        return details;
    }

    @Bean
    public OAuth2RestTemplate redditRestTemplate(OAuth2ClientContext clientContext) {
        OAuth2RestTemplate template = new OAuth2RestTemplate(reddit(), clientContext);
        AccessTokenProvider accessTokenProvider = new AccessTokenProviderChain(
          Arrays.<AccessTokenProvider> asList(
            new MyAuthorizationCodeAccessTokenProvider(), 
            new ImplicitAccessTokenProvider(), 
            new ResourceOwnerPasswordAccessTokenProvider(),
            new ClientCredentialsAccessTokenProvider())
        );
        template.setAccessTokenProvider(accessTokenProvider);
        return template;
    }

}
```

还有“`reddit.properties`”:

```java
clientID=xxxxxxxx
clientSecret=xxxxxxxx
accessTokenUri=https://www.reddit.com/api/v1/access_token
userAuthorizationUri=https://www.reddit.com/api/v1/authorize
```

你可以通过创建一个来自[https://www.reddit.com/prefs/apps/](https://web.archive.org/web/20220627182552/https://www.reddit.com/prefs/apps/)的 Reddit 应用程序来获得你自己的密码

我们将使用`OAuth2RestTemplate`来:

1.  获取访问远程资源所需的访问令牌。
2.  获得访问令牌后访问远程资源。

还要注意我们是如何将范围“`identity`”添加到 Reddit `OAuth2ProtectedResourceDetails`中的，以便我们稍后可以检索用户的帐户信息。

## 4。`AuthorizationCodeAccessTokenProvider`风俗

Reddit OAuth2 实现与标准略有不同。因此——而不是优雅地扩展`AuthorizationCodeAccessTokenProvider`——我们实际上需要覆盖它的一些部分。

有 github 问题跟踪改进将使这一点没有必要，但这些问题尚未完成。

Reddit 做的一件非标准的事情是——当我们重定向用户并提示他向 Reddit 认证时，我们需要在重定向 URL 中有一些自定义参数。更具体地说——如果我们从 Reddit 请求一个永久访问令牌——我们需要添加一个值为“`permanent`”的参数“`duration`”。

因此，在扩展了`AuthorizationCodeAccessTokenProvider`之后，我们在`getRedirectForAuthorization()`方法中添加了这个参数:

```java
 requestParameters.put("duration", "permanent");
```

你可以从[这里](https://web.archive.org/web/20220627182552/https://github.com/Baeldung/reddit-app/tree/master/reddit-common/src/main/java/org/baeldung/security)查看完整的源代码。

## 5。 `ServerInitializer`

接下来，让我们创建我们的定制`ServerInitializer`。

我们需要添加一个 id 为`oauth2ClientContextFilter`的过滤 bean，这样我们就可以用它来存储当前的上下文:

```java
public class ServletInitializer extends AbstractDispatcherServletInitializer {

    @Override
    protected WebApplicationContext createServletApplicationContext() {
        AnnotationConfigWebApplicationContext context = 
          new AnnotationConfigWebApplicationContext();
        context.register(WebConfig.class, SecurityConfig.class);
        return context;
    }

    @Override
    protected String[] getServletMappings() {
        return new String[] { "/" };
    }

    @Override
    protected WebApplicationContext createRootApplicationContext() {
        return null;
    }

    @Override
    public void onStartup(ServletContext servletContext) throws ServletException {
        super.onStartup(servletContext);
        registerProxyFilter(servletContext, "oauth2ClientContextFilter");
        registerProxyFilter(servletContext, "springSecurityFilterChain");
    }

    private void registerProxyFilter(ServletContext servletContext, String name) {
        DelegatingFilterProxy filter = new DelegatingFilterProxy(name);
        filter.setContextAttribute(
          "org.springframework.web.servlet.FrameworkServlet.CONTEXT.dispatcher");
        servletContext.addFilter(name, filter).addMappingForUrlPatterns(null, false, "/*");
    }
}
```

## 6。MVC 配置

现在，让我们来看看我们简单的 web 应用程序的 MVC 配置:

```java
@Configuration
@EnableWebMvc
@ComponentScan(basePackages = { "org.baeldung.web" })
public class WebConfig implements WebMvcConfigurer {

    @Bean
    public static PropertySourcesPlaceholderConfigurer 
      propertySourcesPlaceholderConfigurer() {
        return new PropertySourcesPlaceholderConfigurer();
    }

    @Bean
    public ViewResolver viewResolver() {
        InternalResourceViewResolver viewResolver = new InternalResourceViewResolver();
        viewResolver.setPrefix("/WEB-INF/jsp/");
        viewResolver.setSuffix(".jsp");
        return viewResolver;
    }

    @Override
    public void configureDefaultServletHandling(
      DefaultServletHandlerConfigurer configurer) {
        configurer.enable();
    }

    public void addResourceHandlers(ResourceHandlerRegistry registry) {
        registry.addResourceHandler("/resources/**").addResourceLocations("/resources/");
    }

    @Override
    public void addViewControllers(ViewControllerRegistry registry) {
        registry.addViewController("/home.html");
    }
}
```

## 7。安全配置

接下来，让我们来看一下**主要的 Spring 安全配置**:

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {

    @Override
    protected void configure(AuthenticationManagerBuilder auth) 
      throws Exception {
        auth.inMemoryAuthentication();
    }

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http
            .anonymous().disable()
            .csrf().disable()
            .authorizeRequests()
            .antMatchers("/home.html").hasRole("USER")
            .and()
            .httpBasic()
            .authenticationEntryPoint(oauth2AuthenticationEntryPoint());
    }

    private LoginUrlAuthenticationEntryPoint oauth2AuthenticationEntryPoint() {
        return new LoginUrlAuthenticationEntryPoint("/login");
    }
}
```

注意:我们添加了一个简单的安全配置，该配置重定向到“`/login`”，它获取用户信息并从中加载身份验证——如下一节所述。

## 8。`RedditController`

现在，让我们来看看我们的控制器`RedditController`。

我们使用方法`redditLogin()`从他的 Reddit 帐户获取用户信息，并从中加载一个身份验证——如下例所示:

```java
@Controller
public class RedditController {

    @Autowired
    private OAuth2RestTemplate redditRestTemplate;

    @RequestMapping("/login")
    public String redditLogin() {
        JsonNode node = redditRestTemplate.getForObject(
          "https://oauth.reddit.com/api/v1/me", JsonNode.class);
        UsernamePasswordAuthenticationToken auth = 
          new UsernamePasswordAuthenticationToken(node.get("name").asText(), 
          redditRestTemplate.getAccessToken().getValue(), 
          Arrays.asList(new SimpleGrantedAuthority("ROLE_USER")));

        SecurityContextHolder.getContext().setAuthentication(auth);
        return "redirect:home.html";
    }

}
```

这个看似简单的方法有一个有趣的细节 reddit 模板**在执行任何请求**之前检查访问令牌是否可用；如果令牌不可用，它将获取一个令牌。

接下来，我们将信息呈现给非常简单的前端。

## 9。`home.jsp`

最后，让我们看一下`home.jsp`，显示从用户的 Reddit 帐户中检索到的信息:

```java
<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core"%>
<%@ taglib prefix="sec" uri="http://www.springframework.org/security/tags"%>
<html>
<body>
    <h1>Welcome, <small><sec:authentication property="principal.username" /></small></h1>
</body>
</html>
```

## 10。结论

在这篇介绍性文章中，我们探索了用 Reddit OAuth2 API 进行认证，并在一个简单的前端显示一些非常基本的信息。

现在我们已经通过了身份验证，我们将在这个新系列的下一篇文章中探索用 Reddit API 做更多有趣的事情。

本教程的**完整实现**可以在[的 github 项目](https://web.archive.org/web/20220627182552/https://github.com/Baeldung/reddit-app "The Full Spring / Reddit Example Project on Github")中找到——这是一个基于 Eclipse 的项目，所以它应该很容易导入和运行。