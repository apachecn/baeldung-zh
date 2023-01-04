# Spring Security 中的多个入口点

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-security-multiple-entry-points>

 ![announcement - icon](img/0cfa94adfb72ebe01decbc5956dcb9fe.png)

It’s just plain hard to get true, **real-time visibility into a running auth flow.**

Parts of the process can be completely hidden from us; if the complete authorization process requires a redirect from a remote OAuth production server, then every debugging effort must go through the production server.

It’s practically unfeasible to debug this locally. There’s no way to reproduce the exact state and no way to inspect what is actually happening under the hood. Not ideal.

Knowing these types of challenges, we built Lightrun - a real-time production debugging tool - to allow you to understand complicated flows with code-level information. Add logs, take snapshots (virtual breakpoints), and instrument metrics without a remote debugger, without stopping the running service, and, most importantly - **in real-time and without side effects**.

**Learn more with this 5-minute tutorial** focused on debugging these kinds of scenarios using Lightrun:

[>> Debugging Authentication and Authorization Using Lightrun](/web/20220525124030/https://www.baeldung.com/lightrun-n-security)

## 1。概述

在这个快速教程中，我们将看看如何在 Spring 安全应用程序中**定义多个入口点。**

这主要需要在一个 XML 配置文件中定义多个`http`块，或者通过多次扩展`WebSecurityConfigurerAdapter`类来定义多个`HttpSecurity`实例。

## 2。Maven 依赖关系

对于开发，我们将需要以下依赖项:

```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
    <version>2.6.1</version>
</dependency>
<dependency> 
    <groupId>org.springframework.boot</groupId> 
    <artifactId>spring-boot-starter-web</artifactId> 
    <version>2.6.1</version> 
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-thymeleaf</artifactId>
    <version>2.6.1</version>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-test</artifactId>
    <version>2.6.1</version>
</dependency>    
<dependency>
    <groupId>org.springframework.security</groupId>
    <artifactId>spring-security-test</artifactId>
    <version>5.4.0</version>
</dependency>
```

最新版本的[spring-boot-starter-security](https://web.archive.org/web/20220525124030/https://search.maven.org/classic/#search%7Cga%7C1%7Ca%3A%22spring-boot-starter-security%22)、 [spring-boot-starter-web](https://web.archive.org/web/20220525124030/https://search.maven.org/classic/#search%7Cga%7C1%7Ca%3A%22spring-boot-starter-web%22%20AND%20g%3A%22org.springframework.boot%22) 、[spring-boot-starter-百里香叶](https://web.archive.org/web/20220525124030/https://search.maven.org/classic/#search%7Cga%7C1%7Ca%3A%22spring-boot-starter-thymeleaf%22)、 [spring-boot-starter-test](https://web.archive.org/web/20220525124030/https://search.maven.org/classic/#search%7Cga%7C1%7Ca%3A%22spring-boot-starter-test%22) 、 [spring-security-test](https://web.archive.org/web/20220525124030/https://search.maven.org/classic/#search%7Cga%7C1%7Ca%3A%22spring-security-test%22) 可从 Maven Central 下载。

## 3。多个入口点

### 3.1。具有多个 HTTP 元素的多个入口点

让我们定义将保存用户源的主配置类:

```java
@Configuration
@EnableWebSecurity
public class MultipleEntryPointsSecurityConfig {

    @Bean
    public UserDetailsService userDetailsService() throws Exception {
        InMemoryUserDetailsManager manager = new InMemoryUserDetailsManager();
        manager.createUser(User
          .withUsername("user")
          .password(encoder().encode("userPass"))
          .roles("USER").build());
        manager.createUser(User
          .withUsername("admin")
          .password(encoder().encode("adminPass"))
          .roles("ADMIN").build());
        return manager;
    }

    @Bean
    public PasswordEncoder encoder() {
        return new BCryptPasswordEncoder();
    }
}
```

现在，让我们看看如何在我们的安全配置中定义多个入口点。

我们将在这里使用一个由基本认证驱动的例子，我们将充分利用这样一个事实，即 **Spring Security 在我们的配置中支持多个 HTTP 元素**的定义。

当使用 Java 配置时，定义多个安全领域的方法是拥有多个扩展了`WebSecurityConfigurerAdapter`基类的`@Configuration`类——每个类都有自己的安全配置。这些类可以是静态的，放在主配置中。

在一个应用程序中拥有多个入口点的主要动机是，如果有不同类型的用户可以访问应用程序的不同部分。

让我们定义一个具有三个入口点的配置，每个入口点具有不同的权限和身份验证模式:

*   一个用于使用 HTTP 基本身份验证的管理用户
*   一个用于使用表单身份验证的普通用户
*   另一个用于不需要身份验证的访客用户

为管理用户定义的入口点保护形式为`/admin/**`的 URL，只允许具有管理员角色的用户，并要求使用使用`authenticationEntryPoint()`方法设置的类型为`BasicAuthenticationEntryPoint` 的入口点进行 HTTP 基本身份验证:

```java
@Configuration
@Order(1)
public static class App1ConfigurationAdapter extends WebSecurityConfigurerAdapter {

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.antMatcher("/admin/**")
            .authorizeRequests().anyRequest().hasRole("ADMIN")
            .and().httpBasic().authenticationEntryPoint(authenticationEntryPoint());
    }

    @Bean
    public AuthenticationEntryPoint authenticationEntryPoint(){
        BasicAuthenticationEntryPoint entryPoint = 
          new BasicAuthenticationEntryPoint();
        entryPoint.setRealmName("admin realm");
        return entryPoint;
    }
}
```

每个静态类上的 `@Order`注释指出了配置被考虑的顺序，以找到一个与请求的 URL 匹配的配置。**每个类的`order`值必须唯一。**

类型为`BasicAuthenticationEntryPoint`的 bean 需要设置属性`realName`。

### 3.2。多个入口点，相同的 HTTP 元素

接下来，让我们定义表单 URL 的配置，具有用户角色的普通用户可以使用表单身份验证来访问表单 URL:

```java
@Configuration
@Order(2)
public static class App2ConfigurationAdapter extends WebSecurityConfigurerAdapter {

    protected void configure(HttpSecurity http) throws Exception {
        http.antMatcher("/user/**")
            .authorizeRequests().anyRequest().hasRole("USER")
            .and()
            // formLogin configuration
            .and()
            .exceptionHandling()
            .defaultAuthenticationEntryPointFor(
              loginUrlauthenticationEntryPointWithWarning(),
              new AntPathRequestMatcher("/user/private/**"))
            .defaultAuthenticationEntryPointFor(
              loginUrlauthenticationEntryPoint(), 
              new AntPathRequestMatcher("/user/general/**"));
    }
}
```

正如我们所见，除了 authenticationEntryPoint()方法之外，另一种定义入口点的方法是使用`defaultAuthenticationEntryPointFor()`方法。这可以基于一个`RequestMatcher`对象定义多个匹配不同条件的入口点。

`RequestMatcher`接口具有基于不同条件类型的实现，比如匹配路径、媒体类型或正则表达式。在我们的例子中，我们使用了 AntPathRequestMatch 为表单`/user/private/**`和`/user/general/**`的 URL 设置了两个不同的入口点。

接下来，我们需要在同一个静态配置类中定义入口点 beans:

```java
@Bean
public AuthenticationEntryPoint loginUrlauthenticationEntryPoint(){
    return new LoginUrlAuthenticationEntryPoint("/userLogin");
}

@Bean
public AuthenticationEntryPoint loginUrlauthenticationEntryPointWithWarning(){
    return new LoginUrlAuthenticationEntryPoint("/userLoginWithWarning");
}
```

这里的要点是如何设置这些多个入口点——不一定是每个入口点的实现细节。

在这种情况下，入口点都是类型`LoginUrlAuthenticationEntryPoint`，并使用不同的登录页面 URL: `/userLogin`用于简单的登录页面，而`/userLoginWithWarning`用于在试图访问`/user/`私有 URL 时也显示警告的登录页面。

这个配置还需要定义 `/userLogin`和`/userLoginWithWarning` MVC 映射，以及带有标准登录表单的两个页面。

对于表单认证，非常重要的是要记住配置所必需的任何 URL，比如登录处理 URL 也需要遵循 `/user/**`格式，或者被配置为可访问的。

如果没有适当角色的用户试图访问受保护的 URL，上述两种配置都将重定向到`/403` URL。

**注意为 beans 使用唯一的名称，即使它们在不同的静态类**中，否则一个会覆盖另一个。

### 3.3。新的 HTTP 元素，没有入口点

最后，让我们为形式为`/guest/**`的 URL 定义第三种配置，它将允许所有类型的用户，包括未经身份验证的用户:

```java
@Configuration
@Order(3)
public static class App3ConfigurationAdapter extends WebSecurityConfigurerAdapter {

    protected void configure(HttpSecurity http) throws Exception {
        http.antMatcher("/guest/**").authorizeRequests().anyRequest().permitAll();
    }
}
```

### 3.4。XML 配置

让我们看看上一节中三个 *HttpSecurity* 实例的等效 XML 配置。

正如所料，这将包含三个独立的 XML `<http>`块。

对于`/admin/**`URL，XML 配置将使用`http-basic`元素的`entry-point-ref`属性:

```java
<security:http pattern="/admin/**" use-expressions="true" auto-config="true">
    <security:intercept-url pattern="/**" access="hasRole('ROLE_ADMIN')"/>
    <security:http-basic entry-point-ref="authenticationEntryPoint" />
</security:http>

<bean id="authenticationEntryPoint"
  class="org.springframework.security.web.authentication.www.BasicAuthenticationEntryPoint">
     <property name="realmName" value="admin realm" />
</bean>
```

这里需要注意的是，如果使用 XML 配置，角色的形式必须是`ROLE_<ROLE_NAME>`。

在 xml 中，`/user/**`URL 的配置必须分成两个`http`块，因为没有与`defaultAuthenticationEntryPointFor()`方法直接等价的方法。

URL/user/general/* *的配置是:

```java
<security:http pattern="/user/general/**" use-expressions="true" auto-config="true"
  entry-point-ref="loginUrlAuthenticationEntryPoint">
    <security:intercept-url pattern="/**" access="hasRole('ROLE_USER')" />
    //form-login configuration      
</security:http>

<bean id="loginUrlAuthenticationEntryPoint"
  class="org.springframework.security.web.authentication.LoginUrlAuthenticationEntryPoint">
  <constructor-arg name="loginFormUrl" value="/userLogin" />
</bean>
```

对于`/user/private/**`URL，我们可以定义一个类似的配置:

```java
<security:http pattern="/user/private/**" use-expressions="true" auto-config="true"
  entry-point-ref="loginUrlAuthenticationEntryPointWithWarning">
    <security:intercept-url pattern="/**" access="hasRole('ROLE_USER')"/>
    //form-login configuration
</security:http>

<bean id="loginUrlAuthenticationEntryPointWithWarning"
  class="org.springframework.security.web.authentication.LoginUrlAuthenticationEntryPoint">
    <constructor-arg name="loginFormUrl" value="/userLoginWithWarning" />
</bean>
```

对于`/guest/**`URL，我们将拥有`http`元素:

```java
<security:http pattern="/**" use-expressions="true" auto-config="true">
    <security:intercept-url pattern="/guest/**" access="permitAll()"/>  
</security:http>
```

同样重要的是，至少有一个 XML `<http>`块必须匹配/**模式。

## 4。访问受保护的网址

### 4.1。MVC 配置

让我们创建与我们保护的 URL 模式相匹配的请求映射:

```java
@Controller
public class PagesController {

    @GetMapping("/admin/myAdminPage")
    public String getAdminPage() {
        return "multipleHttpElems/myAdminPage";
    }

    @GetMapping("/user/general/myUserPage")
    public String getUserPage() {
        return "multipleHttpElems/myUserPage";
    }

    @GetMapping("/user/private/myPrivateUserPage")
    public String getPrivateUserPage() {
        return "multipleHttpElems/myPrivateUserPage"; 
    }

    @GetMapping("/guest/myGuestPage")
    public String getGuestPage() {
        return "multipleHttpElems/myGuestPage";
    }

    @GetMapping("/multipleHttpLinks")
    public String getMultipleHttpLinksPage() {
        return "multipleHttpElems/multipleHttpLinks";
    }
}
```

`/multipleHttpLinks`映射将返回一个简单的 HTML 页面，其中包含受保护 URL 的链接:

```java
<a th:href="@{/admin/myAdminPage}">Admin page</a>
<a th:href="@{/user/general/myUserPage}">User page</a>
<a th:href="@{/user/private/myPrivateUserPage}">Private user page</a>
<a th:href="@{/guest/myGuestPage}">Guest page</a>
```

对应于受保护的 URL 的每个 HTML 页面将具有简单的文本和反向链接:

```java
Welcome admin!

<a th:href="@{/multipleHttpLinks}" >Back to links</a>
```

### 4.2。初始化应用程序

我们将把我们的例子作为一个 Spring Boot 应用程序运行，所以让我们用 main 方法定义一个类:

```java
@SpringBootApplication
public class MultipleEntryPointsApplication {
    public static void main(String[] args) {
        SpringApplication.run(MultipleEntryPointsApplication.class, args);
    }
}
```

如果我们想使用 XML 配置，我们还需要向我们的主类添加`@ImportResource({“classpath*:spring-security-multiple-entry.xml”})`注释。

### 4.3。测试安全配置

让我们设置一个 JUnit 测试类，我们可以用它来测试我们受保护的 URL:

```java
@RunWith(SpringRunner.class)
@WebAppConfiguration
@SpringBootTest(classes = MultipleEntryPointsApplication.class)
public class MultipleEntryPointsTest {

    @Autowired
    private WebApplicationContext wac;

    @Autowired
    private FilterChainProxy springSecurityFilterChain;

    private MockMvc mockMvc;

    @Before
    public void setup() {
        this.mockMvc = MockMvcBuilders.webAppContextSetup(this.wac)
          .addFilter(springSecurityFilterChain).build();
    }
}
```

接下来，让我们使用`admin`用户测试 URL。

在没有 HTTP 基本认证的情况下请求`/admin/adminPage` URL 时，我们应该会收到一个未授权的状态代码，在添加认证后，状态代码应该是 200 OK。

如果试图用管理员用户访问`/user/userPage` URL，我们应该收到状态 302 禁止:

```java
@Test
public void whenTestAdminCredentials_thenOk() throws Exception {
    mockMvc.perform(get("/admin/myAdminPage")).andExpect(status().isUnauthorized());

    mockMvc.perform(get("/admin/myAdminPage")
      .with(httpBasic("admin", "adminPass"))).andExpect(status().isOk());

    mockMvc.perform(get("/user/myUserPage")
      .with(user("admin").password("adminPass").roles("ADMIN")))
      .andExpect(status().isForbidden());
}
```

让我们使用常规用户凭证创建一个类似的测试来访问 URL:

```java
@Test
public void whenTestUserCredentials_thenOk() throws Exception {
    mockMvc.perform(get("/user/general/myUserPage")).andExpect(status().isFound());

    mockMvc.perform(get("/user/general/myUserPage")
      .with(user("user").password("userPass").roles("USER")))
      .andExpect(status().isOk());

    mockMvc.perform(get("/admin/myAdminPage")
      .with(user("user").password("userPass").roles("USER")))
      .andExpect(status().isForbidden());
}
```

在第二个测试中，我们可以看到，缺少表单身份验证将导致状态 302 Found 而不是 authorized，因为 Spring Security 将重定向到登录表单。

最后，让我们创建一个测试，在该测试中，我们访问`/guest/guestPage` URL，将进行所有三种类型的身份验证，并验证我们是否收到状态 200 OK:

```java
@Test
public void givenAnyUser_whenGetGuestPage_thenOk() throws Exception {
    mockMvc.perform(get("/guest/myGuestPage")).andExpect(status().isOk());

    mockMvc.perform(get("/guest/myGuestPage")
      .with(user("user").password("userPass").roles("USER")))
      .andExpect(status().isOk());

    mockMvc.perform(get("/guest/myGuestPage")
      .with(httpBasic("admin", "adminPass")))
      .andExpect(status().isOk());
}
```

## 5。结论

在本教程中，我们演示了如何在使用 Spring Security 时配置多个入口点。

示例的完整源代码可以在 GitHub 上找到[。要运行应用程序，取消对`pom.xml`中的`MultipleEntryPointsApplication` `start-class`标签的注释，运行命令`mvn spring-boot:run`，然后访问 `/multipleHttpLinks` URL `.`](https://web.archive.org/web/20220525124030/https://github.com/eugenp/tutorials/tree/master/spring-security-modules/spring-security-web-boot-2)

请注意，使用 HTTP 基本身份验证时无法注销，因此您必须关闭并重新打开浏览器来删除此身份验证。

要运行 JUnit 测试，使用定义的 Maven 概要文件`entryPoints`和下面的命令:

`mvn clean install -PentryPoints`