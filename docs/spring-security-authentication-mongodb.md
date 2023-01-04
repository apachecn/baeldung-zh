# 使用 Spring Security 和 MongoDB 进行身份验证

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-security-authentication-mongodb>

## 1.概观

[春安](/web/20221011171932/https://www.baeldung.com/security-spring) 等提供了不同的认证系统，通过数据库 [`UserDetailService`](/web/20221011171932/https://www.baeldung.com/spring-security-authentication-with-a-database) 。

除了使用一个 [JPA](/web/20221011171932/https://www.baeldung.com/the-persistence-layer-with-spring-data-jpa) 持久层之外，我们可能还想使用一个 [MongoDB 存储库](/web/20221011171932/https://www.baeldung.com/spring-data-mongodb-tutorial)。在本教程中，我们将看到如何使用 Spring Security 和 MongoDB 来认证用户。

## 2.用 MongoDB 进行 Spring 安全认证

**类似于使用 JPA 存储库，我们可以使用 MongoDB 存储库**。但是，为了使用它，我们需要设置一个不同的配置。

### 2.1.Maven 依赖性

**对于本教程，我们将使用[嵌入式 MongoDB](/web/20221011171932/https://www.baeldung.com/spring-boot-embedded-mongodb)** 。然而，MongoDB 实例和 [`Testcontainer`](/web/20221011171932/https://www.baeldung.com/spring-boot-testcontainers-integration-test) 可能是生产环境的有效选项。首先，让我们添加 [`spring-boot-starter-data-mongodb`](https://web.archive.org/web/20221011171932/https://search.maven.org/artifact/org.springframework.boot/spring-boot-starter-data-mongodb) 和 [`de.flapdoodle.embed.mongo`](https://web.archive.org/web/20221011171932/https://search.maven.org/artifact/de.flapdoodle.embed/de.flapdoodle.embed.mongo) 的依赖关系:

```
<dependency>
   <groupId>org.springframework.boot</groupId>
   <artifactId>spring-boot-starter-data-mongodb</artifactId>
</dependency>
<dependency>
    <groupId>de.flapdoodle.embed</groupId>
    <artifactId>de.flapdoodle.embed.mongo</artifactId>
    <version>3.3.1</version>
</dependency>
```

### 2.2.配置

一旦我们设置了依赖关系，我们就可以创建我们的配置:

```
@Configuration
public class MongoConfig {

    private static final String CONNECTION_STRING = "mongodb://%s:%d";
    private static final String HOST = "localhost";

    @Bean
    public MongoTemplate mongoTemplate() throws Exception {

        int randomPort = SocketUtils.findAvailableTcpPort();

        ImmutableMongodConfig mongoDbConfig = MongodConfig.builder()
          .version(Version.Main.PRODUCTION)
          .net(new Net(HOST, randomPort, Network.localhostIsIPv6()))
          .build();

        MongodStarter starter = MongodStarter.getDefaultInstance();
        MongodExecutable mongodExecutable = starter.prepare(mongoDbConfig);
        mongodExecutable.start();
        return new MongoTemplate(MongoClients.create(String.format(CONNECTION_STRING, HOST, randomPort)), "mongo_auth");
    }
}
```

我们还需要为我们的`[AuthenticationManager](https://web.archive.org/web/20221011171932/https://spring.io/guides/topicals/spring-security-architecture)`配置一个 HTTP 基本认证:

```
@Configuration
@EnableWebSecurity
@EnableGlobalMethodSecurity(securedEnabled = true, jsr250Enabled = true)
public class SecurityConfig extends WebSecurityConfigurerAdapter {

    // ...
    public SecurityConfig(UserDetailsService userDetailsService) {
        this.userDetailsService = userDetailsService;
    }

    @Bean
    public AuthenticationManager customAuthenticationManager() throws Exception {
        return authenticationManager();
    }

    @Bean
    public BCryptPasswordEncoder bCryptPasswordEncoder() {
        return new BCryptPasswordEncoder();
    }

    @Override
    protected void configure(@Autowired AuthenticationManagerBuilder auth) throws Exception {
        auth.userDetailsService(userDetailsService)
          .passwordEncoder(bCryptPasswordEncoder());
    }

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.csrf()
          .disable()
          .authorizeRequests()
          .and()
          .httpBasic()
          .and()
          .authorizeRequests()
          .anyRequest()
          .permitAll()
          .and()
          .sessionManagement()
          .sessionCreationPolicy(SessionCreationPolicy.STATELESS);
    }
}
```

### 2.3.用户域和存储库

首先，让我们为我们的认证定义一个简单的用户和角色。我们将让它实现 [`UserDetails`](https://web.archive.org/web/20221011171932/https://docs.spring.io/spring-security/reference/servlet/authentication/passwords/user-details.html#page-title) 接口来重用`Principal`对象的公共方法:

```
@Document
public class User implements UserDetails {
    private @MongoId ObjectId id;
    private String username;
    private String password;
    private Set<UserRole> userRoles;
    // getters and setters
} 
```

现在我们有了用户，让我们定义一个简单存储库:

```
public interface UserRepository extends MongoRepository<User, String> {

    @Query("{username:'?0'}")
    User findUserByUsername(String username);
}
```

### 2.4.认证服务

**最后，让我们实现我们的`UserDetailService` 来检索一个用户并检查它是否经过认证**:

```
@Service
public class MongoAuthUserDetailService implements UserDetailsService {
    // ...
    @Override
    public UserDetails loadUserByUsername(String userName) throws UsernameNotFoundException {

        com.baeldung.mongoauth.domain.User user = userRepository.findUserByUsername(userName);

        Set<GrantedAuthority> grantedAuthorities = new HashSet<>();

        user.getAuthorities()
          .forEach(role -> {
              grantedAuthorities.add(new SimpleGrantedAuthority(role.getRole()
                 .getName()));
          });

        return new User(user.getUsername(), user.getPassword(), grantedAuthorities);
    }

}
```

### 2.5.测试认证

为了测试我们的应用程序，让我们定义一个简单的控制器。例如，我们定义了两个不同的角色来测试特定端点的身份验证和授权:

```
@RestController
public class ResourceController {

    @RolesAllowed("ROLE_ADMIN")
    @GetMapping("/admin")
    public String admin() {
        return "Hello Admin!";
    }

    @RolesAllowed({ "ROLE_ADMIN", "ROLE_USER" })
    @GetMapping("/user")
    public String user() {
        return "Hello User!";
    }

}
```

让我们用一个 [Spring Boot 测试](/web/20221011171932/https://www.baeldung.com/spring-boot-testing)来检查我们的认证是否有效。正如我们所看到的，**我们正在等待一个 401 代码，针对提供无效凭证或在我们的系统中不存在的人**:

```
class MongoAuthApplicationTest {

    // set up

    @Test
    void givenUserCredentials_whenInvokeUserAuthorizedEndPoint_thenReturn200() throws Exception {
        mvc.perform(get("/user").with(httpBasic(USER_NAME, PASSWORD)))
          .andExpect(status().isOk());
    }

    @Test
    void givenUserNotExists_whenInvokeEndPoint_thenReturn401() throws Exception {
        mvc.perform(get("/user").with(httpBasic("not_existing_user", "password")))
          .andExpect(status().isUnauthorized());
    }

    @Test
    void givenUserExistsAndWrongPassword_whenInvokeEndPoint_thenReturn401() throws Exception {
        mvc.perform(get("/user").with(httpBasic(USER_NAME, "wrong_password")))
          .andExpect(status().isUnauthorized());
    }

    @Test
    void givenUserCredentials_whenInvokeAdminAuthorizedEndPoint_thenReturn403() throws Exception {
        mvc.perform(get("/admin").with(httpBasic(USER_NAME, PASSWORD)))
          .andExpect(status().isForbidden());
    }

    @Test
    void givenAdminCredentials_whenInvokeAdminAuthorizedEndPoint_thenReturn200() throws Exception {
        mvc.perform(get("/admin").with(httpBasic(ADMIN_NAME, PASSWORD)))
          .andExpect(status().isOk());

        mvc.perform(get("/user").with(httpBasic(ADMIN_NAME, PASSWORD)))
          .andExpect(status().isOk());
    }
}
```

## 3.结论

在本文中，我们研究了使用 Spring Security 进行身份验证的 MongoDB。

我们已经看到了如何创建一个工作配置并实现我们的定制`UserDetailService`。我们还看到了如何模拟 MVC 上下文并测试认证和授权。

和往常一样，这些例子的代码可以在 GitHub 的[上找到。](https://web.archive.org/web/20221011171932/https://github.com/eugenp/tutorials/tree/master/spring-security-modules/spring-security-web-boot-3)