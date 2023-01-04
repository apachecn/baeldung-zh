# 具有 Spring 安全性的自定义安全表达式

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-security-create-new-custom-security-expression>

## 1。概述

在本教程中，我们将关注于用 Spring Security 创建一个定制的安全表达式。

有时候，[框架](/web/20220812054610/https://www.baeldung.com/spring-security-expressions)中可用的表达式就是不够有表现力。而且，在这些情况下，建立一个语义上比现有表达式更丰富的新表达式是相对简单的。

我们将首先讨论如何创建一个自定义的`PermissionEvaluator`，然后是一个完全自定义的表达式——最后是如何覆盖一个内置的安全表达式。

## 2。用户实体

首先，让我们为创建新的安全表达式打下基础。

让我们看看我们的`User`实体——它有一个`Privileges`和一个`Organization`:

```
@Entity
public class User{
    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    private Long id;

    @Column(nullable = false, unique = true)
    private String username;

    private String password;

    @ManyToMany(fetch = FetchType.EAGER) 
    @JoinTable(name = "users_privileges", 
      joinColumns = 
        @JoinColumn(name = "user_id", referencedColumnName = "id"),
      inverseJoinColumns = 
        @JoinColumn(name = "privilege_id", referencedColumnName = "id")) 
    private Set<Privilege> privileges;

    @ManyToOne(fetch = FetchType.EAGER)
    @JoinColumn(name = "organization_id", referencedColumnName = "id")
    private Organization organization;

    // standard getters and setters
}
```

这里是我们简单的`Privilege`:

```
@Entity
public class Privilege {
    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    private Long id;

    @Column(nullable = false, unique = true)
    private String name;

    // standard getters and setters
}
```

还有我们的`Organization`:

```
@Entity
public class Organization {
    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    private Long id;

    @Column(nullable = false, unique = true)
    private String name;

    // standard setters and getters
}
```

最后，我们将使用一个更简单的定制`Principal`:

```
public class MyUserPrincipal implements UserDetails {

    private User user;

    public MyUserPrincipal(User user) {
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
        List<GrantedAuthority> authorities = new ArrayList<GrantedAuthority>();
        for (Privilege privilege : user.getPrivileges()) {
            authorities.add(new SimpleGrantedAuthority(privilege.getName()));
        }
        return authorities;
    }

    ...
}
```

准备好所有这些类后，我们将在一个基本的`UserDetailsService`实现中使用我们的自定义`Principal` :

```
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
        return new MyUserPrincipal(user);
    }
}
```

如您所见，这些关系并不复杂——用户有一个或多个特权，每个用户属于一个组织。

## 3。数据设置

接下来，让我们用简单的测试数据初始化我们的数据库:

```
@Component
public class SetupData {
    @Autowired
    private UserRepository userRepository;

    @Autowired
    private PrivilegeRepository privilegeRepository;

    @Autowired
    private OrganizationRepository organizationRepository;

    @PostConstruct
    public void init() {
        initPrivileges();
        initOrganizations();
        initUsers();
    }
}
```

下面是我们的`init` 方法:

```
private void initPrivileges() {
    Privilege privilege1 = new Privilege("FOO_READ_PRIVILEGE");
    privilegeRepository.save(privilege1);

    Privilege privilege2 = new Privilege("FOO_WRITE_PRIVILEGE");
    privilegeRepository.save(privilege2);
}
```

```
private void initOrganizations() {
    Organization org1 = new Organization("FirstOrg");
    organizationRepository.save(org1);

    Organization org2 = new Organization("SecondOrg");
    organizationRepository.save(org2);
}
```

```
private void initUsers() {
    Privilege privilege1 = privilegeRepository.findByName("FOO_READ_PRIVILEGE");
    Privilege privilege2 = privilegeRepository.findByName("FOO_WRITE_PRIVILEGE");

    User user1 = new User();
    user1.setUsername("john");
    user1.setPassword("123");
    user1.setPrivileges(new HashSet<Privilege>(Arrays.asList(privilege1)));
    user1.setOrganization(organizationRepository.findByName("FirstOrg"));
    userRepository.save(user1);

    User user2 = new User();
    user2.setUsername("tom");
    user2.setPassword("111");
    user2.setPrivileges(new HashSet<Privilege>(Arrays.asList(privilege1, privilege2)));
    user2.setOrganization(organizationRepository.findByName("SecondOrg"));
    userRepository.save(user2);
}
```

请注意:

*   用户“约翰”只有`FOO_READ_PRIVILEGE`
*   用户“汤姆”同时拥有`FOO_READ_PRIVILEGE`和`FOO_WRITE_PRIVILEGE`

## 4。自定义权限评估器

现在，我们已经准备好开始实现我们的新表达式了——通过一个新的自定义权限评估器。

我们将使用用户的特权来保护我们的方法——但是我们不使用硬编码的特权名称，而是希望实现一个更加开放、灵活的实现。

让我们开始吧。

### 4.1。`PermissionEvaluator`

为了创建我们自己的自定义权限评估器，我们需要实现`PermissionEvaluator`接口:

```
public class CustomPermissionEvaluator implements PermissionEvaluator {
    @Override
    public boolean hasPermission(
      Authentication auth, Object targetDomainObject, Object permission) {
        if ((auth == null) || (targetDomainObject == null) || !(permission instanceof String)){
            return false;
        }
        String targetType = targetDomainObject.getClass().getSimpleName().toUpperCase();

        return hasPrivilege(auth, targetType, permission.toString().toUpperCase());
    }

    @Override
    public boolean hasPermission(
      Authentication auth, Serializable targetId, String targetType, Object permission) {
        if ((auth == null) || (targetType == null) || !(permission instanceof String)) {
            return false;
        }
        return hasPrivilege(auth, targetType.toUpperCase(), 
          permission.toString().toUpperCase());
    }
}
```

下面是我们的`hasPrivilege()`方法:

```
private boolean hasPrivilege(Authentication auth, String targetType, String permission) {
    for (GrantedAuthority grantedAuth : auth.getAuthorities()) {
        if (grantedAuth.getAuthority().startsWith(targetType) && 
          grantedAuth.getAuthority().contains(permission)) {
            return true;
        }
    }
    return false;
}
```

**我们现在有了一个新的安全表达式，可以使用了:`hasPermission`。**

因此，不使用更硬编码的版本:

```
@PostAuthorize("hasAuthority('FOO_READ_PRIVILEGE')")
```

我们可以用 use:

```
@PostAuthorize("hasPermission(returnObject, 'read')")
```

或者

```
@PreAuthorize("hasPermission(#id, 'Foo', 'read')")
```

注意:`#id`表示方法参数，`Foo`表示目标对象类型。

### 4.2。方法安全配置

仅仅定义`CustomPermissionEvaluator`是不够的——我们还需要在我们的方法安全配置中使用它:

```
@Configuration
@EnableGlobalMethodSecurity(prePostEnabled = true)
public class MethodSecurityConfig extends GlobalMethodSecurityConfiguration {

    @Override
    protected MethodSecurityExpressionHandler createExpressionHandler() {
        DefaultMethodSecurityExpressionHandler expressionHandler = 
          new DefaultMethodSecurityExpressionHandler();
        expressionHandler.setPermissionEvaluator(new CustomPermissionEvaluator());
        return expressionHandler;
    }
}
```

### 4.3。实践中的例子

现在让我们开始使用新的表达式——在几个简单的控制器方法中:

```
@Controller
public class MainController {

    @PostAuthorize("hasPermission(returnObject, 'read')")
    @GetMapping("/foos/{id}")
    @ResponseBody
    public Foo findById(@PathVariable long id) {
        return new Foo("Sample");
    }

    @PreAuthorize("hasPermission(#foo, 'write')")
    @PostMapping("/foos")
    @ResponseStatus(HttpStatus.CREATED)
    @ResponseBody
    public Foo create(@RequestBody Foo foo) {
        return foo;
    }
}
```

好了，我们都准备好了，并在实践中使用新的表达方式。

### 4.4。现场测试

现在让我们编写一个简单的现场测试——点击 API 并确保一切正常:

```
@Test
public void givenUserWithReadPrivilegeAndHasPermission_whenGetFooById_thenOK() {
    Response response = givenAuth("john", "123").get("http://localhost:8082/foos/1");
    assertEquals(200, response.getStatusCode());
    assertTrue(response.asString().contains("id"));
}

@Test
public void givenUserWithNoWritePrivilegeAndHasPermission_whenPostFoo_thenForbidden() {
    Response response = givenAuth("john", "123").contentType(MediaType.APPLICATION_JSON_VALUE)
                                                .body(new Foo("sample"))
                                                .post("http://localhost:8082/foos");
    assertEquals(403, response.getStatusCode());
}

@Test
public void givenUserWithWritePrivilegeAndHasPermission_whenPostFoo_thenOk() {
    Response response = givenAuth("tom", "111").contentType(MediaType.APPLICATION_JSON_VALUE)
                                               .body(new Foo("sample"))
                                               .post("http://localhost:8082/foos");
    assertEquals(201, response.getStatusCode());
    assertTrue(response.asString().contains("id"));
}
```

这里是我们的`givenAuth()`方法:

```
private RequestSpecification givenAuth(String username, String password) {
    FormAuthConfig formAuthConfig = 
      new FormAuthConfig("http://localhost:8082/login", "username", "password");

    return RestAssured.given().auth().form(username, password, formAuthConfig);
}
```

## 5。一个新的安全表达式

使用之前的解决方案，我们能够定义和使用`hasPermission`表达式——这非常有用。

然而，我们在这里仍然受到表达式本身的名称和语义的限制。

因此，在这一节中，我们将进行完全自定义，我们将实现一个名为`isMember()`的安全表达式，检查主体是否是某个组织的成员。

### 5.1。自定义方法安全表达式

为了创建这个新的自定义表达式，我们需要从实现根注释开始，所有安全表达式的计算都从这里开始:

```
public class CustomMethodSecurityExpressionRoot 
  extends SecurityExpressionRoot implements MethodSecurityExpressionOperations {

    public CustomMethodSecurityExpressionRoot(Authentication authentication) {
        super(authentication);
    }

    public boolean isMember(Long OrganizationId) {
        User user = ((MyUserPrincipal) this.getPrincipal()).getUser();
        return user.getOrganization().getId().longValue() == OrganizationId.longValue();
    }

    ...
}
```

现在我们如何在这里的根节点中提供这个新的操作；`isMember()`用于检查当前用户是否是给定`Organization`中的成员。

还要注意我们是如何扩展`SecurityExpressionRoot`来包含内置表达式的。

### 5.2。自定义表达式处理器

接下来，我们需要在表达式处理程序中注入我们的`CustomMethodSecurityExpressionRoot`:

```
public class CustomMethodSecurityExpressionHandler 
  extends DefaultMethodSecurityExpressionHandler {
    private AuthenticationTrustResolver trustResolver = 
      new AuthenticationTrustResolverImpl();

    @Override
    protected MethodSecurityExpressionOperations createSecurityExpressionRoot(
      Authentication authentication, MethodInvocation invocation) {
        CustomMethodSecurityExpressionRoot root = 
          new CustomMethodSecurityExpressionRoot(authentication);
        root.setPermissionEvaluator(getPermissionEvaluator());
        root.setTrustResolver(this.trustResolver);
        root.setRoleHierarchy(getRoleHierarchy());
        return root;
    }
}
```

### 5.3。方法安全配置

现在，我们需要在方法安全配置中使用我们的`CustomMethodSecurityExpressionHandler`:

```
@Configuration
@EnableGlobalMethodSecurity(prePostEnabled = true)
public class MethodSecurityConfig extends GlobalMethodSecurityConfiguration {
    @Override
    protected MethodSecurityExpressionHandler createExpressionHandler() {
        CustomMethodSecurityExpressionHandler expressionHandler = 
          new CustomMethodSecurityExpressionHandler();
        expressionHandler.setPermissionEvaluator(new CustomPermissionEvaluator());
        return expressionHandler;
    }
}
```

### 5.4。使用新的表达式

下面是一个使用`isMember()`保护控制器方法的简单例子:

```
@PreAuthorize("isMember(#id)")
@GetMapping("/organizations/{id}")
@ResponseBody
public Organization findOrgById(@PathVariable long id) {
    return organizationRepository.findOne(id);
}
```

### 5.5。现场测试

最后，这里是对用户“`john`”的一个简单的现场测试:

```
@Test
public void givenUserMemberInOrganization_whenGetOrganization_thenOK() {
    Response response = givenAuth("john", "123").get("http://localhost:8082/organizations/1");
    assertEquals(200, response.getStatusCode());
    assertTrue(response.asString().contains("id"));
}

@Test
public void givenUserMemberNotInOrganization_whenGetOrganization_thenForbidden() {
    Response response = givenAuth("john", "123").get("http://localhost:8082/organizations/2");
    assertEquals(403, response.getStatusCode());
}
```

## 6。禁用内置安全表达式

最后，让我们看看如何覆盖内置的安全表达式——我们将讨论禁用`hasAuthority()`。

### 6.1。自定义安全表达式根

我们将从编写自己的`SecurityExpressionRoot`开始，主要是因为内置方法是`final`，所以我们不能覆盖它们:

```
public class MySecurityExpressionRoot implements MethodSecurityExpressionOperations {
    public MySecurityExpressionRoot(Authentication authentication) {
        if (authentication == null) {
            throw new IllegalArgumentException("Authentication object cannot be null");
        }
        this.authentication = authentication;
    }

    @Override
    public final boolean hasAuthority(String authority) {
        throw new RuntimeException("method hasAuthority() not allowed");
    }
    ...
}
```

在定义了这个根节点之后，我们必须将它注入到表达式处理程序中，然后将该处理程序连接到我们的配置中——就像我们在第 5 节中所做的那样。

### 6.2。示例–使用表达式

现在，如果我们想使用`hasAuthority()`来保护方法——如下所示，当我们试图访问方法时，它将抛出`RuntimeException`:

```
@PreAuthorize("hasAuthority('FOO_READ_PRIVILEGE')")
@GetMapping("/foos")
@ResponseBody
public Foo findFooByName(@RequestParam String name) {
    return new Foo(name);
}
```

### 6.3。现场测试

最后，这是我们的简单测试:

```
@Test
public void givenDisabledSecurityExpression_whenGetFooByName_thenError() {
    Response response = givenAuth("john", "123").get("http://localhost:8082/foos?name=sample");
    assertEquals(500, response.getStatusCode());
    assertTrue(response.asString().contains("method hasAuthority() not allowed"));
}
```

## 7。结论

在本指南中，我们深入探讨了在 Spring Security 中实现自定义安全表达式的各种方法，如果现有的方法还不够的话。

和往常一样，完整的源代码可以在 GitHub 上找到[。](https://web.archive.org/web/20220812054610/https://github.com/eugenp/tutorials/tree/master/spring-security-modules/spring-security-web-boot-1)