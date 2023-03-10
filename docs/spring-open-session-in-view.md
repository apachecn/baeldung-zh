# 春季公开会议指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-open-session-in-view>

## 1.概观

每个请求的会话是一种事务模式，将持久性会话和请求生命周期联系在一起。不足为奇的是，Spring 自带了这种模式的实现，名为`[OpenSessionInViewInterceptor](https://web.archive.org/web/20221013182823/https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/orm/hibernate5/support/OpenSessionInViewInterceptor.html)`，以方便使用惰性关联，从而提高开发人员的生产率。

在本教程中，首先，我们将学习拦截器内部是如何工作的，然后，我们将看到这个有争议的[模式如何成为我们应用程序的一把双刃剑！](https://web.archive.org/web/20221013182823/https://github.com/spring-projects/spring-boot/issues/7107)

## 2.在视图中引入开放会话

为了更好地理解 Open Session 在 View (OSIV)中的作用，让我们假设有一个传入请求:

1.  Spring 在请求开始时打开一个新的 Hibernate `Session `。这些`Sessions `不一定连接到数据库。
2.  每当应用程序需要一个`Session, `时，它将重用已经存在的那个。
3.  在请求结束时，同一个拦截器关闭了那个`Session.`

乍一看，启用这个特性可能是有意义的。毕竟，框架处理会话的创建和终止，所以开发人员不用关心这些看似底层的细节。这反过来提高了开发人员的生产力。

然而，有时， **OSIV 会在生产中引起微妙的性能问题**。通常，这类问题很难诊断。

### 2.1.Spring Boot

**默认情况下，OSIV 在 Spring Boot 申请中是活跃的**。尽管如此，从 Spring Boot 2.0 开始，它警告我们，如果我们没有明确配置它，它会在应用程序启动时启用:

```java
spring.jpa.open-in-view is enabled by default. Therefore, database 
queries may be performed during view rendering.Explicitly configure 
spring.jpa.open-in-view to disable this warning
```

无论如何，我们可以通过使用`spring.jpa.open-in-view`配置属性来禁用 OSIV:

```java
spring.jpa.open-in-view=false
```

### 2.2.模式还是反模式？

对 OSIV 的反应总是褒贬不一。支持 OSIV 阵营的主要论点是开发者的生产力，尤其是在处理懒惰联想的时候。

另一方面，数据库性能问题是反 OSIV 运动的主要论点。稍后，我们将详细评估这两个论点。

## 3.懒惰初始化英雄

由于 OSIV 将`Session `生命周期绑定到每个请求， **Hibernate 甚至可以在从一个显式的** `**@Transactional**` **服务**返回后解决懒惰关联。

为了更好地理解这一点，让我们假设我们正在对我们的用户及其安全权限进行建模:

```java
@Entity
@Table(name = "users")
public class User {

    @Id
    @GeneratedValue
    private Long id;

    private String username;

    @ElementCollection
    private Set<String> permissions;

    // getters and setters
}
```

与其他一对多和多对多关系类似，`permissions`属性是一个惰性集合。

然后，在我们的服务层实现中，让我们使用`@Transactional`明确划分我们的事务边界:

```java
@Service
public class SimpleUserService implements UserService {

    private final UserRepository userRepository;

    public SimpleUserService(UserRepository userRepository) {
        this.userRepository = userRepository;
    }

    @Override
    @Transactional(readOnly = true)
    public Optional<User> findOne(String username) {
        return userRepository.findByUsername(username);
    }
}
```

### 3.1.期望

当我们的代码调用`findOne `方法时，我们预计会发生以下情况:

1.  首先，Spring 代理拦截调用并获取当前事务，如果不存在，则创建一个事务。
2.  然后，它将方法调用委托给我们的实现。
3.  **最后，代理提交事务并因此关闭底层`Session`。毕竟，我们只需要服务层中的那个`Session `。**

在`findOne `方法实现中，我们没有初始化`permissions `集合。**因此，我们不应该在** **方法返回后使用`permissions `** **。**如果我们迭代这个属性`, `，我们应该得到一个`LazyInitializationException.`

### 3.2.欢迎来到现实世界

让我们编写一个简单的 REST 控制器，看看我们是否可以使用`permissions` 属性:

```java
@RestController
@RequestMapping("/users")
public class UserController {

    private final UserService userService;

    public UserController(UserService userService) {
        this.userService = userService;
    }

    @GetMapping("/{username}")
    public ResponseEntity<?> findOne(@PathVariable String username) {
        return userService
                .findOne(username)
                .map(DetailedUserDto::fromEntity)
                .map(ResponseEntity::ok)
                .orElse(ResponseEntity.notFound().build());
    }
}
```

这里，我们在实体到 DTO 的转换过程中迭代`permissions `。因为我们预计转换会因`LazyInitializationException,`而失败，所以下面的测试应该不会通过:

```java
@SpringBootTest
@AutoConfigureMockMvc
@ActiveProfiles("test")
class UserControllerIntegrationTest {

    @Autowired
    private UserRepository userRepository;

    @Autowired
    private MockMvc mockMvc;

    @BeforeEach
    void setUp() {
        User user = new User();
        user.setUsername("root");
        user.setPermissions(new HashSet<>(Arrays.asList("PERM_READ", "PERM_WRITE")));

        userRepository.save(user);
    }

    @Test
    void givenTheUserExists_WhenOsivIsEnabled_ThenLazyInitWorksEverywhere() throws Exception {
        mockMvc.perform(get("/users/root"))
          .andExpect(status().isOk())
          .andExpect(jsonPath("$.username").value("root"))
          .andExpect(jsonPath("$.permissions", containsInAnyOrder("PERM_READ", "PERM_WRITE")));
    }
}
```

然而，这个测试没有抛出任何异常，它通过了。

**因为 OSIV 在请求开始时创建了一个`Session `，事务代理使用当前可用的`Session` ，而不是创建一个全新的`.`**

因此，尽管我们可能有所期待，我们实际上甚至可以在显式的`@Transactional`之外使用`permissions `属性。此外，可以在当前请求范围内的任何地方获取这些种类的惰性关联。

### 3.3.论开发者生产力

**如果没有启用 OSIV，我们将不得不在事务上下文中手动初始化所有必要的惰性关联**。最基本的(通常也是错误的)方法是使用`Hibernate.initialize() `方法:

```java
@Override
@Transactional(readOnly = true)
public Optional<User> findOne(String username) {
    Optional<User> user = userRepository.findByUsername(username);
    user.ifPresent(u -> Hibernate.initialize(u.getPermissions()));

    return user;
}
```

到目前为止，OSIV 对开发人员生产力的影响是显而易见的。然而，这并不总是关于开发人员的生产力。

## 4.表演反派

假设我们必须扩展我们的简单用户服务，以便在从数据库获取用户后**调用另一个远程服务:**

```java
@Override
public Optional<User> findOne(String username) {
    Optional<User> user = userRepository.findByUsername(username);
    if (user.isPresent()) {
        // remote call
    }

    return user;
}
```

这里，我们删除了`@Transactional `注释，因为我们显然不想在等待远程服务时保持连接的`Session `。

### 4.1.避免混合 io

让我们弄清楚如果不移除`@Transactional `注释会发生什么。**假设新的远程服务的响应速度比平时稍慢:**

1.  首先，Spring 代理获取当前的`Session`或者创建一个新的。无论哪种方式，这个`Session `都还没有连接。也就是说，它没有使用池中的任何连接。
2.  一旦我们执行查询来找到一个用户，`Session `就会连接起来，并从池中借用一个`Connection `。
3.  如果整个方法是事务性的，那么该方法继续调用慢速远程服务，同时保留被借用的`Connection`。

**想象一下，在此期间，我们收到了对`findOne `方法的大量调用。**然后，过一会儿，所有的`Connections `可以等待 API 调用的响应。因此，**我们可能很快就会耗尽数据库连接。**

在事务上下文中将数据库 io 与其他类型的 io 混合在一起是一种不好的味道，我们应该不惜一切代价避免它。

总之，**因为我们从服务中移除了`@Transactional `注释，所以我们希望是安全的**。

### 4.2.耗尽连接池

**当 OSIV 激活`, `时，在当前请求范围**中总有一个`Session `，即使我们去掉了`@Transactional`。虽然这个`Session `最初没有连接，但是在我们的第一个数据库 IO 之后，它就连接上了，并且一直保持到请求结束。

因此，在 OSIV 面前，我们看似无辜且最近优化的服务实现是一个灾难的处方:

```java
@Override
public Optional<User> findOne(String username) {
    Optional<User> user = userRepository.findByUsername(username);
    if (user.isPresent()) {
        // remote call
    }

    return user;
}
```

以下是启用 OSIV 时发生的情况:

1.  在请求开始时，相应的过滤器创建一个新的`Session`。
2.  当我们调用`findByUsername `方法时，那个`Session `从池中借用了一个`Connection `。
3.  `Session `保持连接，直到请求结束。

即使我们期望我们的服务代码不会耗尽连接池，仅仅是 OSIV 的存在就有可能使整个应用程序没有响应。

更糟糕的是，**问题的根本原因(缓慢的远程服务)和症状(数据库连接池)是不相关的**。因为这种相关性很小，所以在生产环境中很难诊断这种性能问题。

### 4.3.不必要的查询

不幸的是，耗尽连接池并不是唯一与 OSIV 相关的性能问题。

由于`Session `在整个请求生命周期中是开放的，**一些属性导航可能会在事务上下文**之外触发一些不需要的查询。甚至有可能以 [n+1 选择问题](/web/20221013182823/https://www.baeldung.com/hibernate-common-performance-problems-in-logs)结束，最糟糕的消息是我们可能直到生产时才注意到这一点。

雪上加霜的是， **`Session `在[自动提交模式](https://web.archive.org/web/20221013182823/https://docs.oracle.com/en/java/javase/11/docs/api/java.sql/java/sql/Connection.html#setAutoCommit(boolean))** 下执行所有那些额外的查询。在自动提交模式下，每个 SQL 语句都被视为一个事务，并在执行后立即自动提交。这反过来给数据库带来了很大的压力。

## 5.明智地选择

OSIV 是一种模式还是一种反模式并不重要。这里最重要的是我们生活的现实。

如果我们正在开发一个简单的 CRUD 服务，使用 OSIV 可能是有意义的，因为我们可能永远不会遇到那些性能问题。

另一方面，**如果我们发现自己调用了大量远程服务，或者在我们的事务上下文之外发生了如此多的事情，强烈建议完全禁用 OSIV。**

如果有疑问，请在没有 OSIV 的情况下开始，因为我们稍后可以轻松启用它。另一方面，禁用已经启用的 OSIV 可能很麻烦，因为我们可能需要处理大量的`LazyInitializationExceptions.`

底线是，当使用或忽略 OSIV 时，我们应该知道权衡。

## 6.可供选择的事物

如果我们禁用了 OSIV，那么在处理懒惰联想时，我们应该以某种方式阻止潜在的`LazyInitializationExceptions `。在众多应对懒惰联想的方法中，我们将在这里列举其中的两种。

### 6.1.实体图

在 Spring Data JPA 中定义查询方法时，我们可以用`@EntityGraph `到[来注释一个查询方法，急切地获取实体](/web/20221013182823/https://www.baeldung.com/spring-data-jpa-named-entity-graphs)的某个部分:

```java
public interface UserRepository extends JpaRepository<User, Long> {

    @EntityGraph(attributePaths = "permissions")
    Optional<User> findByUsername(String username);
}
```

这里，我们定义了一个特别的实体图来急切地加载`permissions `属性，尽管默认情况下它是一个惰性集合。

如果我们需要从同一个查询返回多个投影，那么我们应该用不同的实体图配置定义多个查询:

```java
public interface UserRepository extends JpaRepository<User, Long> {
    @EntityGraph(attributePaths = "permissions")
    Optional<User> findDetailedByUsername(String username);

    Optional<User> findSummaryByUsername(String username);
}
```

### 6.2.使用`Hibernate.initialize()`时的注意事项

有人可能会说，不使用实体图，我们可以使用臭名昭著的`Hibernate.initialize() `来获取我们需要的懒惰关联:

```java
@Override
@Transactional(readOnly = true)
public Optional<User> findOne(String username) {
    Optional<User> user = userRepository.findByUsername(username);
    user.ifPresent(u -> Hibernate.initialize(u.getPermissions()));

    return user;
}
```

他们可能很聪明，还建议调用`getPermissions() `方法来触发获取过程:

```java
Optional<User> user = userRepository.findByUsername(username);
user.ifPresent(u -> {
    Set<String> permissions = u.getPermissions();
    System.out.println("Permissions loaded: " + permissions.size());
});
```

这两种方法都不推荐，因为**除了最初的查询之外，它们(至少)会导致一个额外的查询**来获取惰性关联。也就是说，Hibernate 生成以下查询来获取用户及其权限:

```java
> select u.id, u.username from users u where u.username=?
> select p.user_id, p.permissions from user_permissions p where p.user_id=? 
```

尽管大多数数据库非常擅长执行第二个查询，但是我们应该避免额外的网络往返。

另一方面，如果我们使用实体图或者甚至是[获取连接](/web/20221013182823/https://www.baeldung.com/jpa-join-types)，Hibernate 将通过一个查询获取所有必要的数据:

```java
> select u.id, u.username, p.user_id, p.permissions from users u 
  left outer join user_permissions p on u.id=p.user_id where u.username=?
```

## 7.结论

在本文中，我们将注意力转向 Spring 和其他一些企业框架中一个颇有争议的特性:Open Session in View。首先，我们在概念上和实现上都有了这个模式。然后我们从生产力和性能的角度进行了分析。

像往常一样，GitHub 上的示例代码[可用。](https://web.archive.org/web/20221013182823/https://github.com/eugenp/tutorials/tree/master/persistence-modules/spring-data-jpa-enterprise)