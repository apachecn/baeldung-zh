# Spring Social 的二级脸书登录

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/facebook-authentication-with-spring-security-and-social>

## 1。概述

在本教程中，我们将重点关注向现有的表单登录应用程序添加一个新的脸书登录。

我们将使用 Spring Social support 与脸书互动，保持事情的简洁明了。

## 2.Maven 配置

首先，我们需要将`spring-social-facebook`依赖项添加到我们的`pom.xml`中:

```
<dependency>
    <groupId>org.springframework.social</groupId>
    <artifactId>spring-social-facebook</artifactId>
    <version>2.0.3.RELEASE</version>
</dependency>
```

## 3.安全配置–只是表单登录

让我们首先从简单的安全配置开始，这里我们只有基于表单的身份验证:

```
@Configuration
@EnableWebSecurity
@ComponentScan(basePackages = { "com.baeldung.security" })
public class SecurityConfig extends WebSecurityConfigurerAdapter {

    @Autowired
    private UserDetailsService userDetailsService;

    @Override
    protected void configure(AuthenticationManagerBuilder auth) 
      throws Exception {
        auth.userDetailsService(userDetailsService);
    }

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http
        .csrf().disable()
        .authorizeRequests()
        .antMatchers("/login*").permitAll()
        .anyRequest().authenticated()
        .and()
        .formLogin().loginPage("/login").permitAll();
    } 
}
```

我们不会在这个配置上花太多时间——如果你想更好地理解它，请看一下[表单登录文章](/web/20220822105736/https://www.baeldung.com/spring-security-login)。

## 4.脸书房产

接下来，让我们在`application.properties`中配置脸书属性:

```
spring.social.facebook.appId=YOUR_APP_ID
spring.social.facebook.appSecret=YOUR_APP_SECRET
```

请注意:

*   我们需要创建一个脸书应用程序来获取`appId`和`appSecret`
*   从脸书应用程序设置，确保添加平台“网站”和`http://localhost:8080/`是“网站网址”

## 5。安全配置–添加脸书

现在，让我们在系统中添加一种新的身份认证方式，由脸书推动:

```
public class SecurityConfig extends WebSecurityConfigurerAdapter {

    @Autowired
    private FacebookConnectionSignup facebookConnectionSignup;

    @Value("${spring.social.facebook.appSecret}")
    String appSecret;

    @Value("${spring.social.facebook.appId}")
    String appId;

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http
        .authorizeRequests()
        .antMatchers("/login*","/signin/**","/signup/**").permitAll()
        ...
    } 

    @Bean
    public ProviderSignInController providerSignInController() {
        ConnectionFactoryLocator connectionFactoryLocator = 
            connectionFactoryLocator();
        UsersConnectionRepository usersConnectionRepository = 
            getUsersConnectionRepository(connectionFactoryLocator);
        ((InMemoryUsersConnectionRepository) usersConnectionRepository)
            .setConnectionSignUp(facebookConnectionSignup);
        return new ProviderSignInController(connectionFactoryLocator, 
            usersConnectionRepository, new FacebookSignInAdapter());
    }

    private ConnectionFactoryLocator connectionFactoryLocator() {
        ConnectionFactoryRegistry registry = new ConnectionFactoryRegistry();
        registry.addConnectionFactory(new FacebookConnectionFactory(appId, appSecret));
        return registry;
    }

    private UsersConnectionRepository getUsersConnectionRepository(ConnectionFactoryLocator 
        connectionFactoryLocator) {
        return new InMemoryUsersConnectionRepository(connectionFactoryLocator);
    }
}
```

让我们仔细看看新的配置:

*   我们使用一个`ProviderSignInController`来启用脸书认证，这需要两件事:
    首先，一个`ConnectionFactoryLocator`用我们之前定义的脸书属性注册为一个`FacebookConnectionFactory`。
    第二，一个`InMemoryUsersConnectionRepository`。
*   通过向“`/signin/facebook`”发送`POST`——该控制器将使用脸书服务提供商启动用户登录
*   我们正在设置一个`SignInAdapter`来处理应用程序中的登录逻辑
*   我们还设置了一个`ConnectionSignUp`来处理注册用户，当他们第一次通过脸书认证时

## 6.登录适配器

简而言之，这个适配器是上面的控制器(驱动脸书用户登录流程)和我们特定的本地应用程序之间的桥梁:

```
public class FacebookSignInAdapter implements SignInAdapter {
    @Override
    public String signIn(
      String localUserId, 
      Connection<?> connection, 
      NativeWebRequest request) {

        SecurityContextHolder.getContext().setAuthentication(
          new UsernamePasswordAuthenticationToken(
          connection.getDisplayName(), null, 
          Arrays.asList(new SimpleGrantedAuthority("FACEBOOK_USER"))));

        return null;
    }
}
```

请注意，使用脸书登录的用户将拥有角色`FACEBOOK_USER`，而使用表单登录的用户将拥有角色`USER.` 

## 7.连接注册

当用户第一次通过脸书认证时，他们在我们的应用程序中没有现有的帐户。

这就是我们需要为他们自动创建帐户的地方；我们将使用`ConnectionSignUp`来驱动用户创建逻辑:

```
@Service
public class FacebookConnectionSignup implements ConnectionSignUp {

    @Autowired
    private UserRepository userRepository;

    @Override
    public String execute(Connection<?> connection) {
        User user = new User();
        user.setUsername(connection.getDisplayName());
        user.setPassword(randomAlphabetic(8));
        userRepository.save(user);
        return user.getUsername();
    }
}
```

如您所见，我们为新用户创建了一个帐户——使用他们的`DisplayName`作为用户名。

## 8.前端

最后，我们来看看我们的前端。

我们现在将在登录页面上支持这两种身份验证流程，即表单登录和脸书:

```
<html>
<body>
<div th:if="${param.logout}">You have been logged out</div>
<div th:if="${param.error}">There was an error, please try again</div>

<form th:action="@{/login}" method="POST" >
    <input type="text" name="username" />
    <input type="password" name="password" />
    <input type="submit" value="Login" />
</form>

<form action="/signin/facebook" method="POST">
    <input type="hidden" name="scope" value="public_profile" />
    <input type="submit" value="Login using Facebook"/>
</form>
</body>
</html>
```

最后，这里是`index.html`:

```
<html>
<body>
<nav>
    <p sec:authentication="name">Username</p>      
    <a th:href="@{/logout}">Logout</a>                     
</nav>

<h1>Welcome, <span sec:authentication="name">Username</span></h1>
<p sec:authentication="authorities">User authorities</p>
</body>
</html>
```

注意这个索引页面是如何显示用户名和权限的。

就这样，我们现在有两种方法来验证应用程序。

## 9.结论

在这篇简短的文章中，我们学习了如何使用`spring-social-facebook`为我们的应用程序实现二次认证流程。

当然，和往常一样，源代码在 GitHub 上完全可用[。](https://web.archive.org/web/20220822105736/https://github.com/eugenp/tutorials/tree/master/spring-security-modules/spring-security-social-login)