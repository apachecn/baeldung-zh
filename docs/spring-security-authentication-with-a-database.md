# Spring Security:使用数据库支持的 UserDetailsService 进行身份验证

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-security-authentication-with-a-database>

## 1。概述

在本文中，我们将展示如何创建一个定制的数据库支持的`UserDetailsService`来使用 Spring Security 进行身份验证。

## 2。`UserDetailsService`

`UserDetailsService`接口用于检索用户相关数据。它有一个名为`loadUserByUsername()`的方法，可以被覆盖来定制查找用户的过程。

在认证期间，`DaoAuthenticationProvider`使用它来加载关于用户的详细信息。

## 3。`User`模型

为了存储用户，我们将创建一个映射到数据库表的`User`实体，它具有以下属性:

```java
@Entity
public class User {

    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    private Long id;

    @Column(nullable = false, unique = true)
    private String username;

    private String password;

    // standard getters and setters
}
```

## 4。检索用户

为了检索与用户名相关联的用户，我们将通过扩展`JpaRepository`接口使用`Spring Data`创建一个`DAO`类:

```java
public interface UserRepository extends JpaRepository<User, Long> {

    User findByUsername(String username);
}
```

## 5。`UserDetailsService`

为了提供我们自己的用户服务，我们需要实现`UserDetailsService`接口。

我们将创建一个名为`MyUserDetailsService`的类来覆盖接口的方法`loadUserByUsername()`。

在这个方法中，我们使用`DAO`检索`User`对象，如果它存在，将它包装成一个 *MyUserPrincipal* 对象，它实现了`UserDetails`，并返回它:

```java
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

让我们如下定义 `MyUserPrincipal`类:

```java
public class MyUserPrincipal implements UserDetails {
    private User user;

    public MyUserPrincipal(User user) {
        this.user = user;
    }
    //...
}
```

## 6。弹簧配置

我们将演示两种类型的 Spring 配置:XML 和基于注释的，这是使用我们的定制`UserDetailsService`实现所必需的。

### 6.1。注释配置

**要启用我们的自定义`UserDetailsService `，我们需要做的就是将它作为 bean 添加到我们的应用程序上下文中。**

因为我们用`@Service` 注释配置了我们的类，所以应用程序将在组件扫描期间自动检测它，并且它将从这个类中创建一个 bean。因此，我们在这里不需要做任何其他事情。

或者，我们可以:

*   使用`AuthenticationManagerBuilder#userDetailsService` 方法在`authenticationManager` 中进行配置
*   将其设置为自定义`authenticationProvider` bean 中的一个属性，然后使用`AuthenticationManagerBuilder# authenticationProvider`函数注入该属性

### 6.2。XML 配置

另一方面，对于 XML 配置，我们需要定义一个类型为`MyUserDetailsService`的 bean，并将其注入到 Spring 的`authentication-provider` bean 中:

```java
<bean id="myUserDetailsService" 
  class="org.baeldung.security.MyUserDetailsService"/>

<security:authentication-manager>
    <security:authentication-provider 
      user-service-ref="myUserDetailsService" >
        <security:password-encoder ref="passwordEncoder">
        </security:password-encoder>
    </security:authentication-provider>
</security:authentication-manager>

<bean id="passwordEncoder" 
  class="org.springframework.security
  .crypto.bcrypt.BCryptPasswordEncoder">
    <constructor-arg value="11"/>
</bean>
```

## 7。其他数据库支持的认证选项

`AuthenticationManagerBuilder `提供了在我们的应用程序中配置基于 JDBC 的认证的另一种方法。

我们必须用一个`DataSource`实例来配置 `AuthenticationManagerBuilder.jdbcAuthentication` 。如果我们的数据库遵循 [Spring 用户模式](https://web.archive.org/web/20220630141413/https://docs.spring.io/spring-security/reference/servlet/appendix/database-schema.html#_user_schema)，那么默认配置将非常适合我们。

在之前的文章中，我们已经看到了使用这种方法的基本配置。

由此配置产生的`JdbcUserDetailsManager `实体也实现了`UserDetailsService `。

因此，我们可以得出结论，这种配置更容易实现，特别是如果我们使用自动为我们配置 [`DataSource`](/web/20220630141413/https://www.baeldung.com/spring-boot-configure-data-source-programmatic) 的 Spring Boot。

无论如何，如果我们需要更高层次的灵活性，精确定制应用程序获取用户详细信息的方式，那么我们将选择我们在本教程中遵循的方法。

## 8。结论

总之，在本文中，我们已经展示了如何创建一个基于 Spring 的、由持久数据支持的定制`UserDetailsService`。

这个实现可以在 GitHub 项目中找到——这是一个基于 Maven 的项目，所以应该很容易导入和运行。