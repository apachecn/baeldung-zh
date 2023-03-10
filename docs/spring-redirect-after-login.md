# 使用 Spring Security 登录后重定向到不同的页面

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-redirect-after-login>

## 1。概述

web 应用程序的一个常见需求是**在登录后将不同类型的用户重定向到不同的页面**。例如，将标准用户重定向到一个`/homepage.html`页面，将管理员用户重定向到一个`/console.html`页面。

本文将展示如何使用 Spring Security 快速安全地实现这一机制。这篇文章也是建立在 Spring MVC 教程的基础上的，该教程讨论了如何设置项目所需的核心 MVC 内容。

## 2。春季安全配置

Spring Security 提供了一个组件，它直接负责决定在成功认证之后做什么——`AuthenticationSuccessHandler`。

### 2.1.基本配置

让我们首先配置一个基本的`@Configuration`和`@Service`类:

```java
@Configuration
@EnableWebSecurity
public class SecSecurityConfig {

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            .authorizeRequests()
            // ... endpoints
            .formLogin()
                .loginPage("/login.html")
                .loginProcessingUrl("/login")
                .defaultSuccessUrl("/homepage.html", true)
            // ... other configuration   
        return http.build();
    }
}
```

这个配置中需要关注的部分是`defaultSuccessUrl()`方法。成功登录后，任何用户都将被重定向到`homepage.html`。

此外，我们需要配置用户及其角色。出于本文的目的，我们将实现一个简单的有两个用户的`UserDetailService`，每个用户有一个角色。关于这个话题的更多信息，请阅读我们的文章[Spring Security——角色和特权](/web/20221023124647/https://www.baeldung.com/role-and-privilege-for-spring-security-registration)。

```java
@Service
public class MyUserDetailsService implements UserDetailsService {

    private Map<String, User> roles = new HashMap<>();

    @PostConstruct
    public void init() {
        roles.put("admin2", new User("admin", "{noop}admin1", getAuthority("ROLE_ADMIN")));
        roles.put("user2", new User("user", "{noop}user1", getAuthority("ROLE_USER")));
    }

    @Override
    public UserDetails loadUserByUsername(String username) {
        return roles.get(username);
    }

    private List<GrantedAuthority> getAuthority(String role) {
        return Collections.singletonList(new SimpleGrantedAuthority(role));
    }
} 
```

还要注意，在这个简单的例子中，我们不会使用密码编码器，因此[密码的前缀是`{noop}`](/web/20221023124647/https://www.baeldung.com/spring-security-5-default-password-encoder) 。

### 2.2.添加自定义成功处理程序

我们现在有两个用户，他们有两个不同的角色:`user`和`admin`。成功登录后，两者都会被重定向到`hompeage.html`。**让我们看看如何根据用户的角色进行不同的重定向。**

首先，我们需要将自定义成功处理程序定义为 bean:

```java
@Bean
public AuthenticationSuccessHandler myAuthenticationSuccessHandler(){
    return new MySimpleUrlAuthenticationSuccessHandler();
} 
```

然后用`successHandler`方法替换`defaultSuccessUrl`调用，该方法接受我们的自定义成功处理程序作为参数:

```java
@Bean
public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
    http
        .authorizeRequests()
        // endpoints
        .formLogin()
            .loginPage("/login.html")
            .loginProcessingUrl("/login")
            .successHandler(myAuthenticationSuccessHandler())
        // other configuration      
    return http.build();
} 
```

### 2.3.XML 配置

在查看我们的自定义成功处理程序的实现之前，让我们先来看看等效的 XML 配置:

```java
<http use-expressions="true" >
    <!-- other configuration -->
    <form-login login-page='/login.html' 
      authentication-failure-url="/login.html?error=true"
      authentication-success-handler-ref="myAuthenticationSuccessHandler"/>
    <logout/>
</http>

<beans:bean id="myAuthenticationSuccessHandler"
  class="com.baeldung.security.MySimpleUrlAuthenticationSuccessHandler" />

<authentication-manager>
    <authentication-provider>
        <user-service>
            <user name="user1" password="{noop}user1Pass" authorities="ROLE_USER" />
            <user name="admin1" password="{noop}admin1Pass" authorities="ROLE_ADMIN" />
        </user-service>
    </authentication-provider>
</authentication-manager>
```

## 3。自定义认证成功处理程序

除了`AuthenticationSuccessHandler`接口，Spring 还为这个策略组件提供了一个合理的缺省值——`AbstractAuthenticationTargetUrlRequestHandler`和一个简单的实现——`SimpleUrlAuthenticationSuccessHandler`。通常，这些实现将在登录后确定 URL，并执行到该 URL 的重定向。

虽然有点灵活，但确定该目标 URL 的机制不允许以编程方式进行确定——因此我们将实现该接口，并提供成功处理程序的自定义实现。**这个实现将根据用户的角色确定用户登录后重定向到的 URL。**

首先，我们需要覆盖`onAuthenticationSuccess`方法:

```java
public class MySimpleUrlAuthenticationSuccessHandler
  implements AuthenticationSuccessHandler {

    protected Log logger = LogFactory.getLog(this.getClass());

    private RedirectStrategy redirectStrategy = new DefaultRedirectStrategy();

    @Override
    public void onAuthenticationSuccess(HttpServletRequest request, 
      HttpServletResponse response, Authentication authentication)
      throws IOException {

        handle(request, response, authentication);
        clearAuthenticationAttributes(request);
    } 
```

我们的定制方法调用两个助手方法:

```java
protected void handle(
        HttpServletRequest request,
        HttpServletResponse response, 
        Authentication authentication
) throws IOException {

    String targetUrl = determineTargetUrl(authentication);

    if (response.isCommitted()) {
        logger.debug(
                "Response has already been committed. Unable to redirect to "
                        + targetUrl);
        return;
    }

    redirectStrategy.sendRedirect(request, response, targetUrl);
} 
```

其中下面的方法执行实际工作并将用户映射到目标 URL:

```java
protected String determineTargetUrl(final Authentication authentication) {

    Map<String, String> roleTargetUrlMap = new HashMap<>();
    roleTargetUrlMap.put("ROLE_USER", "/homepage.html");
    roleTargetUrlMap.put("ROLE_ADMIN", "/console.html");

    final Collection<? extends GrantedAuthority> authorities = authentication.getAuthorities();
    for (final GrantedAuthority grantedAuthority : authorities) {
        String authorityName = grantedAuthority.getAuthority();
        if(roleTargetUrlMap.containsKey(authorityName)) {
            return roleTargetUrlMap.get(authorityName);
        }
    }

    throw new IllegalStateException();
} 
```

请注意，该方法将返回用户拥有的第一个角色的映射 URL。因此，如果一个用户有多个角色，映射的 URL 将是与`authorities`集合中给出的第一个角色相匹配的那个。

```java
protected void clearAuthenticationAttributes(HttpServletRequest request) {
    HttpSession session = request.getSession(false);
    if (session == null) {
        return;
    }
    session.removeAttribute(WebAttributes.AUTHENTICATION_EXCEPTION);
}
```

策略的核心是`determineTargetUrl`,它只是查看用户的类型(由权威机构决定),然后**根据这个角色**选择目标 URL。

因此，由`ROLE_ADMIN`权限决定的**管理员用户**登录后将被重定向到控制台页面，而由`ROLE_USER`决定的**标准用户**将被重定向到主页。

## 4。结论

和往常一样，本文中的代码可以从 GitHub 上的[处获得。这是一个基于 Maven 的项目，因此应该很容易导入和运行。](https://web.archive.org/web/20221023124647/https://github.com/eugenp/tutorials/tree/master/spring-security-modules/spring-security-web-mvc-custom)