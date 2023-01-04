# 春季安全 vs 阿帕奇·希罗

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-security-vs-apache-shiro>

## 1.概观

安全性是应用程序开发领域中的主要问题，尤其是在企业 web 和移动应用程序领域。

在这个快速教程中，**我们将比较两个流行的 Java 安全框架——[阿帕奇·希罗](https://web.archive.org/web/20220628145159/https://shiro.apache.org/)和 [Spring 安全](https://web.archive.org/web/20220628145159/https://spring.io/projects/spring-security)** 。

## 2.一点背景

阿帕奇·希罗于 2004 年出生，取名为 JSecurity，2008 年被阿帕奇基金会接受。到目前为止，它已经发布了很多版本，最新的版本是 1.5.3。

Spring Security 始于 2003 年的 Acegi，并在 2008 年首次公开发布时被纳入 Spring 框架。从一开始，它已经经历了几次迭代，目前的 GA 版本是 5.3.2。

这两种技术都提供了**认证和授权支持，以及加密和会话管理解决方案**。此外，Spring Security 针对诸如 CSRF 和会话固定等攻击提供了一流的保护。

在接下来的几节中，我们将看到这两种技术如何处理认证和授权的例子。为了简单起见，我们将使用带有 [FreeMarker 模板](/web/20220628145159/https://www.baeldung.com/spring-template-engines#4-freemarker-in-spring-boot)的基本 using basic 应用程序。

## 3.配置阿帕奇·希罗

首先，让我们看看这两个框架之间的配置有何不同。

### 3.1.Maven 依赖性

因为我们将在 Spring Boot 应用程序中使用 Shiro，所以我们需要它的启动器和`shiro-core`模块:

```
<dependency>
    <groupId>org.apache.shiro</groupId>
    <artifactId>shiro-spring-boot-web-starter</artifactId>
    <version>1.5.3</version>
</dependency>
<dependency>
    <groupId>org.apache.shiro</groupId>
    <artifactId>shiro-core</artifactId>
    <version>1.5.3</version>
</dependency>
```

最新版本可以在 [Maven Central](https://web.archive.org/web/20220628145159/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22org.apache.shiro%22) 上找到。

### 3.2.创建一个领域

为了在内存中声明用户及其角色和权限，我们需要创建一个扩展 Shiro 的`JdbcRealm`的领域。我们将定义两个用户–Tom 和 Jerry，角色分别为 USER 和 ADMIN:

```
public class CustomRealm extends JdbcRealm {

    private Map<String, String> credentials = new HashMap<>();
    private Map<String, Set> roles = new HashMap<>();
    private Map<String, Set> permissions = new HashMap<>();

    {
        credentials.put("Tom", "password");
        credentials.put("Jerry", "password");

        roles.put("Jerry", new HashSet<>(Arrays.asList("ADMIN")));
        roles.put("Tom", new HashSet<>(Arrays.asList("USER")));

        permissions.put("ADMIN", new HashSet<>(Arrays.asList("READ", "WRITE")));
        permissions.put("USER", new HashSet<>(Arrays.asList("READ")));
    }
}
```

接下来，为了能够检索这个身份验证和授权，我们需要覆盖一些方法:

```
@Override
protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken token) 
  throws AuthenticationException {
    UsernamePasswordToken userToken = (UsernamePasswordToken) token;

    if (userToken.getUsername() == null || userToken.getUsername().isEmpty() ||
      !credentials.containsKey(userToken.getUsername())) {
        throw new UnknownAccountException("User doesn't exist");
    }
    return new SimpleAuthenticationInfo(userToken.getUsername(), 
      credentials.get(userToken.getUsername()), getName());
}

@Override
protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principals) {
    Set roles = new HashSet<>();
    Set permissions = new HashSet<>();

    for (Object user : principals) {
        try {
            roles.addAll(getRoleNamesForUser(null, (String) user));
            permissions.addAll(getPermissions(null, null, roles));
        } catch (SQLException e) {
            logger.error(e.getMessage());
        }
    }
    SimpleAuthorizationInfo authInfo = new SimpleAuthorizationInfo(roles);
    authInfo.setStringPermissions(permissions);
    return authInfo;
} 
```

方法`doGetAuthorizationInfo`使用几个助手方法来获取用户的角色和权限:

```
@Override
protected Set getRoleNamesForUser(Connection conn, String username) 
  throws SQLException {
    if (!roles.containsKey(username)) {
        throw new SQLException("User doesn't exist");
    }
    return roles.get(username);
}

@Override
protected Set getPermissions(Connection conn, String username, Collection roles) 
  throws SQLException {
    Set userPermissions = new HashSet<>();
    for (String role : roles) {
        if (!permissions.containsKey(role)) {
            throw new SQLException("Role doesn't exist");
        }
        userPermissions.addAll(permissions.get(role));
    }
    return userPermissions;
} 
```

接下来，我们需要将这个`CustomRealm`作为 bean 包含在我们的引导应用程序中:

```
@Bean
public Realm customRealm() {
    return new CustomRealm();
}
```

此外，要为我们的端点配置身份验证，我们需要另一个 bean:

```
@Bean
public ShiroFilterChainDefinition shiroFilterChainDefinition() {
    DefaultShiroFilterChainDefinition filter = new DefaultShiroFilterChainDefinition();

    filter.addPathDefinition("/home", "authc");
    filter.addPathDefinition("/**", "anon");
    return filter;
}
```

在这里，使用一个`DefaultShiroFilterChainDefinition`实例，我们指定我们的`/home`端点只能被经过身份验证的用户访问。

这就是我们进行配置所需的全部内容，Shiro 会为我们完成剩下的工作。

## 4.配置 Spring 安全性

现在让我们看看如何在春天达到同样的效果。

### 4.1.Maven 依赖性

首先，依赖性:

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>

<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>
```

最新版本可以在 [Maven Central](https://web.archive.org/web/20220628145159/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22org.springframework.boot%22%20AND%20a%3A%22spring-boot-starter%22) 上找到。

### 4.2.配置类

接下来，我们将在类`SecurityConfig`中定义我们的 Spring 安全配置，扩展`WebSecurityConfigurerAdapter`:

```
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http
          .authorizeRequests(authorize -> authorize
            .antMatchers("/index", "/login").permitAll()
            .antMatchers("/home", "/logout").authenticated()
            .antMatchers("/admin/**").hasRole("ADMIN"))
          .formLogin(formLogin -> formLogin
            .loginPage("/login")
            .failureUrl("/login-error"));
    }

    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        auth.inMemoryAuthentication()
          .withUser("Jerry")
            .password(passwordEncoder().encode("password"))
            .authorities("READ", "WRITE")
            .roles("ADMIN")
            .and()
          .withUser("Tom")
            .password(passwordEncoder().encode("password"))
            .authorities("READ")
            .roles("USER");
    }

    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }
} 
```

正如我们所看到的，我们构建了一个`AuthenticationManagerBuilder`对象来声明我们的用户及其角色和权限。此外，我们使用一个`BCryptPasswordEncoder`对密码进行编码。

Spring Security 还为我们提供了它的`HttpSecurity`对象，用于进一步的配置。在我们的示例中，我们允许:

*   每个人都可以访问我们的`index`和`login`页面
*   只有经过认证的用户才能进入`home`页面和`logout`
*   只有具有管理员角色的用户才能访问`admin`页面

我们还定义了对基于表单的认证的支持，以将用户发送到`login`端点。如果登录失败，我们的用户将被重定向到`/login-error`。

## 5.控制器和端点

现在让我们来看看这两个应用程序的 web 控制器映射。虽然它们将使用相同的端点，但一些实现会有所不同。

### 5.1.视图渲染的端点

对于呈现视图的端点，实现是相同的:

```
@GetMapping("/")
public String index() {
    return "index";
}

@GetMapping("/login")
public String showLoginPage() {
    return "login";
}

@GetMapping("/home")
public String getMeHome(Model model) {
    addUserAttributes(model);
    return "home";
}
```

我们的控制器实现 Shiro 和 Spring Security 都在根端点返回`index.ftl`，在登录端点返回`login.ftl`，在本地端点返回`home.ftl`。

然而，在两个控制器之间，`/home`端点处的方法`addUserAttributes`的定义会有所不同。该方法检查当前登录用户的属性。

Shiro 提供了一个`SecurityUtils#getSubject`来检索当前的`Subject`，以及它的角色和权限:

```
private void addUserAttributes(Model model) {
    Subject currentUser = SecurityUtils.getSubject();
    String permission = "";

    if (currentUser.hasRole("ADMIN")) {
        model.addAttribute("role", "ADMIN");
    } else if (currentUser.hasRole("USER")) {
        model.addAttribute("role", "USER");
    }
    if (currentUser.isPermitted("READ")) {
        permission = permission + " READ";
    }
    if (currentUser.isPermitted("WRITE")) {
        permission = permission + " WRITE";
    }
    model.addAttribute("username", currentUser.getPrincipal());
    model.addAttribute("permission", permission);
}
```

另一方面，Spring Security 为此目的从其`SecurityContextHolder`的上下文中提供了一个`Authentication`对象:

```
private void addUserAttributes(Model model) {
    Authentication auth = SecurityContextHolder.getContext().getAuthentication();
    if (auth != null && !auth.getClass().equals(AnonymousAuthenticationToken.class)) {
        User user = (User) auth.getPrincipal();
        model.addAttribute("username", user.getUsername());
        Collection<GrantedAuthority> authorities = user.getAuthorities();

        for (GrantedAuthority authority : authorities) {
            if (authority.getAuthority().contains("USER")) {
                model.addAttribute("role", "USER");
                model.addAttribute("permissions", "READ");
            } else if (authority.getAuthority().contains("ADMIN")) {
                model.addAttribute("role", "ADMIN");
                model.addAttribute("permissions", "READ WRITE");
            }
        }
    }
}
```

### 5.2.登录后端点

在 Shiro 中，我们将用户输入的凭证映射到一个 POJO:

```
public class UserCredentials {

    private String username;
    private String password;

    // getters and setters
}
```

然后我们将创建一个`UsernamePasswordToken` 来登录用户，或`Subject`，登录:

```
@PostMapping("/login")
public String doLogin(HttpServletRequest req, UserCredentials credentials, RedirectAttributes attr) {

    Subject subject = SecurityUtils.getSubject();
    if (!subject.isAuthenticated()) {
        UsernamePasswordToken token = new UsernamePasswordToken(credentials.getUsername(),
          credentials.getPassword());
        try {
            subject.login(token);
        } catch (AuthenticationException ae) {
            logger.error(ae.getMessage());
            attr.addFlashAttribute("error", "Invalid Credentials");
            return "redirect:/login";
        }
    }
    return "redirect:/home";
}
```

在 Spring 安全方面，这只是重定向到主页的问题。 **Spring 的登录过程由它的`UsernamePasswordAuthenticationFilter`处理，对我们**是透明的:

```
@PostMapping("/login")
public String doLogin(HttpServletRequest req) {
    return "redirect:/home";
}
```

### 5.3.仅限管理员的端点

现在让我们来看一个场景，我们必须执行基于角色的访问。假设我们有一个`/admin`端点，应该只允许管理员角色访问它。

让我们看看如何在 Shiro 中做到这一点:

```
@GetMapping("/admin")
public String adminOnly(ModelMap modelMap) {
    addUserAttributes(modelMap);
    Subject currentUser = SecurityUtils.getSubject();
    if (currentUser.hasRole("ADMIN")) {
        modelMap.addAttribute("adminContent", "only admin can view this");
    }
    return "home";
}
```

在这里，我们提取当前登录的用户，检查他们是否有管理员角色，并相应地添加内容。

在 Spring Security 中，不需要以编程方式检查角色，我们已经在`SecurityConfig`中定义了谁可以到达这个端点。所以现在，只是添加业务逻辑的问题:

```
@GetMapping("/admin")
public String adminOnly(HttpServletRequest req, Model model) {
    addUserAttributes(model);
    model.addAttribute("adminContent", "only admin can view this");
    return "home";
}
```

### 5.4.注销端点

最后，让我们实现注销端点。

在 Shiro 中，我们将简单地调用`Subject#logout`:

```
@PostMapping("/logout")
public String logout() {
    Subject subject = SecurityUtils.getSubject();
    subject.logout();
    return "redirect:/";
}
```

对于 Spring，我们没有为注销定义任何映射。在这种情况下，它的默认注销机制开始起作用，这是自动应用的，因为我们在配置中扩展了`WebSecurityConfigurerAdapter`。

## 6.阿帕奇·希罗 vs 春天安全

既然我们已经了解了实现的差异，那么让我们来看看其他几个方面。

在社区支持方面， **Spring 框架通常有一个庞大的开发者社区**，他们积极地参与到它的开发和使用中。既然春保是伞的一部分，那它也必须享有同样的优势。Shiro 虽然很受欢迎，却没有如此庞大的支持。

关于文档，Spring 再次胜出。

然而，Spring Security 有一点学习曲线。另一方面，Shiro 很容易理解。对于桌面应用程序，通过`[shiro.ini](https://web.archive.org/web/20220628145159/https://shiro.apache.org/configuration.html#Configuration-CreatingaSecurityManagerfromINI)`进行配置更加容易。

但是，正如我们在示例片段中看到的， **Spring Security 在将业务逻辑和安全性** **分开**方面做得很好，并且真正将安全性作为一个跨领域的问题来提供。

## 7.结论

在本教程中，**我们将阿帕奇·希罗与 Spring Security** 进行了比较。

我们只是触及了这些框架所能提供的皮毛，还有很多东西需要进一步探索。有相当多的选择，比如 JAAS、T2、OACC 和 T3。尽管如此，凭借其优势，[春天证券](/web/20220628145159/https://www.baeldung.com/security-spring)似乎在这一点上胜出。

和往常一样，源代码可以在 GitHub 上找到[。](https://web.archive.org/web/20220628145159/https://github.com/eugenp/tutorials/tree/master/apache-shiro)