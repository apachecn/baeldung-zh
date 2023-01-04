# Spring 方法安全性介绍

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-security-method-security>

## 1。概述

简单地说，Spring Security 支持方法级的授权语义。

通常，我们可以通过限制哪些角色能够执行特定的方法来保护我们的服务层，并使用专用的方法级安全测试支持来测试它。

在本教程中，我们将回顾一些安全注释的使用。然后，我们将重点用不同的策略测试我们的方法安全性。

## 延伸阅读:

## [春天表情语言指南](/web/20221204215749/https://www.baeldung.com/spring-expression-language)

This article explores Spring Expression Language (SpEL), a powerful expression language that supports querying and manipulating object graphs at runtime.[Read more](/web/20221204215749/https://www.baeldung.com/spring-expression-language) →

## [带有 Spring 安全的自定义安全表达式](/web/20221204215749/https://www.baeldung.com/spring-security-create-new-custom-security-expression)

A guide to creating a new, custom security expression with Spring Security, and then using the new expression with the Pre and Post authorize annotations.[Read more](/web/20221204215749/https://www.baeldung.com/spring-security-create-new-custom-security-expression) →

## 2。启用方法安全性

首先，要使用 Spring 方法安全性，我们需要添加`spring-security-config`依赖项:

```
<dependency>
    <groupId>org.springframework.security</groupId>
    <artifactId>spring-security-config</artifactId>
</dependency>
```

我们可以在 [Maven Central](https://web.archive.org/web/20221204215749/https://search.maven.org/classic/#search%7Cga%7C1%7C%22spring-security-config%22) 上找到它的最新版本。

如果我们想使用 Spring Boot，我们可以使用`spring-boot-starter-security`依赖项，它包括`spring-security-config`:

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>
```

同样，最新版本可以在 [Maven Central](https://web.archive.org/web/20221204215749/https://search.maven.org/classic/#search%7Cga%7C1%7C%22spring-boot-starter-security%22) 上找到。

**接下来，我们需要启用全局方法安全性**:

```
@Configuration
@EnableGlobalMethodSecurity(
  prePostEnabled = true, 
  securedEnabled = true, 
  jsr250Enabled = true)
public class MethodSecurityConfig 
  extends GlobalMethodSecurityConfiguration {
}
```

*   `prePostEnabled`属性启用 Spring 安全前/后注释。
*   `securedEnabled`属性确定是否应该启用`@Secured`注释。
*   属性允许我们使用注释。

我们将在下一节探讨更多关于这些注释的内容。

## 3。应用方法安全性

### 3.1。使用`@Secured`标注

**`@Secured`注释用于指定一个方法上的角色列表。**因此，一个用户只有至少拥有一个指定的角色才能访问这个方法。

让我们定义一个`getUsername`方法:

```
@Secured("ROLE_VIEWER")
public String getUsername() {
    SecurityContext securityContext = SecurityContextHolder.getContext();
    return securityContext.getAuthentication().getName();
}
```

这里的`@Secured(“ROLE_VIEWER”)`注释定义了只有角色为`ROLE_VIEWER`的用户才能执行`getUsername`方法。

此外，我们可以在一个`@Secured`注释中定义一个角色列表:

```
@Secured({ "ROLE_VIEWER", "ROLE_EDITOR" })
public boolean isValidUsername(String username) {
    return userRoleRepository.isValidUsername(username);
}
```

在这种情况下，配置声明如果用户拥有`ROLE_VIEWER` 或`ROLE_EDITOR`，该用户可以调用`isValidUsername`方法。

**`@Secured`批注不支持 Spring 表达式语言(SpEL)。**

### 3.2。使用`@RolesAllowed`标注

**`@RolesAllowed` 标注是`@Secured` 标注的 JSR-250 等效标注。**

基本上，我们可以像使用`@Secured`一样使用`@RolesAllowed`注释。

这样，我们可以重新定义`getUsername`和`isValidUsername`方法:

```
@RolesAllowed("ROLE_VIEWER")
public String getUsername2() {
    //...
}

@RolesAllowed({ "ROLE_VIEWER", "ROLE_EDITOR" })
public boolean isValidUsername2(String username) {
    //...
}
```

同样，只有拥有角色`ROLE_VIEWER`的用户才能执行`getUsername2`。

同样，只有当用户拥有至少一个`ROLE_VIEWER` 或`ROLER_EDITOR`角色时，她才能够调用`isValidUsername2`。

### 3.3。使用`@PreAuthorize`和`@PostAuthorize`注释

**`@PreAuthorize`和`@PostAuthorize`注释都提供了基于表达式的访问控制。**因此，可以使用[SpEL(Spring Expression Language)](/web/20221204215749/https://www.baeldung.com/spring-expression-language)编写谓词。

**`@PreAuthorize`注释在进入方法**之前检查给定的表达式，而**`@PostAuthorize`注释在方法执行之后验证它，并且可以改变结果。**

现在让我们声明一个`getUsernameInUpperCase`方法如下:

```
@PreAuthorize("hasRole('ROLE_VIEWER')")
public String getUsernameInUpperCase() {
    return getUsername().toUpperCase();
}
```

`@PreAuthorize(“hasRole(‘ROLE_VIEWER')”)` 与我们在上一节中使用的`@Secured(“ROLE_VIEWER”)`含义相同。在以前的文章中，您可以随意发现更多[安全表达细节。](/web/20221204215749/https://www.baeldung.com/spring-security-expressions)

因此，注释`@Secured({“ROLE_VIEWER”,”ROLE_EDITOR”})` 可以替换为`@PreAuthorize(“hasRole(‘ROLE_VIEWER')`或`hasRole(‘ROLE_EDITOR')”)`:

```
@PreAuthorize("hasRole('ROLE_VIEWER') or hasRole('ROLE_EDITOR')")
public boolean isValidUsername3(String username) {
    //...
}
```

此外，**我们实际上可以使用方法参数作为表达式**的一部分:

```
@PreAuthorize("#username == authentication.principal.username")
public String getMyRoles(String username) {
    //...
}
```

这里，只有当参数`username`的值与当前主体的用户名相同时，用户才能调用`getMyRoles`方法。

**值得注意的是`@PreAuthorize`表情可以用`@PostAuthorize`表情代替。**

让我们重写一下`getMyRoles`:

```
@PostAuthorize("#username == authentication.principal.username")
public String getMyRoles2(String username) {
    //...
}
```

然而，在前面的例子中，授权会在目标方法执行之后被延迟。

另外，**`@PostAuthorize`注释提供了访问方法结果**的能力:

```
@PostAuthorize
  ("returnObject.username == authentication.principal.nickName")
public CustomUser loadUserDetail(String username) {
    return userRoleRepository.loadUserByUserName(username);
}
```

这里，`loadUserDetail`方法只有在返回的`CustomUser`的`username`等于当前认证主体的`nickname`时才会成功执行。

在本节中，我们主要使用简单的弹簧表达式。对于更复杂的场景，我们可以创建[自定义安全表达式](/web/20221204215749/https://www.baeldung.com/spring-security-create-new-custom-security-expression)。

### 3.4。使用`@PreFilter`和`@PostFilter`注释

**Spring Security 提供了`@PreFilter`注释，在执行**方法之前过滤集合参数:

```
@PreFilter("filterObject != authentication.principal.username")
public String joinUsernames(List<String> usernames) {
    return usernames.stream().collect(Collectors.joining(";"));
}
```

在这个例子中，我们加入了所有的用户名，除了经过身份验证的用户名。

这里，**在我们的表达式中，我们使用名称`filterObject`来表示集合中的当前对象。**

然而，如果该方法有多个集合类型的参数，我们需要使用`filterTarget`属性来指定我们想要过滤的参数:

```
@PreFilter
  (value = "filterObject != authentication.principal.username",
  filterTarget = "usernames")
public String joinUsernamesAndRoles(
  List<String> usernames, List<String> roles) {

    return usernames.stream().collect(Collectors.joining(";")) 
      + ":" + roles.stream().collect(Collectors.joining(";"));
}
```

另外，**我们还可以通过使用`@PostFilter`注释**来过滤一个方法返回的集合:

```
@PostFilter("filterObject != authentication.principal.username")
public List<String> getAllUsernamesExceptCurrent() {
    return userRoleRepository.getAllUsernames();
}
```

在这种情况下，名称`filterObject`指的是返回集合中的当前对象。

有了这个配置，Spring Security 将遍历返回的列表，删除任何与主体用户名匹配的值。

我们的[Spring Security—@ pre filter 和@PostFilter](/web/20221204215749/https://www.baeldung.com/spring-security-prefilter-postfilter) 文章更详细地描述了这两种注释。

### 3.5。方法安全元注释

我们通常会发现自己处于这样一种情况:我们使用相同的安全配置来保护不同的方法。

在这种情况下，我们可以定义一个安全元注释:

```
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
@PreAuthorize("hasRole('VIEWER')")
public @interface IsViewer {
}
```

接下来，我们可以直接使用@IsViewer 注释来保护我们的方法:

```
@IsViewer
public String getUsername4() {
    //...
}
```

安全元注释是一个好主意，因为它们增加了更多的语义，并将我们的业务逻辑从安全框架中分离出来。

### 3.6。类级别的安全注释

如果我们发现自己对一个类中的每个方法使用相同的安全注释，我们可以考虑将该注释放在类级别:

```
@Service
@PreAuthorize("hasRole('ROLE_ADMIN')")
public class SystemService {

    public String getSystemYear(){
        //...
    }

    public String getSystemDate(){
        //...
    }
}
```

在上面的例子中，安全规则`hasRole(‘ROLE_ADMIN')`将同时应用于`getSystemYear`和`getSystemDate`方法。

### 3.7。一个方法上的多个安全注释

我们还可以在一个方法上使用多个安全注释:

```
@PreAuthorize("#username == authentication.principal.username")
@PostAuthorize("returnObject.username == authentication.principal.nickName")
public CustomUser securedLoadUserDetail(String username) {
    return userRoleRepository.loadUserByUserName(username);
}
```

这样，Spring 将在执行`securedLoadUserDetail`方法之前和之后验证授权。

## 4。重要注意事项

关于方法安全性，我们要记住两点:

*   默认情况下，Spring AOP 代理用于应用方法安全性。如果一个安全的方法 A 被同一个类中的另一个方法调用，那么 A 中的安全性将被完全忽略。这意味着方法 A 将在没有任何安全检查的情况下执行。这同样适用于私有方法。
*   **弹簧`SecurityContext`被线捆住。**默认情况下，安全上下文不会传播到子线程。更多信息，请参考我们的 [Spring 安全上下文传播](/web/20221204215749/https://www.baeldung.com/spring-security-async-principal-propagation)文章。

## 5。测试方法安全性

### 5.1。配置

**为了测试 JUnit 的 Spring 安全性，我们需要`spring-security-test`依赖项**:

```
<dependency>
    <groupId>org.springframework.security</groupId>
    <artifactId>spring-security-test</artifactId>
</dependency>
```

我们不需要指定依赖版本，因为我们正在使用 Spring Boot 插件。我们可以在 [Maven Central](https://web.archive.org/web/20221204215749/https://search.maven.org/classic/#search%7Cga%7C1%7C%22spring-security-test%22) 上找到这种依赖的最新版本。

接下来，让我们通过指定转轮和`ApplicationContext` 配置来配置一个简单的弹簧集成测试:

```
@RunWith(SpringRunner.class)
@ContextConfiguration
public class MethodSecurityIntegrationTest {
    // ...
}
```

### 5.2。测试用户名和角色

现在我们的配置已经准备好了，让我们试着测试我们用`@Secured(“ROLE_VIEWER”)` 注释保护的`getUsername` 方法:

```
@Secured("ROLE_VIEWER")
public String getUsername() {
    SecurityContext securityContext = SecurityContextHolder.getContext();
    return securityContext.getAuthentication().getName();
}
```

因为我们在这里使用了`@Secured` 注释，所以它要求用户通过身份验证才能调用该方法。否则我们会得到一个`AuthenticationCredentialsNotFoundException`。

因此，**我们需要提供一个用户来测试我们的安全方法。**

**为了实现这一点，我们用`@WithMockUser` 修饰测试方法，并提供一个用户和角色**:

```
@Test
@WithMockUser(username = "john", roles = { "VIEWER" })
public void givenRoleViewer_whenCallGetUsername_thenReturnUsername() {
    String userName = userRoleService.getUsername();

    assertEquals("john", userName);
}
```

我们提供了一个经过身份验证的用户，其用户名为`john`，角色为`ROLE_VIEWER`。如果我们不指定`username`或者`role`，默认`username`是`user`，默认`role`是`ROLE_USER`。

**注意，没有必要在这里添加前缀`ROLE_`，因为 Spring Security 会自动添加该前缀。**

如果不想要那个前缀，可以考虑用`authority`代替`role`。

例如，让我们声明一个`getUsernameInLowerCase`方法:

```
@PreAuthorize("hasAuthority('SYS_ADMIN')")
public String getUsernameLC(){
    return getUsername().toLowerCase();
}
```

我们可以利用权威来测试:

```
@Test
@WithMockUser(username = "JOHN", authorities = { "SYS_ADMIN" })
public void givenAuthoritySysAdmin_whenCallGetUsernameLC_thenReturnUsername() {
    String username = userRoleService.getUsernameInLowerCase();

    assertEquals("john", username);
}
```

为了方便起见，**如果我们想要为许多测试用例使用同一个用户，我们可以在测试类**中声明`@WithMockUser`注释:

```
@RunWith(SpringRunner.class)
@ContextConfiguration
@WithMockUser(username = "john", roles = { "VIEWER" })
public class MockUserAtClassLevelIntegrationTest {
    //...
}
```

**如果我们想以匿名用户的身份运行我们的测试，我们可以使用`@WithAnonymousUser`注释**:

```
@Test(expected = AccessDeniedException.class)
@WithAnonymousUser
public void givenAnomynousUser_whenCallGetUsername_thenAccessDenied() {
    userRoleService.getUsername();
}
```

在上面的例子中，我们期待一个 *AccessDeniedException* ，因为匿名用户没有被授予角色`ROLE_VIEWER`或权限`SYS_ADMIN`。

### 5.3。用自定义`UserDetailsService`测试

**对于大多数应用程序，通常使用一个自定义类作为身份验证主体。**在这种情况下，自定义类需要实现`org.springframework.security.core.userdetails.` `UserDetails`接口。

在本文中，我们声明了一个`CustomUser`类，它扩展了`UserDetails`的现有实现，即`org.springframework.security.core.userdetails.` `User`:

```
public class CustomUser extends User {
    private String nickName;
    // getter and setter
}
```

让我们回头看看第 3 节中带有`@PostAuthorize`注释的例子:

```
@PostAuthorize("returnObject.username == authentication.principal.nickName")
public CustomUser loadUserDetail(String username) {
    return userRoleRepository.loadUserByUserName(username);
}
```

在这种情况下，只有当返回的`CustomUser`的`username`等于当前认证主体的`nickname`时，该方法才会成功执行。

如果我们想测试这个方法，**，我们可以提供一个`UserDetailsService`的实现，它可以根据用户名**加载我们的`CustomUser`:

```
@Test
@WithUserDetails(
  value = "john", 
  userDetailsServiceBeanName = "userDetailService")
public void whenJohn_callLoadUserDetail_thenOK() {

    CustomUser user = userService.loadUserDetail("jane");

    assertEquals("jane", user.getNickName());
}
```

这里的`@WithUserDetails`注释声明我们将使用一个`UserDetailsService`来初始化我们的认证用户。服务由`userDetailsServiceBeanName` 属性`.` 引用，这个`UserDetailsService` 可能是一个真实的实现，也可能是一个测试用的假实现。

此外，服务将使用属性`value`的值作为用户名来加载`UserDetails`。

方便的是，我们也可以在类级别用一个`@WithUserDetails` 注释来装饰，类似于我们用`@WithMockUser` 注释`.`所做的

### 5.4。使用元注释进行测试

我们经常发现自己在各种测试中一遍又一遍地重用相同的用户/角色。

对于这些情况，创建一个`meta-annotation`很方便。

再看前面的例子`@WithMockUser(username=”john”, roles={“VIEWER”})`，我们可以声明一个元注释:

```
@Retention(RetentionPolicy.RUNTIME)
@WithMockUser(value = "john", roles = "VIEWER")
public @interface WithMockJohnViewer { }
```

那么我们可以在测试中简单地使用`@WithMockJohnViewer`:

```
@Test
@WithMockJohnViewer
public void givenMockedJohnViewer_whenCallGetUsername_thenReturnUsername() {
    String userName = userRoleService.getUsername();

    assertEquals("john", userName);
}
```

同样，我们可以使用元注释通过`@WithUserDetails`创建特定于领域的用户。

## 6。结论

在本文中，我们探索了在 Spring 安全性中使用方法安全性的各种选项。

我们还学习了一些技术来轻松测试方法安全性，并学习了如何在不同的测试中重用被模仿的用户。

这篇文章的所有例子都可以在 GitHub 上找到[。](https://web.archive.org/web/20221204215749/https://github.com/eugenp/tutorials/tree/master/spring-security-modules/spring-security-core)