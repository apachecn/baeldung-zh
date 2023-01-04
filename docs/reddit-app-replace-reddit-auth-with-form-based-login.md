# 在 Reddit 应用程序中将注册与登录分离

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/reddit-app-replace-reddit-auth-with-form-based-login>

## 1。概述

在本教程中—**我们将用一个更简单的基于表单的登录**来取代 Reddit 支持的 OAuth2 认证过程。

在我们登录之后，我们仍然能够将 Reddit 连接到应用程序,我们只是不使用 Reddit 来驱动我们的主登录流程。

## 2。基本用户注册

首先，让我们替换旧的认证流程。

### 2.1。`User`实体

我们将对用户实体进行一些更改:使`username`唯一，添加一个`password`字段(临时) :

```java
@Entity
public class User {
    ...

    @Column(nullable = false, unique = true)
    private String username;

    private String password;

    ...
}
```

### 2.2。注册新用户

接下来，让我们看看如何在后端注册新用户:

```java
@Controller
@RequestMapping(value = "/user")
public class UserController {

    @Autowired
    private UserService service;

    @RequestMapping(value = "/register", method = RequestMethod.POST)
    @ResponseStatus(HttpStatus.OK)
    public void register(
      @RequestParam("username") String username, 
      @RequestParam("email") String email,
      @RequestParam("password") String password) 
    {
        service.registerNewUser(username, email, password);
    }
}
```

显然，这对于用户来说是一个基本的创建操作——没有多余的东西。

下面是服务层中的**实际实现:**

```java
@Service
public class UserService {
    @Autowired
    private UserRepository userRepository;

    @Autowired
    private PreferenceRepository preferenceReopsitory;

    @Autowired
    private PasswordEncoder passwordEncoder;

    @Override
    public void registerNewUser(String username, String email, String password) {
        User existingUser = userRepository.findByUsername(username);
        if (existingUser != null) {
            throw new UsernameAlreadyExistsException("Username already exists");
        }

        User user = new User();
        user.setUsername(username);
        user.setPassword(passwordEncoder.encode(password));
        Preference pref = new Preference();
        pref.setTimezone(TimeZone.getDefault().getID());
        pref.setEmail(email);
        preferenceReopsitory.save(pref);
        user.setPreference(pref);
        userRepository.save(user);
    }
}
```

### 2.3。异常处理

还有简单的`UserAlreadyExistsException`:

```java
public class UsernameAlreadyExistsException extends RuntimeException {

    public UsernameAlreadyExistsException(String message) {
        super(message);
    }
    public UsernameAlreadyExistsException(String message, Throwable cause) {
        super(message, cause);
    }
}
```

该异常在应用程序的主异常处理程序**中处理:**

```java
@ExceptionHandler({ UsernameAlreadyExistsException.class })
public ResponseEntity<Object> 
  handleUsernameAlreadyExists(RuntimeException ex, WebRequest request) {
    logger.error("400 Status Code", ex);
    String bodyOfResponse = ex.getLocalizedMessage();
    return new 
      ResponseEntity<Object>(bodyOfResponse, new HttpHeaders(), HttpStatus.BAD_REQUEST);
}
```

### 2.4。一个简单的注册页面

最后，一个简单的前端`signup.html`:

```java
<form>
    <input  id="username"/>
    <input  id="email"/>
    <input type="password" id="password" />
    <button onclick="register()">Sign up</button>
</form>

<script>
function register(){
    $.post("user/register", {username: $("#username").val(),
      email: $("#email").val(), password: $("#password").val()}, 
      function (data){
        window.location.href= "./";
    }).fail(function(error){
        alert("Error: "+ error.responseText);
    }); 
}
</script>
```

值得一提的是，这并不是一个完全成熟的注册流程，只是一个非常快速的流程。要了解完整的注册流程，您可以在 Baeldung 查看主注册系列。

## 3。新登录页面

这是我们的**新的简单登录页面**:

```java
<div th:if="${param.containsKey('error')}">
Invalid username or password
</div>
<form method="post" action="j_spring_security_check">
    <input name="username" />
    <input type="password" name="password"/>  
    <button type="submit" >Login</button>
</form>
<a href="signup">Sign up</a>
```

## 4。安全配置

现在，让我们来看看**新的安全配置**:

```java
@Configuration
@EnableWebSecurity
@ComponentScan({ "org.baeldung.security" })
public class SecurityConfig extends WebSecurityConfigurerAdapter {

    @Autowired
    private MyUserDetailsService userDetailsService;

    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        auth.userDetailsService(userDetailsService).passwordEncoder(encoder());
    }

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http
            ...
            .formLogin()
            .loginPage("/")
            .loginProcessingUrl("/j_spring_security_check")
            .defaultSuccessUrl("/home")
            .failureUrl("/?error=true")
            .usernameParameter("username")
            .passwordParameter("password")
            ...
    }

    @Bean
    public PasswordEncoder encoder() { 
        return new BCryptPasswordEncoder(11); 
    }
}
```

大多数事情都很简单，所以我们不会在这里详细讨论。

这里有一个习俗:

```java
@Service
public class MyUserDetailsService implements UserDetailsService {

    @Autowired
    private UserRepository userRepository;

    @Override
    public UserDetails loadUserByUsername(String username) {
        User user = userRepository.findByUsername(username); 
        if (user == null) { 
            throw new UsernameNotFoundException(username);
        } 
        return new UserPrincipal(user);
    }
}
```

这是我们的习俗`Principal``UserPrincipal”`，它实现了`UserDetails`:

```java
public class UserPrincipal implements UserDetails {

    private User user;

    public UserPrincipal(User user) {
        super();
        this.user = user;
    }

    @Override
    public String getUsername() {
        return user.getUsername();
    }

    @Override
    public String getPassword() {
        return user.getPassword();
    }

    @Override
    public Collection<? extends GrantedAuthority> getAuthorities() {
        return Arrays.asList(new SimpleGrantedAuthority("ROLE_USER"));
    }

    @Override
    public boolean isAccountNonExpired() {
        return true;
    }

    @Override
    public boolean isAccountNonLocked() {
        return true;
    }

    @Override
    public boolean isCredentialsNonExpired() {
        return true;
    }

    @Override
    public boolean isEnabled() {
        return true;
    }
}
```

注意:我们使用我们的自定义`Principal` " `UserPrincipal”`而不是 Spring 安全默认`User`。

## 5。认证 Reddit

既然我们的认证流程不再依赖 Reddit，我们需要**让用户在登录后将他们的账户连接到 Reddit** 。

首先，我们需要修改旧的 Reddit 登录逻辑:

```java
@RequestMapping("/redditLogin")
public String redditLogin() {
    OAuth2AccessToken token = redditTemplate.getAccessToken();
    service.connectReddit(redditTemplate.needsCaptcha(), token);
    return "redirect:home";
}
```

以及实际的实现——`connectReddit()`方法:

```java
@Override
public void connectReddit(boolean needsCaptcha, OAuth2AccessToken token) {
    UserPrincipal userPrincipal = (UserPrincipal) 
      SecurityContextHolder.getContext().getAuthentication().getPrincipal();
    User currentUser = userPrincipal.getUser();
    currentUser.setNeedCaptcha(needsCaptcha);
    currentUser.setAccessToken(token.getValue());
    currentUser.setRefreshToken(token.getRefreshToken().getValue());
    currentUser.setTokenExpiration(token.getExpiration());
    userRepository.save(currentUser);
}
```

注意现在如何使用`redditLogin()`逻辑通过获取用户的`AccessToken`将我们系统中的用户帐户与他的 Reddit 帐户连接起来。

至于前端——这很简单:

```java
<h1>Welcome, 
<a href="profile" sec:authentication="principal.username">Bob</a></small>
</h1>
<a th:if="${#authentication.principal.user.accessToken == null}" href="redditLogin" >
    Connect your Account to Reddit
</a>
```

我们还需要确保用户在尝试提交帖子之前确实将他们的帐户连接到 Reddit:

```java
@RequestMapping("/post")
public String showSubmissionForm(Model model) {
    if (getCurrentUser().getAccessToken() == null) {
        model.addAttribute("msg", "Sorry, You did not connect your account to Reddit yet");
        return "submissionResponse";
    }
    ...
}
```

## 6。结论

小小的 reddit 应用肯定是在前进的。

旧的认证流程——完全由 Reddit 支持——引发了一些问题。所以现在，**我们有了一个干净简单的基于表单的登录**，同时仍然能够在后端连接你的 Reddit API。

好东西。