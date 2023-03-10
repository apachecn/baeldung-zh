# Spring Boot 集成测试的春季安全性

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-security-integration-tests>

## 1。简介

执行集成测试而不需要独立的集成环境的能力对于任何软件栈来说都是一个有价值的特性。Spring Boot 与 Spring Security 的无缝集成使得测试与安全层交互的组件变得简单。

在这个快速教程中，我们将探索如何使用`@MockMvcTest`和`@SpringBootTest`来执行支持安全性的集成测试。

## 2。依赖性

让我们首先引入我们的示例所需的依赖项:

```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-test</artifactId>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>org.springframework.security</groupId>
    <artifactId>spring-security-test</artifactId>
    <scope>test</scope>
</dependency>
```

`[spring-boot-starter-web](https://web.archive.org/web/20220904220939/https://search.maven.org/search?q=g:org.springframework.boot%20AND%20a:spring-boot-starter-web&core=gav), ` `[spring-boot-starter-security](https://web.archive.org/web/20220904220939/https://search.maven.org/search?q=g:org.springframework.boot%20AND%20a:spring-boot-starter-security&core=gav),`和`[spring-boot-starter-test](https://web.archive.org/web/20220904220939/https://search.maven.org/search?q=g:org.springframework.boot%20AND%20a:spring-boot-starter-test&core=gav) `启动器为我们提供了对 Spring MVC、Spring Security 和 Spring Boot 测试工具的访问。

此外，我们将引入`[spring-security-test](https://web.archive.org/web/20220904220939/https://search.maven.org/search?q=g:org.springframework.security%20AND%20a:spring-security-test&core=gav)`来访问我们将使用的`@WithMockUser`注释。

## 3。网络安全配置

我们的 web 安全配置将非常简单。只有经过身份验证的用户才能访问匹配`/private/**`的路径。任何用户都可以使用与`/public/**`匹配的路径:

```java
@Configuration
public class WebSecurityConfigurer extends WebSecurityConfigurerAdapter {

    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        PasswordEncoder encoder = PasswordEncoderFactories.createDelegatingPasswordEncoder();
        auth.inMemoryAuthentication()
         .passwordEncoder(encoder)
         .withUser("spring")
         .password(encoder.encode("secret"))
         .roles("USER");
    }

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.authorizeRequests()
          .antMatchers("/private/**")
          .authenticated()
          .antMatchers("/public/**")
          .permitAll()
          .and()
          .httpBasic();
    }
} 
```

## 4。方法安全配置

除了我们在`WebSecurityConfigurer,`中定义的基于 URL 路径的安全性，我们还可以通过提供额外的配置文件来配置基于方法的安全性:

```java
@Configuration
@EnableGlobalMethodSecurity(prePostEnabled = true)
public class MethodSecurityConfigurer 
  extends GlobalMethodSecurityConfiguration {
}
```

这种配置支持 Spring Security 的前/后注释。如果需要额外的支持，其他属性也是可用的。有关 Spring 方法安全性的更多信息，请看我们的主题为的[文章。](/web/20220904220939/https://www.baeldung.com/spring-security-method-security)

## 5。用`@WebMvcTest` 测试控制器

当使用带有 Spring 安全性的`@WebMvcTest`注释方法时， **`MockMvc`会自动配置必要的过滤器链**，以测试我们的安全性配置。

因为`MockMvc`是为我们配置的，所以我们能够使用`@WithMockUser`进行测试，而不需要任何额外的配置:

```java
@RunWith(SpringRunner.class)
@WebMvcTest(SecuredController.class)
public class SecuredControllerWebMvcIntegrationTest {

    @Autowired
    private MockMvc mvc;

    // ... other methods

    @WithMockUser(value = "spring")
    @Test
    public void givenAuthRequestOnPrivateService_shouldSucceedWith200() throws Exception {
        mvc.perform(get("/private/hello").contentType(MediaType.APPLICATION_JSON))
          .andExpect(status().isOk());
    }
}
```

注意，使用`@WebMvcTest`将告诉 Spring Boot 只实例化 web 层，而不是整个上下文。因此，使用`@WebMvcTest `的**控制器测试将比使用其他方法**运行得更快。

## 6。用`@SpringBootTest` 测试控制器

使用`@SpringBootTest`注释测试带 Spring 安全的控制器时，**需要在设置`MockMvc`** 时显式配置过滤器链。

使用由`SecurityMockMvcConfigurer`提供的静态`springSecurity`方法是最好的方法:

```java
@RunWith(SpringRunner.class)
@SpringBootTest(webEnvironment = WebEnvironment.RANDOM_PORT)
public class SecuredControllerSpringBootIntegrationTest {

    @Autowired
    private WebApplicationContext context;

    private MockMvc mvc;

    @Before
    public void setup() {
        mvc = MockMvcBuilders
          .webAppContextSetup(context)
          .apply(springSecurity())
          .build();
    }

    // ... other methods

    @WithMockUser("spring")
    @Test
    public void givenAuthRequestOnPrivateService_shouldSucceedWith200() throws Exception {
        mvc.perform(get("/private/hello").contentType(MediaType.APPLICATION_JSON))
          .andExpect(status().isOk());
    }
} 
```

## 7。用`@SpringBootTest` 测试安全方法

`@SpringBootTest`不需要任何额外的配置来测试安全方法。我们可以**简单地直接调用方法，并根据需要使用`@WithMockUser`:**

```java
@RunWith(SpringRunner.class)
@SpringBootTest
public class SecuredMethodSpringBootIntegrationTest {

    @Autowired
    private SecuredService service;

    @Test(expected = AuthenticationCredentialsNotFoundException.class)
    public void givenUnauthenticated_whenCallService_thenThrowsException() {
        service.sayHelloSecured();
    }

    @WithMockUser(username="spring")
    @Test
    public void givenAuthenticated_whenCallServiceWithSecured_thenOk() {
        assertThat(service.sayHelloSecured()).isNotBlank();
    }
}
```

## 8。使用`@SpringBootTest`和`TestRestTemplate`和进行测试

在为安全的 REST 端点编写集成测试时，`TestRestTemplate`是一个方便的选项。

我们可以**在请求安全端点之前简单地自动连接一个模板并设置凭证:**

```java
@RunWith(SpringRunner.class)
@SpringBootTest(webEnvironment = WebEnvironment.RANDOM_PORT)
public class SecuredControllerRestTemplateIntegrationTest {

    @Autowired
    private TestRestTemplate template;

    // ... other methods

    @Test
    public void givenAuthRequestOnPrivateService_shouldSucceedWith200() throws Exception {
        ResponseEntity<String> result = template.withBasicAuth("spring", "secret")
          .getForEntity("/private/hello", String.class);
        assertEquals(HttpStatus.OK, result.getStatusCode());
    }
}
```

`TestRestTemplate`非常灵活，提供了许多有用的安全相关选项。关于`TestRestTemplate`的更多细节，请查看我们关于主题的[文章。](/web/20220904220939/https://www.baeldung.com/spring-boot-testresttemplate)

## 9。结论

在本文中，我们研究了几种执行支持安全性的集成测试的方法。

我们研究了如何使用 mvccontroller 和 REST 端点以及安全方法。

像往常一样，这里例子的所有源代码都可以在 GitHub 上找到。