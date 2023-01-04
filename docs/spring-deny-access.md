# 在缺少对 Spring 控制器方法的@PreAuthorize 时拒绝访问

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-deny-access>

## 1.介绍

在我们关于 [Spring 方法安全性](/web/20220630135241/https://www.baeldung.com/spring-security-method-security)的教程中，我们看到了如何使用`@PreAuthorize`和`@PostAuthorize`注释。

在本教程中，我们将看到**如何拒绝对缺少授权注释**的方法的访问。

## 2.默认安全性

毕竟，我们只是人类，所以我们可能会忘记保护我们的一个端点。不幸的是，没有简单的方法来拒绝对无注释端点的访问。

幸运的是，Spring Security 默认要求对所有端点进行身份验证。然而，它不需要特定的角色。此外，当我们没有添加安全注释时，它**不会拒绝访问。**

## 3.设置

首先，让我们看看这个例子的应用程序。我们有一个简单的 Spring Boot 应用程序:

```java
@SpringBootApplication
public class DenyApplication {
    public static void main(String[] args) {
        SpringApplication.run(DenyApplication.class, args);
    }
}
```

其次，我们有一个安全配置。我们设置了两个用户并启用了前/后注释:

```java
@Configuration
@EnableWebSecurity
@EnableGlobalMethodSecurity(prePostEnabled = true)
public class DenyMethodSecurityConfig extends GlobalMethodSecurityConfiguration {
    @Bean
    public UserDetailsService userDetailsService() {
        return new InMemoryUserDetailsManager(
            User.withUsername("user").password("{noop}password").roles("USER").build(),
            User.withUsername("guest").password("{noop}password").roles().build()
        );
    }
}
```

最后，我们有一个带有两种方法的 rest 控制器。然而，我们“忘记”保护`/bye`端点:

```java
@RestController
public class DenyOnMissingController {
    @GetMapping(path = "hello")
    @PreAuthorize("hasRole('USER')")
    public String hello() {
        return "Hello world!";
    }

    @GetMapping(path = "bye")
    // whoops!
    public String bye() {
        return "Bye bye world!";
    }
}
```

运行示例时，我们可以用`user` / `password`登录。然后，我们访问`/hello`端点。我们也可以用`guest` / `guest`签到。在这种情况下，我们无法访问`/hello`端点。

然而，**任何经过身份验证的用户都可以访问`/bye`端点**。在下一节中，我们将编写一个测试来证明这一点。

## 4.测试解决方案

使用 [MockMvc](/web/20220630135241/https://www.baeldung.com/integration-testing-in-spring#3-mocking-web-context-beans) 我们可以建立一个测试。我们检查我们的非注释方法是否仍然可访问:

```java
@RunWith(SpringJUnit4ClassRunner.class)
@SpringBootTest(classes = DenyApplication.class)
public class DenyOnMissingControllerIntegrationTest {
    @Rule
    public ExpectedException expectedException = ExpectedException.none();

    @Autowired
    private WebApplicationContext context;
    private MockMvc mockMvc;

    @Before
    public void setUp() {
        mockMvc = MockMvcBuilders.webAppContextSetup(context).build();
    }

    @Test
    @WithMockUser(username = "user")
    public void givenANormalUser_whenCallingHello_thenAccessDenied() throws Exception {
        mockMvc.perform(get("/hello"))
          .andExpect(status().isOk())
          .andExpect(content().string("Hello world!"));
    }

    @Test
    @WithMockUser(username = "user")
    // This will fail without the changes from the next section
    public void givenANormalUser_whenCallingBye_thenAccessDenied() throws Exception {
        expectedException.expectCause(isA(AccessDeniedException.class));

        mockMvc.perform(get("/bye"));
    }
}
```

第二个测试失败了，因为`/bye`端点是可访问的。在下一节中，**我们将更新我们的配置，以拒绝未标注端点**的访问。

## 5.解决方案:默认拒绝

让我们扩展我们的`MethodSecurityConfig`类并设置一个`MethodSecurityMetadataSource:`

```java
@Configuration 
@EnableWebSecurity 
@EnableGlobalMethodSecurity(prePostEnabled = true) 
public class DenyMethodSecurityConfig extends GlobalMethodSecurityConfiguration {
    @Override
    protected MethodSecurityMetadataSource customMethodSecurityMetadataSource() {
        return new CustomPermissionAllowedMethodSecurityMetadataSource();
    }
    // setting up in memory users not repeated
    ...
}
```

现在让我们实现`MethodSecurityMetadataSource`接口:

```java
public class CustomPermissionAllowedMethodSecurityMetadataSource 
  extends AbstractFallbackMethodSecurityMetadataSource {
    @Override
    protected Collection findAttributes(Class<?> clazz) { return null; }

    @Override
    protected Collection findAttributes(Method method, Class<?> targetClass) {
        Annotation[] annotations = AnnotationUtils.getAnnotations(method);
        List attributes = new ArrayList<>();

        // if the class is annotated as @Controller we should by default deny access to all methods
        if (AnnotationUtils.findAnnotation(targetClass, Controller.class) != null) {
            attributes.add(DENY_ALL_ATTRIBUTE);
        }

        if (annotations != null) {
            for (Annotation a : annotations) {
                // but not if the method has at least a PreAuthorize or PostAuthorize annotation
                if (a instanceof PreAuthorize || a instanceof PostAuthorize) {
                    return null;
                }
            }
        }
        return attributes;
    }

    @Override
    public Collection getAllConfigAttributes() { return null; }
}
```

**我们将把`DENY_ALL_ATTRIBUTE `添加到`@Controller`类的所有方法中。**

但是，如果找到一个`@PreAuthorize` / `@PostAuthorize`注释，我们不会添加它们。我们这样做是通过返回`null`，[表示没有元数据适用](https://web.archive.org/web/20220630135241/https://docs.spring.io/spring-security/site/docs/5.1.x/api/org/springframework/security/access/method/AbstractFallbackMethodSecurityMetadataSource.html#findAttributes-java.lang.reflect.Method-java.lang.Class-)。

有了更新的代码，我们的`/bye`端点得到了保护，测试成功了。

## 6.结论

在这个简短的教程中，我们向**展示了如何保护缺少`@PreAuthorize` / `@PostAuthorize`注释**的端点。

此外，我们还展示了非注释方法现在确实受到了保护。

和往常一样，GitHub 上的[提供了这篇文章的完整源代码。](https://web.archive.org/web/20220630135241/https://github.com/eugenp/tutorials/tree/master/spring-security-modules/spring-security-core)