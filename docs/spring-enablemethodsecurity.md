# Spring @EnableMethodSecurity 批注

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-enablemethodsecurity>

## 1.概观

使用 [Spring Security](/web/20221115043036/https://www.baeldung.com/security-spring) ，我们可以为端点等方法配置应用程序的认证和授权。例如，如果用户在我们的域上进行身份验证，我们可以通过对现有方法应用限制来分析他对应用程序的使用。

使用 [`@EnableGlobalMethodSecurity`](https://web.archive.org/web/20221115043036/https://docs.spring.io/spring-security/reference/servlet/authorization/method-security.html#jc-enable-global-method-security) 注释一直是一个标准，直到 5.6 版本的 [`@EnableMethodSecurity`](https://web.archive.org/web/20221115043036/https://docs.spring.io/spring-security/reference/servlet/authorization/method-security.html#_enablemethodsecurity) 引入了一种更加灵活的方式来为[方法安全性](/web/20221115043036/https://www.baeldung.com/spring-security-method-security)配置授权。

在本教程中，我们将看到`@EnableMethodSecurity`如何替换旧的注释。我们还将看到它的前身和一些代码示例之间的区别。

## 2.`@EnableMethodSecurity`对`@EnableGlobalMethodSecurity`

如果我们首先检查一下方法授权如何与`@EnableGlobalMethodSecurity`一起工作，我们可以更好地理解这个主题。

### 2.1.`@EnableGlobalMethodSecurity`

`[@EnableGlobalMethodSecurity](https://web.archive.org/web/20221115043036/https://docs.spring.io/spring-security/site/docs/current/api/org/springframework/security/config/annotation/method/configuration/EnableGlobalMethodSecurity.html)`是一个功能接口，我们需要它和 [`@EnableWebSecurity`](https://web.archive.org/web/20221115043036/https://docs.spring.io/spring-security/site/docs/current/api/org/springframework/security/config/annotation/web/configuration/EnableWebSecurity.html) 一起创建我们的安全层并获得方法授权。

让我们创建一个示例配置类:

```java
@EnableWebSecurity
@EnableGlobalMethodSecurity(prePostEnabled = true, securedEnabled = true, jsr250Enabled = true)
@Configuration
public class SecurityConfig {
    // security beans
}
```

所有方法安全实现都使用一个在需要授权时触发的[`MethodInterceptor`](https://web.archive.org/web/20221115043036/https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/aopalliance/intercept/MethodInterceptor.html)`. `在这种情况下， [`GlobalMethodSecurityConfiguration`](https://web.archive.org/web/20221115043036/https://docs.spring.io/spring-security/site/docs/6.0.0-M3/api/org/springframework/security/config/annotation/method/configuration/GlobalMethodSecurityConfiguration.html) 类是启用全局方法安全的基本配置。

**[`methodSecurityInterceptor()`](https://web.archive.org/web/20221115043036/https://docs.spring.io/spring-security/site/docs/6.0.0-M3/api/org/springframework/security/config/annotation/method/configuration/GlobalMethodSecurityConfiguration.html#methodSecurityInterceptor(org.springframework.security.access.method.MethodSecurityMetadataSource))方法使用我们可能想要使用的不同授权类型的元数据来创建`MethodInterceptor` bean。**

Spring Security 支持三种内置的方法安全注释:

*   `prePostEnabled`对于 Spring 前/后注释
*   `securedEnabled`为弹簧`@Secured`标注
*   `jsr250Enabled`为标准 Java `@RoleAllowed`注释

此外，在`methodSecurityInterceptor()`中，还设置了:

*   [`AccessDecisionManager`](https://web.archive.org/web/20221115043036/https://docs.spring.io/spring-security/site/docs/6.0.0-M3/api/org/springframework/security/access/AccessDecisionManager.html) ，它使用基于[投票的](https://web.archive.org/web/20221115043036/https://docs.spring.io/spring-security/reference/servlet/authorization/architecture.html#authz-voting-based)机制来“决定”是否授权访问
*   我们从安全上下文中得到它，它负责身份验证
*   [`AfterInvocationManager`](https://web.archive.org/web/20221115043036/https://docs.spring.io/spring-security/site/docs/current/api/org/springframework/security/access/intercept/AfterInvocationManager.html) ，负责为前置/后置表达式提供处理程序

该框架有一个投票机制来拒绝或授予对特定方法的访问。我们可以将此作为`[Jsr250Voter](https://web.archive.org/web/20221115043036/https://docs.spring.io/spring-security/site/docs/current/api/org/springframework/security/access/annotation/Jsr250Voter.html):`的一个例子

```java
@Override
public int vote(Authentication authentication, Object object, Collection<ConfigAttribute> definition) {
    boolean jsr250AttributeFound = false;
    for (ConfigAttribute attribute : definition) {
        if (Jsr250SecurityConfig.PERMIT_ALL_ATTRIBUTE.equals(attribute)) {
            return ACCESS_GRANTED;
        }
        if (Jsr250SecurityConfig.DENY_ALL_ATTRIBUTE.equals(attribute)) {
            return ACCESS_DENIED;
        }
        if (supports(attribute)) {
            jsr250AttributeFound = true;
            // Attempt to find a matching granted authority
            for (GrantedAuthority authority : authentication.getAuthorities()) {
                if (attribute.getAttribute().equals(authority.getAuthority())) {
                    return ACCESS_GRANTED;
                }
            }
        }
    }
    return jsr250AttributeFound ? ACCESS_DENIED : ACCESS_ABSTAIN;
}
```

投票时，Spring Security 从当前方法中提取元数据属性，例如，我们的 REST 端点。最后，它根据用户授予的权限对它们进行检查。

我们还应注意到，投票人可能不支持投票制度而弃权。

然后，我们的`AccessDecisionManager`评估所有来自可用投票者的回复:

```java
for (AccessDecisionVoter voter : getDecisionVoters()) {
    int result = voter.vote(authentication, object, configAttributes);
    switch (result) {
        case AccessDecisionVoter.ACCESS_GRANTED:
            return;
        case AccessDecisionVoter.ACCESS_DENIED:
            deny++;
            break;
        default:
            break;
    }
}
if (deny > 0) {
    throw new AccessDeniedException(this.messages.getMessage("AbstractAccessDecisionManager.accessDenied", "Access is denied"));
}
```

如果我们想要定制我们的 beans，我们可以扩展`GlobalMethodSecurityConfiguration`类`. `例如，我们可能想要一个[定制安全表达式](/web/20221115043036/https://www.baeldung.com/spring-security-create-new-custom-security-expression)，而不是内置了 Spring 安全的 [Spring EL](https://web.archive.org/web/20221115043036/https://docs.spring.io/spring-security/reference/servlet/authorization/expression-based.html#el-access) 。或者我们可能想让我们的[自定义安全投票器](/web/20221115043036/https://www.baeldung.com/spring-security-custom-voter)。

### 2.2.`@EnableMethodSecurity`

通过`@EnableMethodSecurity`，我们可以看到 Spring Security 将授权类型转移到基于 bean 的配置的意图。

我们现在对每种类型都有一个配置，而不是一个全局配置。让我们看看，例如， [`Jsr250MethodSecurityConfiguration`](https://web.archive.org/web/20221115043036/https://docs.spring.io/spring-security/site/docs/current/api/org/springframework/security/access/annotation/Jsr250SecurityConfig.html) :

```java
@Configuration(proxyBeanMethods = false)
@Role(BeanDefinition.ROLE_INFRASTRUCTURE)
class Jsr250MethodSecurityConfiguration {
    // ...
    @Bean
    @Role(BeanDefinition.ROLE_INFRASTRUCTURE)
    Advisor jsr250AuthorizationMethodInterceptor() {
        return AuthorizationManagerBeforeMethodInterceptor.jsr250(this.jsr250AuthorizationManager);
    }

    @Autowired(required = false)
    void setGrantedAuthorityDefaults(GrantedAuthorityDefaults grantedAuthorityDefaults) {
        this.jsr250AuthorizationManager.setRolePrefix(grantedAuthorityDefaults.getRolePrefix());
    }
}
```

**`MethodInterceptor`本质上包含一个 [`AuthorizationManager`](https://web.archive.org/web/20221115043036/https://docs.spring.io/spring-security/site/docs/current/api/org/springframework/security/authorization/AuthorizationManager.html) ，它现在将检查和返回一个`AuthorizationDecision`对象的责任委托给适当的实现**，在本例中是`AuthenticatedAuthorizationManager` **:**

```java
@Override
public AuthorizationDecision check(Supplier<Authentication> authentication, T object) {
    boolean granted = isGranted(authentication.get());
    return new AuthorityAuthorizationDecision(granted, this.authorities);
}

private boolean isGranted(Authentication authentication) {
    return authentication != null && authentication.isAuthenticated() && isAuthorized(authentication);
}

private boolean isAuthorized(Authentication authentication) {
    Set<String> authorities = AuthorityUtils.authorityListToSet(this.authorities);
    for (GrantedAuthority grantedAuthority : authentication.getAuthorities()) {
        if (authorities.contains(grantedAuthority.getAuthority())) {
            return true;
        }
    }
    return false;
} 
```

如果我们不能访问资源，`MethodInterceptor`将抛出一个`AccesDeniedException`:

```java
AuthorizationDecision decision = this.authorizationManager.check(AUTHENTICATION_SUPPLIER, mi);
if (decision != null && !decision.isGranted()) {
    // ...
    throw new AccessDeniedException("Access Denied");
}
```

## 3.`@EnableMethodSecurity`特性

`@EnableMethodSecurity`与之前的传统实施相比，带来了微小和重大的改进。

### 3.1.微小的改进

仍然支持所有授权类型。例如，它仍然符合 [JSR-250](https://web.archive.org/web/20221115043036/https://www.jcp.org/en/jsr/detail?id=250) 。然而，我们不需要将`prePostEnabled` 添加到注释中，因为它现在默认为`true:`

```java
@EnableMethodSecurity(securedEnabled = true, jsr250Enabled = true) 
```

如果我们想禁用它，我们需要将`prePostEnabled` 设置为`false`。

### 3.2.主要改进

`GlobalMethodSecurityConfiguration`类不再被使用了。 **Spring Security 用分段配置和一个`[AuthorizationManager](https://web.archive.org/web/20221115043036/https://docs.spring.io/spring-security/reference/servlet/authorization/architecture.html#_the_authorizationmanager)`来代替它，这意味着我们可以在不扩展任何基本配置类**的情况下定义我们的授权 beans。

值得注意的是，`AuthorizationManager`接口是通用的，可以适应任何对象，尽管标准安全性适用于`MethodInvocation`:

```java
AuthorizationDecision check(Supplier<Authentication> authentication, T object); 
```

**总的来说，这为我们提供了使用[委托](https://web.archive.org/web/20221115043036/https://en.wikipedia.org/wiki/Delegation_(object-oriented_programming))的细粒度授权。**所以，实际上，我们对每种类型都有一个`AuthorizationManager`。当然，我们也可以自己建造。

此外，这也意味着`@EnableMethodSecurity`不允许`[@AspectJ](/web/20221115043036/https://www.baeldung.com/aspectj)`像遗留实现中一样使用`AspectJ`方法拦截器进行注释:

```java
public final class AspectJMethodSecurityInterceptor extends MethodSecurityInterceptor {
    public Object invoke(JoinPoint jp) throws Throwable {
        return super.invoke(new MethodInvocationAdapter(jp));
    }
    // ...
}
```

然而，我们仍然有 AOP 和 T2 的全力支持。例如，让我们看看我们之前讨论过的`Jsr250MethodSecurityConfiguration`使用的拦截器:

```java
public final class AuthorizationManagerBeforeMethodInterceptor
  implements Ordered, MethodInterceptor, PointcutAdvisor, AopInfrastructureBean {
    // ...
    public AuthorizationManagerBeforeMethodInterceptor(
      Pointcut pointcut, AuthorizationManager<MethodInvocation> authorizationManager) {
        Assert.notNull(pointcut, "pointcut cannot be null");
        Assert.notNull(authorizationManager, "authorizationManager cannot be null");
        this.pointcut = pointcut;
        this.authorizationManager = authorizationManager;
    }

    @Override
    public Object invoke(MethodInvocation mi) throws Throwable {
        attemptAuthorization(mi);
        return mi.proceed();
    }
}
```

## 4.自定义`AuthorizationManager`应用程序

因此，让我们看看如何创建自定义授权管理器。

假设我们有想要应用策略的端点。我们希望只有当用户有权访问该策略时才授权他。否则，我们将阻止该用户。

第一步，我们通过添加一个字段来访问受限策略，从而定义我们的用户:

```java
public class SecurityUser implements UserDetails {
    private String userName;
    private String password;
    private List<GrantedAuthority> grantedAuthorityList;
    private boolean accessToRestrictedPolicy;

    // getters and setters
}
```

现在，让我们看看我们的身份验证层，它定义了系统中的用户。为此，我们将创建一个自定义 [`UserDetailService`](https://web.archive.org/web/20221115043036/https://docs.spring.io/spring-security/site/docs/5.7.5/api/org/springframework/security/core/userdetails/UserDetailsService.html) 。我们将使用内存映射来存储用户:

```java
public class CustomUserDetailService implements UserDetailsService {
    private final Map<String, SecurityUser> userMap = new HashMap<>();

    public CustomUserDetailService(BCryptPasswordEncoder bCryptPasswordEncoder) {
        userMap.put("user", createUser("user", bCryptPasswordEncoder.encode("userPass"), false, "USER"));
        userMap.put("admin", createUser("admin", bCryptPasswordEncoder.encode("adminPass"), true, "ADMIN", "USER"));
    }

    @Override
    public UserDetails loadUserByUsername(final String username) throws UsernameNotFoundException {
        return Optional.ofNullable(map.get(username))
          .orElseThrow(() -> new UsernameNotFoundException("User " + username + " does not exists"));
    }

    private SecurityUser createUser(String userName, String password, boolean withRestrictedPolicy, String... role) {
        return SecurityUser.builder().withUserName(userName)
          .withPassword(password)
          .withGrantedAuthorityList(Arrays.stream(role)
            .map(SimpleGrantedAuthority::new)
            .collect(Collectors.toList()))
          .withAccessToRestrictedPolicy(withRestrictedPolicy);
    }
}
```

一旦用户存在于我们的系统中，我们希望通过检查他是否有权访问某些受限策略来限制他可以访问的信息。

为了演示，我们创建一个 Java 注释`@Policy`来应用于方法和策略枚举:

```java
@Target(METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface Policy {
    PolicyEnum value();
}
```

```java
public enum PolicyEnum {
    RESTRICTED, OPEN
}
```

让我们创建一个我们想要应用这个策略的服务:

```java
@Service
public class PolicyService {
    @Policy(PolicyEnum.OPEN)
    public String openPolicy() {
        return "Open Policy Service";
    }

    @Policy(PolicyEnum.RESTRICTED)
    public String restrictedPolicy() {
        return "Restricted Policy Service";
    }
}
```

我们不能使用内置的授权管理器，比如`Jsr250AuthorizationManager`。它不知道何时以及如何拦截服务策略检查。因此，让我们来定义我们的自定义管理器:

```java
public class CustomAuthorizationManager<T> implements AuthorizationManager<MethodInvocation> {
    ...
    @Override
    public AuthorizationDecision check(Supplier<Authentication> authentication, MethodInvocation methodInvocation) {
        if (hasAuthentication(authentication.get())) {
            Policy policyAnnotation = AnnotationUtils.findAnnotation(methodInvocation.getMethod(), Policy.class);
            SecurityUser user = (SecurityUser) authentication.get().getPrincipal();
            return new AuthorizationDecision(Optional.ofNullable(policyAnnotation)
              .map(Policy::value).filter(policy -> policy == PolicyEnum.OPEN 
                || (policy == PolicyEnum.RESTRICTED && user.hasAccessToRestrictedPolicy())).isPresent());
        }
        return new AuthorizationDecision(false);
    }

    private boolean hasAuthentication(Authentication authentication) {
        return authentication != null && isNotAnonymous(authentication) && authentication.isAuthenticated();
    }

    private boolean isNotAnonymous(Authentication authentication) {
        return !this.trustResolver.isAnonymous(authentication);
    }
}
```

当服务方法被触发时，我们再次检查用户是否已经过身份验证。然后，如果策略是开放的，我们就授予访问权。在受限的情况下，我们检查用户是否有权访问受限的策略。

为此，我们需要定义一个`MethodInterceptor`,例如，在执行之前，但也可能在执行之后。因此，让我们将它与我们的安全配置类包装在一起:

```java
@EnableWebSecurity
@EnableMethodSecurity
@Configuration
public class SecurityConfig {
    @Bean
    public AuthenticationManager authenticationManager(
      HttpSecurity httpSecurity, UserDetailsService userDetailsService, BCryptPasswordEncoder bCryptPasswordEncoder) throws Exception {
        AuthenticationManagerBuilder authenticationManagerBuilder = httpSecurity.getSharedObject(AuthenticationManagerBuilder.class);
        authenticationManagerBuilder.userDetailsService(userDetailsService).passwordEncoder(bCryptPasswordEncoder);
        return authenticationManagerBuilder.build();
    }

    @Bean
    public UserDetailsService userDetailsService(BCryptPasswordEncoder bCryptPasswordEncoder) {
        return new CustomUserDetailService(bCryptPasswordEncoder);
    }

    @Bean
    public AuthorizationManager<MethodInvocation> authorizationManager() {
        return new CustomAuthorizationManager<>();
    }

    @Bean
    @Role(ROLE_INFRASTRUCTURE)
    public Advisor authorizationManagerBeforeMethodInterception(AuthorizationManager<MethodInvocation> authorizationManager) {
        JdkRegexpMethodPointcut pattern = new JdkRegexpMethodPointcut();
        pattern.setPattern("com.baeldung.enablemethodsecurity.services.*");
        return new AuthorizationManagerBeforeMethodInterceptor(pattern, authorizationManager);
    }

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http.csrf()
          .disable()
          .authorizeRequests()
          .anyRequest()
          .authenticated()
          .and()
          .sessionManagement()
          .sessionCreationPolicy(SessionCreationPolicy.STATELESS);

        return http.build();
    }

    @Bean
    public BCryptPasswordEncoder bCryptPasswordEncoder() {
        return new BCryptPasswordEncoder();
    }
}
```

我们用的是`AuthorizationManagerBeforeMethodInterceptor`。它与我们的策略服务模式相匹配，并使用定制的授权管理器。

此外，我们还需要让我们的`AuthenticationManager`意识到习俗`UserDetailsService`。然后，当 Spring Security 截获服务方法时，我们可以访问我们的自定义用户，并检查用户的策略访问。

## 5.试验

让我们定义一个 REST 控制器:

```java
@RestController
public class ResourceController {
    // ...
    @GetMapping("/openPolicy")
    public String openPolicy() {
        return policyService.openPolicy();
    }

    @GetMapping("/restrictedPolicy")
    public String restrictedPolicy() {
        return policyService.restrictedPolicy();
    }
}
```

我们将在应用程序中使用 [Spring Boot 测试](/web/20221115043036/https://www.baeldung.com/spring-boot-testing)来模拟方法安全性:

```java
@SpringBootTest(classes = EnableMethodSecurityApplication.class)
public class EnableMethodSecurityTest {
    @Autowired
    private WebApplicationContext context;

    private MockMvc mvc;

    @BeforeEach
    public void setup() {
        mvc = MockMvcBuilders.webAppContextSetup(context)
          .apply(springSecurity())
          .build();
    }

    @Test
    @WithUserDetails(value = "admin")
    public void whenAdminAccessOpenEndpoint_thenOk() throws Exception {
        mvc.perform(get("/openPolicy"))
          .andExpect(status().isOk());
    }

    @Test
    @WithUserDetails(value = "admin")
    public void whenAdminAccessRestrictedEndpoint_thenOk() throws Exception {
        mvc.perform(get("/restrictedPolicy"))
          .andExpect(status().isOk());
    }

    @Test
    @WithUserDetails()
    public void whenUserAccessOpenEndpoint_thenOk() throws Exception {
        mvc.perform(get("/openPolicy"))
          .andExpect(status().isOk());
    }

    @Test
    @WithUserDetails()
    public void whenUserAccessRestrictedEndpoint_thenIsForbidden() throws Exception {
        mvc.perform(get("/restrictedPolicy"))
          .andExpect(status().isForbidden());
    }
}
```

所有响应都应该被授权，除了用户调用他无权访问受限策略的服务的响应。

## 6.结论

在本文中，我们已经看到了`@EnableMethodSecurity`的主要特性以及它如何取代`@EnableGlobalMethodSecurity.`

我们还通过查看实现流程`.` 了解了这些注释之间的差异。然后，我们讨论了`@EnableMethodSecurity`如何通过基于 bean 的配置提供更大的灵活性`.`。最后，我们了解了如何创建自定义授权管理器和 MVC 测试。

一如既往，我们可以在 GitHub 上找到工作代码示例[。](https://web.archive.org/web/20221115043036/https://github.com/eugenp/tutorials/tree/master/spring-security-modules/spring-security-web-boot-4)