# 春季安全活动

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/activiti-spring-security>

## 1。概述

Activiti 是一个开源的 BPM(业务流程管理)系统。关于介绍，请查看我们的[Java Activiti 指南](/web/20220627084230/https://www.baeldung.com/java-activiti)。

Activiti 和 Spring 框架都提供了自己的身份管理。然而，在集成了这两个项目的应用程序中，我们可能希望将这两个项目合并成一个用户管理流程。

在下文中，我们将探讨实现这一点的两种可能性:一种是通过为 Spring Security 提供 Activiti 支持的用户服务，另一种是通过将 Spring Security 用户源插入 Activiti 身份管理。

## 2。Maven 依赖关系

要在 Spring Boot 项目中设置 Activiti，请查看我们之前的文章。除了`activiti-spring-boot-starter-basic,`我们还需要[activiti-spring-boot-starter-security](https://web.archive.org/web/20220627084230/https://search.maven.org/classic/#search%7Cga%7C1%7Ca%3A%22activiti-spring-boot-starter-security%22)依赖项:

```java
<dependency>
    <groupId>org.activiti</groupId>
    <artifactId>activiti-spring-boot-starter-security</artifactId>
    <version>6.0.0</version>
</dependency>
```

## 3。使用 Activiti 进行身份管理

对于这个场景，Activiti 启动器提供了一个 Spring Boot 自动配置类，它通过`HTTP Basic`认证保护所有 REST 端点。

**自动配置还创建了一个`IdentityServiceUserDetailsService.`** 类的`UserDetailsService` bean

该类实现了 Spring 接口`UserDetailsService`并覆盖了`loadUserByUsername()`方法。这个方法用给定的`id`检索一个 Activiti `User`对象，并使用它创建一个 Spring `UserDetails`对象。

此外，Activiti `Group`对象对应于一个 Spring 用户角色。

这意味着当我们登录到 Spring 安全应用程序时，我们将使用 Activiti 凭证。

### 3.1。设置 Activiti 用户

首先，让我们使用`IdentityService:`在主`@SpringBootApplication`类中定义的`InitializingBean`中创建一个用户

```java
@Bean
InitializingBean usersAndGroupsInitializer(IdentityService identityService) {
    return new InitializingBean() {
        public void afterPropertiesSet() throws Exception {
            User user = identityService.newUser("activiti_user");
            user.setPassword("pass");
            identityService.saveUser(user);

            Group group = identityService.newGroup("user");
            group.setName("ROLE_USER");
            group.setType("USER");
            identityService.saveGroup(group);
            identityService.createMembership(user.getId(), group.getId());
        }
    };
}
```

你会注意到**因为它将被 Spring Security 使用，`Group`对象`name`必须是`“ROLE_X”`** `.`的形式

### 3.2。Spring 安全配置

如果我们想使用不同的安全配置来代替 HTTP 基本身份验证，首先我们必须排除自动配置:

```java
@SpringBootApplication(
  exclude = org.activiti.spring.boot.SecurityAutoConfiguration.class)
public class ActivitiSpringSecurityApplication {
    // ...
}
```

然后，我们可以提供自己的 Spring 安全配置类，它使用`IdentityServiceUserDetailsService` 从 Activiti 数据源中检索用户:

```java
@Configuration
public class SecurityConfig extends WebSecurityConfigurerAdapter {

    @Autowired
    private IdentityService identityService;

    @Autowired
    public void configureGlobal(AuthenticationManagerBuilder auth)
      throws Exception {

        auth.userDetailsService(userDetailsService());
    }

    @Bean
    public UserDetailsService userDetailsService() {
        return new IdentityServiceUserDetailsService(
          this.identityService);
    }

    // spring security configuration
}
```

## 4。使用 Spring 安全的身份管理

如果我们已经用 Spring Security 设置了用户管理，并且希望将 Activiti 添加到我们的应用程序中，那么我们需要定制 Activiti 的身份管理。

为此，我们必须扩展两个主要的类:`UserEntityManagerImpl`和`GroupEntityManagerImpl`，它们处理用户和组。

让我们更详细地看一下其中的每一项。

### 4.1。`UserEntityManagerImpl`延伸

让我们创建自己的类来扩展`UserEntityManagerImpl`类:

```java
public class SpringSecurityUserManager extends UserEntityManagerImpl {

    private JdbcUserDetailsManager userManager;

    public SpringSecurityUserManager(
      ProcessEngineConfigurationImpl processEngineConfiguration, 
      UserDataManager userDataManager, 
      JdbcUserDetailsManager userManager) {

        super(processEngineConfiguration, userDataManager);
        this.userManager = userManager;
    }

    // ...
}
```

这个类需要一个以上形式的构造函数，以及 Spring Security 用户管理器。在我们的例子中，我们使用了数据库支持的`UserDetailsManager.`

**我们想要覆盖的主要方法是那些处理用户检索的方法:`findById(),` `findUserByQueryCriteria()`和`findGroupsByUser().`**

`findById()`方法使用`JdbcUserDetailsManager`找到一个`UserDetails`对象并将其转换成一个`User`对象:

```java
@Override
public UserEntity findById(String userId) {
    UserDetails userDetails = userManager.loadUserByUsername(userId);
    if (userDetails != null) {
        UserEntityImpl user = new UserEntityImpl();
        user.setId(userId);
        return user;
    }
    return null;
}
```

接下来，`findGroupsByUser()`方法找到一个用户的所有 Spring 安全权限，并返回一个`Group`对象的`List`:

```java
public List<Group> findGroupsByUser(String userId) {
    UserDetails userDetails = userManager.loadUserByUsername(userId);
    if (userDetails != null) {
        return userDetails.getAuthorities().stream()
          .map(a -> {
            Group g = new GroupEntityImpl();
            g.setId(a.getAuthority());
            return g;
          })
          .collect(Collectors.toList());
    }
    return null;
}
```

`findUserByQueryCriteria()`方法基于一个具有多个属性的`UserQueryImpl`对象，我们将从中提取组 id 和用户 id，因为它们在 Spring Security 中有对应关系:

```java
@Override
public List<User> findUserByQueryCriteria(
  UserQueryImpl query, Page page) {
    // ...
}
```

这个方法遵循与上面相似的原则，通过从`UserDetails`对象创建`User`对象。完整的实现见最后的 GitHub 链接。

类似地，我们得到了`findUserCountByQueryCriteria()`方法:

```java
public long findUserCountByQueryCriteria(
  UserQueryImpl query) {

    return findUserByQueryCriteria(query, null).size();
}
```

由于密码验证不是由 Activiti:

```java
@Override
public Boolean checkPassword(String userId, String password) {
    return true;
}
```

对于其他方法，比如处理更新用户的方法，我们将抛出一个异常，因为这是由 Spring Security 处理的:

```java
public User createNewUser(String userId) {
    throw new UnsupportedOperationException("This operation is not supported!");
}
```

### 4.2。`GroupEntityManagerImpl`延伸

`SpringSecurityGroupManager`类似于用户管理器类，除了它处理用户组:

```java
public class SpringSecurityGroupManager extends GroupEntityManagerImpl {

    private JdbcUserDetailsManager userManager;

    public SpringSecurityGroupManager(ProcessEngineConfigurationImpl 
      processEngineConfiguration, GroupDataManager groupDataManager) {
        super(processEngineConfiguration, groupDataManager);
    }

    // ...
}
```

这里**覆盖的主要方法是`findGroupsByUser()`方法:**

```java
@Override
public List<Group> findGroupsByUser(String userId) {
    UserDetails userDetails = userManager.loadUserByUsername(userId);
    if (userDetails != null) {
        return userDetails.getAuthorities().stream()
          .map(a -> {
            Group g = new GroupEntityImpl();
            g.setId(a.getAuthority());
            return g;
          })
          .collect(Collectors.toList());
    }
    return null;
}
```

该方法检索 Spring Security 用户的权限，并将它们转换成一列`Group`对象。

基于此，我们也可以重写`findGroupByQueryCriteria()`和`findGroupByQueryCriteriaCount()`方法:

```java
@Override
public List<Group> findGroupByQueryCriteria(GroupQueryImpl query, Page page) {
    if (query.getUserId() != null) {
        return findGroupsByUser(query.getUserId());
    }
    return null;
}

@Override
public long findGroupCountByQueryCriteria(GroupQueryImpl query) {
    return findGroupByQueryCriteria(query, null).size();
}
```

可以重写更新组的其他方法以引发异常:

```java
public Group createNewGroup(String groupId) {
    throw new UnsupportedOperationException("This operation is not supported!");
}
```

### 4.3。流程引擎配置

在定义了两个 identity manager 类之后，我们需要将它们连接到配置中。

弹簧启动器为我们自动配置了一个`SpringProcessEngineConfiguration`。为了修改这一点，我们可以使用一个`InitializingBean:`

```java
@Autowired
private SpringProcessEngineConfiguration processEngineConfiguration;

@Autowired
private JdbcUserDetailsManager userManager;

@Bean
InitializingBean processEngineInitializer() {
    return new InitializingBean() {
        public void afterPropertiesSet() throws Exception {
            processEngineConfiguration.setUserEntityManager(
              new SpringSecurityUserManager(processEngineConfiguration, 
              new MybatisUserDataManager(processEngineConfiguration), userManager));
            processEngineConfiguration.setGroupEntityManager(
              new SpringSecurityGroupManager(processEngineConfiguration, 
              new MybatisGroupDataManager(processEngineConfiguration)));
            }
        };
    }
```

这里，现有的`processEngineConfiguration`被修改为使用我们的定制身份管理器。

如果我们想在 Activiti 中设置当前用户，我们可以使用方法:

```java
identityService.setAuthenticatedUserId(userId);
```

请记住，这将设置一个`ThreadLocal`属性，因此每个线程的值都不同。

## 5。结论

在本文中，我们看到了将 Activiti 与 Spring Security 集成的两种方式。

完整的源代码可以在 GitHub 上找到[。](https://web.archive.org/web/20220627084230/https://github.com/eugenp/tutorials/tree/master/spring-activiti)