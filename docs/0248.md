# spring @ enable web security vs . @ EnableGlobalMethodSecurity

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-enablewebsecurity-vs-enableglobalmethodsecurity>

## 1.概观

我们可能希望在 Spring Boot 应用程序的不同路径中应用多个安全过滤器。

在本教程中，我们将了解两种定制安全性的方法——通过使用 [`@EnableWebSecurity`](https://web.archive.org/web/20220628132448/https://docs.spring.io/spring-security/site/docs/current/api/org/springframework/security/config/annotation/web/configuration/EnableWebSecurity.html) 和 [`@EnableGlobalMethodSecurity`](https://web.archive.org/web/20220628132448/https://docs.spring.io/spring-security/site/docs/current/api/org/springframework/security/config/annotation/method/configuration/GlobalMethodSecurityConfiguration.html) 。

为了说明不同之处，我们将使用一个简单的应用程序，其中包含一些管理资源和经过身份验证的用户资源。我们还会给它一个包含公共资源的部分，我们很乐意让任何人下载。

## 2.Spring Boot 安全

### 2.1.Maven 依赖性

无论我们采用哪种方法，为了安全起见，我们首先需要添加 spring boot starter:

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>
```

### 2.2。Spring Boot 自动配置

在类路径上有了 Spring Security， [Spring Boot 安全自动配置](/web/20220628132448/https://www.baeldung.com/spring-boot-security-autoconfiguration "Spring Boot Security Auto-Configuration")的`WebSecurityEnablerConfiguration`为我们激活`@EnableWebSecurity`。

这将 Spring 的默认安全配置应用于我们的应用程序。

**默认安全** **激活 HTTP 安全过滤器和安全过滤器链**，并对我们的端点应用基本认证。

## 3.保护我们的终端

对于我们的第一种方法，让我们从创建一个扩展了`WebSecurityConfigurerAdapter`的`MySecurityConfigurer`类开始，确保我们用`@EnableWebSecurity.`对它进行了注释

```
@EnableWebSecurity
public class MySecurityConfigurer extends WebSecurityConfigurerAdapter {
}
```

通过扩展适配器，我们获得了 Spring Security 其他防御的好处，同时还能够添加定制。

### 3.1.快速浏览默认 Web 安全性

首先，让我们看看`WebSecurityConfigurerAdapter`的默认`configure` 方法，这样我们就知道我们要覆盖什么:

```
@Override
protected void configure(HttpSecurity http) {
    http.authorizeRequests((requests) -> requests.anyRequest().authenticated());
    http.formLogin();
    http.httpBasic();
}
```

这里我们看到，我们收到的任何请求都经过了身份验证，我们有一个基本的表单登录来提示输入凭证。

当我们想使用`HttpSecurity` DSL 时，我们把它写成:

```
http.authorizeRequests().anyRequest().authenticated()
  .and().formLogin()
  .and().httpBasic()
```

### 3.2.要求用户拥有适当的角色

现在让我们配置我们的安全性，只允许拥有`ADMIN`角色的用户访问我们的`/admin`端点。我们还将只允许拥有`USER`角色的用户访问我们的`/protected`端点。

我们通过覆盖`configure`的`HttpSecurity`霸主来实现这一点:

```
@Override
protected void configure(HttpSecurity http) {
    http.authorizeRequests()
      .antMatchers("/admin/**")
      .hasRole("ADMIN")
      .antMatchers("/protected/**")
      .hasRole("USER");
}
```

### 3.3.放松对公共资源的安全保护

我们不需要对我们的公共`/hello`资源进行认证，所以我们将配置`WebSecurity`不为它们做任何事情。

就像之前一样，让我们重写`WebSecurityConfigurerAdapter`的`configure`方法之一，但是这次`WebSecurity`重载了:

```
@Override
public void configure(WebSecurity web) {
    web.ignoring()
      .antMatchers("/hello/*");
}
```

### 3.4.替换 Spring 的默认安全性

虽然我们的大部分需求可以通过扩展`WebSecurityConfigurerAdapter`来满足，但是可能有些时候我们想要完全替换 Spring 的默认安全配置。为此，我们可以实现`WebSecurityConfigurer`而不是扩展`WebSecurityConfigurerAdapter`。

我们应该注意到**通过实现`WebSecurityConfigurer,`我们失去了 Spring 的标准安全防御**，所以在走这条路之前我们应该非常仔细地考虑。

## 4.使用注释保护我们的端点

为了使用注释驱动的方法来应用安全性，我们可以使用`@EnableGlobalMethodSecurity.`

### 4.1.使用安全注释要求用户拥有适当的角色

现在让我们使用方法注释来配置我们的安全性，只允许`ADMIN`用户访问我们的`/admin`端点，允许`USER`用户访问我们的`/protected`端点。

让我们通过在`EnableGlobalMethodSecurity`注释中设置`jsr250Enabled=true`来启用 [JSR-250](https://web.archive.org/web/20220628132448/https://jcp.org/aboutJava/communityprocess/final/jsr250/index.html) 注释:

```
@EnableGlobalMethodSecurity(jsr250Enabled = true)
@Controller
public class AnnotationSecuredController {
    @RolesAllowed("ADMIN")
    @RequestMapping("/admin")
    public String adminHello() {
        return "Hello Admin";
    }

    @RolesAllowed("USER")
    @RequestMapping("/protected")
    public String jsr250Hello() {
        return "Hello Jsr250";
    }
}
```

### 4.2.强制所有公共方法具有安全性

当我们使用注释作为实现安全性的方式时，我们可能会忘记注释方法。这将无意中造成一个安全漏洞。

为了防止这种情况，**我们应该[拒绝对所有没有授权注释](/web/20220628132448/https://www.baeldung.com/spring-deny-access)的方法的访问。**

### 4.3.允许访问公共资源

Spring 的默认安全性强制对我们所有的端点进行认证，不管我们是否添加了基于角色的安全性。

尽管我们前面的例子对我们的`/admin`和`/protected`端点应用了安全性，但是我们仍然希望允许访问`/hello`中基于文件的资源。

虽然我们可以再次扩展`WebSecurityAdapter`,但是 Spring 为我们提供了一个更简单的选择。

用注释保护了我们的方法之后，我们现在可以添加`WebSecurityCustomizer`来打开`/hello/*`资源:

```
public class MyPublicPermitter implements WebSecurityCustomizer {
    public void customize(WebSecurity webSecurity) {
        webSecurity.ignoring()
          .antMatchers("/hello/*");
    }
}
```

或者，我们可以简单地创建一个 bean，在我们的配置类中实现它:

```
@Configuration
public class MyWebConfig {
    @Bean
    public WebSecurityCustomizer ignoreResources() {
        return (webSecurity) -> webSecurity
          .ignoring()
          .antMatchers("/hello/*");
    }
}
```

当 Spring Security 初始化时，它会调用它找到的任何`WebSecurityCustomizer` s，包括我们的。

## 5.测试我们的安全性

既然我们已经配置了我们的安全性，我们应该检查它的行为是否符合我们的预期。

根据我们为安全性选择的方法，我们有一个或两个自动化测试选项。我们既可以向应用程序发送 web 请求，也可以直接调用控制器方法。

### 5.1。通过 Web 请求进行测试

对于第一个选项，我们将创建一个带有`@TestRestTemplate`的`@SpringBootTest`测试类:

```
@RunWith(SpringRunner.class)
@SpringBootTest(webEnvironment = RANDOM_PORT)
public class WebSecuritySpringBootIntegrationTest {
    @Autowired
    private TestRestTemplate template;
}
```

现在，让我们添加一个测试来确保我们的公共资源可用:

```
@Test
public void givenPublicResource_whenGetViaWeb_thenOk() {
    ResponseEntity<String> result = template.getForEntity("/hello/baeldung.txt", String.class);
    assertEquals("Hello From Baeldung", result.getBody());
}
```

我们还可以看到当我们尝试访问我们的一个受保护资源时会发生什么:

```
@Test
public void whenGetProtectedViaWeb_thenForbidden() {
    ResponseEntity<String> result = template.getForEntity("/protected", String.class);
    assertEquals(HttpStatus.FORBIDDEN, result.getStatusCode());
}
```

这里我们得到了一个`FORBIDDEN`响应，因为我们的匿名请求没有所需的角色。

因此，无论我们选择哪种安全方法，我们都可以使用这种方法来测试我们的安全应用程序。

### 5.2。通过自动布线和注释进行测试

现在让我们看看我们的第二个选择。让我们设置一个`@SpringBootTest`并自动连接我们的`AnnotationSecuredController:`

```
@RunWith(SpringRunner.class)
@SpringBootTest(webEnvironment = RANDOM_PORT)
public class GlobalMethodSpringBootIntegrationTest {
    @Autowired
    private AnnotationSecuredController api;
}
```

让我们从使用`@WithAnonymousUser`测试我们的可公开访问的方法开始:

```
@Test
@WithAnonymousUser
public void givenAnonymousUser_whenPublic_thenOk() {
    assertThat(api.publicHello()).isEqualTo(HELLO_PUBLIC);
} 
```

现在我们已经访问了我们的公共资源，让我们使用`@WithMockUser`注释来访问我们受保护的方法。

首先，让我们用一个具有“用户”角色的用户来测试我们的 JSR-250 保护方法:

```
@WithMockUser(username="baeldung", roles = "USER")
@Test
public void givenUserWithRole_whenJsr250_thenOk() {
    assertThat(api.jsr250Hello()).isEqualTo("Hello Jsr250");
}
```

现在，当我们的用户没有正确的角色时，让我们尝试访问相同的方法:

```
@WithMockUser(username="baeldung", roles = "NOT-USER")
@Test(expected = AccessDeniedException.class)
public void givenWrongRole_whenJsr250_thenAccessDenied() {
    api.jsr250Hello();
}
```

我们的请求被 Spring Security 拦截，抛出了一个`AccessDeniedException`。

我们只能在选择基于注释的安全性时使用这种方法。

## 6.注释注意事项

当我们选择基于注释的方法时，有一些要点需要注意。

只有当我们通过公共方法进入一个类时，我们的带注释的安全性才会被应用。

### 6.1.间接方法调用

早些时候，当我们调用一个带注释的方法时，我们看到我们的安全性被成功地应用。然而，现在让我们在同一个类中创建一个公共方法，但是没有安全注释。我们将让它调用我们的带注释的`jsr250Hello`方法:

```
@GetMapping("/indirect")
public String indirectHello() {
    return jsr250Hello();
} 
```

现在让我们只使用匿名访问来调用我们的“/间接”端点:

```
@Test
@WithAnonymousUser
public void givenAnonymousUser_whenIndirectCall_thenNoSecurity() {
    assertThat(api.indirectHello()).isEqualTo(HELLO_JSR_250);
}
```

我们的测试通过了，因为我们的“安全”方法被调用，而没有触发任何安全性。换句话说，**没有安全性被应用于同一类**内的内部调用。

### 6.2.对不同类的间接方法调用

现在让我们看看当我们的未受保护的方法调用不同类上的带注释的方法时会发生什么。

首先，让我们用带注释的方法`differentJsr250Hello`创建一个`DifferentClass` :

```
@Component
public class DifferentClass {
    @RolesAllowed("USER")
    public String differentJsr250Hello() {
        return "Hello Jsr250";
    }
}
```

现在，让我们将`DifferentClass`自动连接到我们的控制器中，并添加一个不受保护的`differentClassHello`公共方法来调用它。

```
@Autowired
DifferentClass differentClass;

@GetMapping("/differentclass")
public String differentClassHello() {
    return differentClass.differentJsr250Hello();
}
```

最后，让我们测试调用，看看我们的安全性是否得到了加强:

```
@Test(expected = AccessDeniedException.class)
@WithAnonymousUser
public void givenAnonymousUser_whenIndirectToDifferentClass_thenAccessDenied() {
    api.differentClassHello();
}
```

因此，我们看到，虽然我们的安全注释在调用同一个类中的另一个方法时不会受到尊重，但当我们调用不同类中的一个带注释的方法时，它们会受到尊重。

### 6.3.最后要注意的是

我们应该确保正确配置我们的`@EnableGlobalMethodSecurity`。如果我们不这样做，那么不管我们所有的安全注释，它们都不会有任何效果。

例如，如果我们使用 JSR-250 注释，但是我们指定了`prePostEnabled=true`而不是`jsr250Enabled=true`，那么我们的 JSR-250 注释将什么也不做！

```
@EnableGlobalMethodSecurity(prePostEnabled = true)
```

当然，我们可以声明我们将使用不止一种注释类型，方法是将它们都添加到我们的`@EnableGlobalMethodSecurity`注释中:

```
@EnableGlobalMethodSecurity(jsr250Enabled = true, prePostEnabled = true)
```

## 7.当我们需要更多的时候

相比于 JSR-250，我们还可以使用[弹簧法安全](/web/20220628132448/https://www.baeldung.com/spring-security-method-security "Introduction to Spring Method Security")。这包括为更高级的授权场景使用更强大的 [Spring 安全表达式](/web/20220628132448/https://www.baeldung.com/spring-security-expressions)语言(SpEL)。我们可以通过设置`prePostEnabled=true:`在我们的`EnableGlobalMethodSecurity`注释上启用 SpEL

```
@EnableGlobalMethodSecurity(prePostEnabled = true)
```

此外，当我们想要根据用户是否拥有一个域对象来实施安全性时，我们可以使用 [Spring 安全访问控制列表](/web/20220628132448/https://www.baeldung.com/spring-security-acl)。

我们还应该注意，当我们编写反应式应用程序时，我们使用`@EnableWebFluxSecurity`和`@EnableReactiveMethodSecurity`来代替。

## 8.结论

在本教程中，我们首先看了如何使用集中的安全规则方法和`@EnableWebSecurity.`来保护我们的应用程序

然后，我们在此基础上，通过配置我们的安全性，将这些规则放在它们影响的代码附近。我们通过使用`@EnableGlobalMethodSecurity`并注释我们想要保护的方法来做到这一点。

最后，我们介绍了一种替代方法，可以为不需要安全性的公共资源放松安全性。

与往常一样，GitHub 上的示例代码[可用。](https://web.archive.org/web/20220628132448/https://github.com/eugenp/tutorials/tree/master/spring-boot-modules/spring-boot-security)