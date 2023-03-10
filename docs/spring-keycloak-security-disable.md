# 在 Spring Boot 禁用钥匙锁安全

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-keycloak-security-disable>

## 1.概观

Keycloak 是一个免费的开源身份和访问管理程序，目前经常在我们的软件栈中使用。在测试阶段，禁用它来专注于业务测试可能是有用的。我们的测试环境中可能也没有 Keycloak 服务器。

在本教程中，**我们将禁用由 Keycloak starter** 设置的配置。当 Spring 安全性在我们的项目中启用时，我们还将研究如何修改它。

## 2.在非 Spring 安全环境中禁用 Keycloak

我们将从如何在不使用 Spring Security 的应用程序中禁用 Keycloak 开始。

### 2.1.应用程序设置

让我们从添加`[keycloak-spring-boot-starter](https://web.archive.org/web/20220815155800/https://search.maven.org/search?q=keycloak-spring-boot-starter)`依赖项到我们的项目开始:

```java
<dependency>
    <groupId>org.keycloak</groupId>
    <artifactId>keycloak-spring-boot-starter</artifactId>
</dependency>
```

此外，我们需要添加由`[keycloak-adapter-bom](https://web.archive.org/web/20220815155800/https://search.maven.org/search?q=keycloak-adapter-bom)`依赖项带来的各种嵌入式容器的依赖项:

```java
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.keycloak.bom</groupId>
            <artifactId>keycloak-adapter-bom</artifactId>
            <version>15.0.2</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
```

接下来，我们将在`application.properties`中添加 Keycloak 服务器的配置:

```java
keycloak.auth-server-url=http://localhost:8180/auth
keycloak.realm=SpringBootKeycloak
keycloak.resource=login-app
keycloak.public-client=true
keycloak.security-constraints[0].authRoles[0]=user
keycloak.security-constraints[0].securityCollections[0].patterns[0]=/users/*
```

**这种配置确保了对`/users` URL 的请求只能被具有`user`角色**的认证用户访问。

最后，让我们添加一个检索`User`的`UserController`:

```java
@RestController
@RequestMapping("/users")
public class UserController {
    @GetMapping("/{userId}")
    public User getCustomer(@PathVariable Long userId) {
        return new User(userId, "John", "Doe");
    }
} 
```

### 2.2.禁用键盘锁

现在我们的应用程序已经就绪，让我们编写一个简单的测试来获得用户:

```java
@Test
public void givenUnauthenticated_whenGettingUser_shouldReturnUser() {
    ResponseEntity<User> responseEntity = restTemplate.getForEntity("/users/1", User.class);

    assertEquals(HttpStatus.SC_OK, responseEntity.getStatusCodeValue());
    assertNotNull(responseEntity.getBody()
        .getFirstname());
}
```

**该测试将失败，因为我们没有向`restTemplate`** 提供任何认证，或者因为 Keycloak 服务器不可用。

键盘锁适配器实现了键盘锁安全的 [Spring 自动配置](https://web.archive.org/web/20220815155800/https://docs.spring.io/spring-boot/docs/current/reference/html/using.html#using.auto-configuration)。自动配置依赖于类路径中类的存在或属性值。具体来说，`@ConditionalOnProperty`注释对于这种特殊需求非常方便。

**要禁用 Keycloak 安全性，我们需要通知适配器不要加载相应的配置**。我们可以通过如下方式分配属性来实现这一点:

```java
keycloak.enabled=false
```

如果我们再次启动我们的测试，它现在将成功，不需要任何身份验证。

## 3.在 Spring 安全环境中禁用 Keycloak

我们经常将 [Keycloak 与 Spring Security](/web/20220815155800/https://www.baeldung.com/spring-boot-keycloak#springsecurity) 结合使用。**在这种情况下，禁用 Keycloak 配置是不够的，我们还需要修改 Spring 安全配置**以允许匿名请求到达控制器。

### 3.1.应用程序设置

让我们从添加[spring-boot-starter-security](https://web.archive.org/web/20220815155800/https://search.maven.org/search?q=g:org.springframework.boot%20AND%20a:spring-boot-starter-security)依赖项到我们的项目开始:

```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency> 
```

接下来，我们实现`WebSecurityConfigurerAdapter`来定义 Spring 安全性所需的配置。Keycloak 适配器为此提供了一个抽象类和注释:

```java
@KeycloakConfiguration
public class KeycloakSecurityConfig extends KeycloakWebSecurityConfigurerAdapter {

    @Autowired
    public void configureGlobal(AuthenticationManagerBuilder auth) {
        auth.authenticationProvider(keycloakAuthenticationProvider());
    }

    @Bean
    @Override
    protected SessionAuthenticationStrategy sessionAuthenticationStrategy() {
        return new NullAuthenticatedSessionStrategy();
    }

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        super.configure(http);

        http.csrf()
            .disable()
            .authorizeRequests()
            .anyRequest()
            .authenticated();
    }
}
```

这里，我们将 Spring Security 配置为只允许来自经过身份验证的用户的请求。

### 3.2.禁用键盘锁

除了像我们之前做的那样禁用 Keycloak 之外，**我们现在还需要禁用 Spring Security** 。

我们可以使用[配置文件](/web/20220815155800/https://www.baeldung.com/spring-profiles)来告诉 Spring 在测试期间是否激活 Keycloak 配置:

```java
@KeycloakConfiguration
@Profile("tests")
public class KeycloakSecurityConfig extends KeycloakWebSecurityConfigurerAdapter {
    // ...
}
```

**然而，一种更优雅的方式是重用`keycloak.enable`属性**，类似于键锁适配器:

```java
@KeycloakConfiguration
@ConditionalOnProperty(name = "keycloak.enabled", havingValue = "true", matchIfMissing = true)
public class KeycloakSecurityConfig extends KeycloakWebSecurityConfigurerAdapter {
    // ...
}
```

因此，只有当`keycloak.enable`属性为`true`时，Spring 才会启用 Keycloak 配置。如果属性丢失，`matchIfMissing`默认启用。

因为我们使用的是 Spring Security starter，所以禁用我们的 Spring 安全配置是不够的。事实上，遵循 Spring 固执己见的默认配置原则，**starter 将创建一个默认安全层**。

让我们创建一个配置类来禁用它:

```java
@Configuration
@ConditionalOnProperty(name = "keycloak.enabled", havingValue = "false")
public class DisableSecurityConfiguration extends WebSecurityConfigurerAdapter {

    @Override
    protected void configure(final HttpSecurity http) throws Exception {
        http.csrf()
            .disable()
            .authorizeRequests()
            .anyRequest()
            .permitAll();
    }
}
```

我们仍然使用我们的`keycloak.enable`属性，但是这次 **Spring 启用了配置，如果它的值被设置为`false`** 。

## 4.结论

在本文中，我们研究了如何在 Spring 环境中禁用 Keycloak 安全性，不管有没有 Spring 安全性。

像往常一样，本文中使用的所有代码示例都可以在 GitHub 上找到[。](https://web.archive.org/web/20220815155800/https://github.com/eugenp/tutorials/tree/master/spring-boot-modules/spring-boot-keycloak-2)