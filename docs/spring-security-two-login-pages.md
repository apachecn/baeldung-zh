# 使用 Spring Security 的两个登录页面

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-security-two-login-pages>

## 1。简介

在本教程中，我们将看到如何使用配置中的两个不同的 Spring Security `http`元素来配置 Spring Security，使其与两个不同的登录页面协同工作。

## 2。配置 2 个 Http 元素

我们可能需要两个登录页面的情况之一是，一个页面用于应用程序的管理员，另一个页面用于普通用户。

我们将**配置两个`http`元素**，这两个元素将通过与之相关联的 URL 模式来区分:

*   **/user*** 对于需要普通用户认证才能访问的页面
*   **/admin*** 对于将由管理员访问的页面

每个`http`元素将有不同的登录页面和不同的登录处理 URL。

为了配置两个不同的`http`元素，让我们创建两个用`@Configuration`标注的静态类。

两者都将被放在一个常规的`@Configuration`类中:

```
@Configuration
@EnableWebSecurity
public class SecurityConfig {
    ...
}
```

让我们为`“ADMIN”`用户定义`ConfigurerAdapter`:

```
@Configuration
@Order(1)
public static class App1ConfigurationAdapter {

    @Bean
    public SecurityFilterChain filterChainApp1(HttpSecurity http) throws Exception {
        http.antMatcher("/admin*")
          .authorizeRequests()
          .anyRequest()
          .hasRole("ADMIN")

          .and()
          .formLogin()
          .loginPage("/loginAdmin")
          .loginProcessingUrl("/admin_login")
          .failureUrl("/loginAdmin?error=loginError")
          .defaultSuccessUrl("/adminPage")

          .and()
          .logout()
          .logoutUrl("/admin_logout")
          .logoutSuccessUrl("/protectedLinks")
          .deleteCookies("JSESSIONID")

          .and()
          .exceptionHandling()
          .accessDeniedPage("/403")

          .and()
          .csrf().disable();
       return http.build();
    }
}
```

现在，让我们为普通用户定义`ConfigurerAdapter`:

```
@Configuration
@Order(2)
public static class App2ConfigurationAdapter {

    @Bean 
    public SecurityFilterChain filterChainApp2(HttpSecurity http) throws Exception {
        http.antMatcher("/user*")
          .authorizeRequests()
          .anyRequest()
          .hasRole("USER")

          .and()
          .formLogin()
          .loginPage("/loginUser")
          .loginProcessingUrl("/user_login")
          .failureUrl("/loginUser?error=loginError")
          .defaultSuccessUrl("/userPage")

          .and()
          .logout()
          .logoutUrl("/user_logout")
          .logoutSuccessUrl("/protectedLinks")
          .deleteCookies("JSESSIONID")

          .and()
          .exceptionHandling()
          .accessDeniedPage("/403")

          .and()
          .csrf().disable();
       return http.build();
    }
}
```

注意，通过在每个静态类上放置`@Order`注释，我们指定了当请求 URL 时，基于模式匹配考虑这两个类的顺序。

**两个配置类不能有相同的顺序。**

## 3。自定义登录页面

我们将为每种类型的用户创建自己的自定义登录页面。对于管理员用户，登录表单将有一个`“user_login”`动作，如配置中所定义:

```
<p>User login page</p>
<form name="f" action="user_login" method="POST">
    <table>
        <tr>
            <td>User:</td>
            <td><input type="text" name="username" value=""></td>
        </tr>
        <tr>
            <td>Password:</td>
            <td><input type="password" name="password" /></td>
        </tr>
        <tr>
            <td><input name="submit" type="submit" value="submit" /></td>
        </tr>
    </table>
</form>
```

管理员登录页面与此类似，只是根据 java 配置，表单将有一个动作`“admin_login”`。

## 4。认证配置

现在我们需要**为我们的应用程序**配置认证。让我们来看看实现这一点的两种方法——一种使用一个公共源进行用户身份验证，另一种使用两个单独的源。

### 4.1。使用公共用户认证源

如果两个登录页面共享一个认证用户的公共源，那么您可以创建一个类型为`UserDetailsService`的 bean 来处理认证。

让我们用一个定义了两个用户的`InMemoryUserDetailsManager`来演示这个场景——一个角色是`“USER”`,另一个角色是`“ADMIN”`:

```
@Bean
public UserDetailsService userDetailsService() throws Exception {
    InMemoryUserDetailsManager manager = new InMemoryUserDetailsManager();
    manager.createUser(User
      .withUsername("user")
      .password(encoder().encode("userPass"))
      .roles("USER")
      .build());

    manager.createUser(User
      .withUsername("admin")
      .password(encoder().encode("adminPass"))
      .roles("ADMIN")
      .build());

    return manager;
}

@Bean
public static PasswordEncoder encoder() {
    return new BCryptPasswordEncoder();
}
```

### 4.2。使用两种不同的用户认证来源

如果您有不同的用户认证来源——一个用于管理员，一个用于普通用户——您可以在每个静态`@Configuration`类中配置一个`AuthenticationManagerBuilder`。让我们看一个针对`“ADMIN”`用户的认证管理器的例子:

```
@Configuration
@Order(1)
public static class App1ConfigurationAdapter {

    @Bean
    public UserDetailsService userDetailsServiceApp1() {
         UserDetails user = User.withUsername("admin")
             .password(encoder().encode("admin"))
             .roles("ADMIN")
             .build();
         return new InMemoryUserDetailsManager(user);
    }
}
```

在这种情况下，前面部分的`UserDetailsService` bean 将不再被使用。

## 6。结论

在这个快速教程中，我们展示了如何在同一个 Spring 安全应用程序中实现两个不同的登录页面。

本文的完整代码可以在 [GitHub 项目](https://web.archive.org/web/20221024122636/https://github.com/eugenp/tutorials/tree/master/spring-security-modules/spring-security-web-boot-2)中找到。

当您运行应用程序时，您可以在`/protectedLinks` URI 上访问上面的示例。