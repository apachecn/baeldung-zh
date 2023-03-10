# Spring Security:升级已弃用的 WebSecurityConfigurerAdapter

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-deprecated-websecurityconfigureradapter>

## 1.概观

Spring Security 允许通过扩展一个`WebSecurityConfigurerAdapter`类为端点授权或身份验证管理器配置等特性定制 HTTP 安全性。然而，从最近的版本开始，Spring 不再使用这种方法，而是鼓励基于组件的安全配置。

在本教程中，我们将看到一个例子，如何在 Spring Boot 应用程序中替换这个弃用，并运行一些 MVC 测试。

## 2.春天的安全没有了`WebSecurityConfigurerAdapter`

**我们通常会看到扩展了`WebSecurityConfigureAdapter`类的 Spring [HTTP 安全配置](/web/20220916120910/https://www.baeldung.com/java-config-spring-security)类。**

然而，从版本 5.7.0-M2 开始，Spring 反对使用`WebSecurityConfigureAdapter`并建议创建没有的[配置。](https://web.archive.org/web/20220916120910/https://spring.io/blog/2022/02/21/spring-security-without-the-websecurityconfigureradapter)

我们将创建一个使用内存认证的示例 Spring Boot 应用程序来展示这种新的配置类型。

首先，让我们定义我们的配置类:

```java
@EnableWebSecurity
@EnableGlobalMethodSecurity(prePostEnabled = true, securedEnabled = true, jsr250Enabled = true)
public class SecurityConfig {

    // config

}
```

我们正在添加[方法安全性](/web/20220916120910/https://www.baeldung.com/spring-security-method-security)注释，以支持基于不同角色的处理。

### 2.1.配置身份验证

对于`WebSecurityConfigureAdapter,` ，我们使用一个`AuthenticationManagerBuilder`来设置我们的认证上下文。

**现在，如果我们想避免被弃用，我们可以定义一个`UserDetailsManager`或`UserDetailsService` 组件:**

```java
@Bean
public UserDetailsService userDetailsService(BCryptPasswordEncoder bCryptPasswordEncoder) {
    InMemoryUserDetailsManager manager = new InMemoryUserDetailsManager();
    manager.createUser(User.withUsername("user")
      .password(bCryptPasswordEncoder.encode("userPass"))
      .roles("USER")
      .build());
    manager.createUser(User.withUsername("admin")
      .password(bCryptPasswordEncoder.encode("adminPass"))
      .roles("USER", "ADMIN")
      .build());
    return manager;
}
```

或者，给定我们的`UserDetailService`，我们甚至可以设置一个`AuthenticationManager`:

```java
@Bean
public AuthenticationManager authManager(HttpSecurity http, BCryptPasswordEncoder bCryptPasswordEncoder, UserDetailService userDetailService) 
  throws Exception {
    return http.getSharedObject(AuthenticationManagerBuilder.class)
      .userDetailsService(userDetailsService)
      .passwordEncoder(bCryptPasswordEncoder)
      .and()
      .build();
} 
```

类似地，如果我们使用 JDBC 或 LDAP 身份验证，这也可以工作。

### 2.2.配置 HTTP 安全性

**更重要的是，如果我们想避免对 HTTP 安全性的反对，我们现在可以创建一个`SecurityFilterChain` bean。**

例如，假设我们想要保护依赖于角色的端点，并只为登录留下一个匿名入口点。我们还将任何删除请求限制为管理员角色。我们将使用[基本认证:](/web/20220916120910/https://www.baeldung.com/spring-security-basic-authentication)

```java
@Bean
public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
    http.csrf()
      .disable()
      .authorizeRequests()
      .antMatchers(HttpMethod.DELETE)
      .hasRole("ADMIN")
      .antMatchers("/admin/**")
      .hasAnyRole("ADMIN")
      .antMatchers("/user/**")
      .hasAnyRole("USER", "ADMIN")
      .antMatchers("/login/**")
      .anonymous()
      .anyRequest()
      .authenticated()
      .and()
      .httpBasic()
      .and()
      .sessionManagement()
      .sessionCreationPolicy(SessionCreationPolicy.STATELESS);

    return http.build();
} 
```

**HTTP 安全将构建一个`DefaultSecurityFilterChain`对象来加载请求匹配器和过滤器。**

### 2.3.配置 Web 安全性

同样，为了 Web 安全，我们现在可以使用回调接口`WebSecurityCustomizer.`

让我们添加一个调试级别，忽略一些路径，如图像或脚本:

```java
@Bean
public WebSecurityCustomizer webSecurityCustomizer() {
    return (web) -> web.debug(securityDebug)
      .ignoring()
      .antMatchers("/css/**", "/js/**", "/img/**", "/lib/**", "/favicon.ico");
}
```

## 3.端点控制器

让我们为我们的应用程序定义一个简单的 REST 控制器类:

```java
@RestController
public class ResourceController {
    @GetMapping("/login")
    public String loginEndpoint() {
        return "Login!";
    }

    @GetMapping("/admin")
    public String adminEndpoint() {
        return "Admin!";
    }

    @GetMapping("/user")
    public String userEndpoint() {
        return "User!";
    }

    @GetMapping("/all")
    public String allRolesEndpoint() {
        return "All Roles!";
    }

    @DeleteMapping("/delete")
    public String deleteEndpoint(@RequestBody String s) {
        return "I am deleting " + s;
    }
}
```

正如我们之前在定义 HTTP 安全性时提到的，我们将添加一个任何人都可以访问的通用`/login`端点、管理员和用户的特定端点，以及一个不受角色保护但仍然需要身份验证的`/all`端点。

## 4.测试终点

让我们使用 MVC 模拟来测试我们的端点，将我们的新配置添加到 [Spring Boot 测试](/web/20220916120910/https://www.baeldung.com/spring-boot-testing)中。

### 4.1.测试匿名用户

匿名用户可以访问`/login`端点。如果他们试图访问其他东西，他们将是未经授权的(`401`):

```java
@Test
@WithAnonymousUser
public void whenAnonymousAccessLogin_thenOk() throws Exception {
    mvc.perform(get("/login"))
      .andExpect(status().isOk());
}

@Test
@WithAnonymousUser
public void whenAnonymousAccessRestrictedEndpoint_thenIsUnauthorized() throws Exception {
    mvc.perform(get("/all"))
      .andExpect(status().isUnauthorized());
} 
```

此外，对于除了`/login`之外的所有端点，我们总是需要认证，就像对于`/all`端点一样。

### 4.2.测试用户角色

用户角色可以访问通用端点和我们授予该角色的所有其他路径:

```java
@Test
@WithUserDetails()
public void whenUserAccessUserSecuredEndpoint_thenOk() throws Exception {
    mvc.perform(get("/user"))
      .andExpect(status().isOk());
}

@Test
@WithUserDetails()
public void whenUserAccessRestrictedEndpoint_thenOk() throws Exception {
    mvc.perform(get("/all"))
      .andExpect(status().isOk());
}

@Test
@WithUserDetails()
public void whenUserAccessAdminSecuredEndpoint_thenIsForbidden() throws Exception {
    mvc.perform(get("/admin"))
      .andExpect(status().isForbidden());
}

@Test
@WithUserDetails()
public void whenUserAccessDeleteSecuredEndpoint_thenIsForbidden() throws Exception {
    mvc.perform(delete("/delete"))
      .andExpect(status().isForbidden());
} 
```

值得注意的是，如果用户角色试图访问受管理保护的端点，用户会得到一个“禁止”(`403`)错误。

相反，没有凭证的人，比如前面例子中的匿名用户，将得到一个“未授权”的错误(`401`)。

### 4.3.测试管理员角色

如我们所见，具有管理员角色的人可以访问任何端点:

```java
@Test
@WithUserDetails(value = "admin")
public void whenAdminAccessUserEndpoint_thenOk() throws Exception {
    mvc.perform(get("/user"))
      .andExpect(status().isOk());
}

@Test
@WithUserDetails(value = "admin")
public void whenAdminAccessAdminSecuredEndpoint_thenIsOk() throws Exception {
    mvc.perform(get("/admin"))
      .andExpect(status().isOk());
}

@Test
@WithUserDetails(value = "admin")
public void whenAdminAccessDeleteSecuredEndpoint_thenIsOk() throws Exception {
    mvc.perform(delete("/delete").content("{}"))
      .andExpect(status().isOk());
} 
```

## 5.结论

在本文中，我们看到了如何在不使用`WebSecurityConfigureAdapter`的情况下创建 Spring 安全配置，并在创建用于身份验证、HTTP 安全和 Web 安全的组件时替换它。

一如既往，我们可以在 GitHub 上找到工作代码示例[。](https://web.archive.org/web/20220916120910/https://github.com/eugenp/tutorials/tree/master/spring-security-modules/spring-security-web-boot-4)