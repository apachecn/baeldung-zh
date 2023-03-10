# 带有 Spring Boot 测试的嵌入式 Redis 服务器

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-embedded-redis>

## 1.概观

Spring Data Redis 提供了一种与 [Redis](https://web.archive.org/web/20221206144637/https://redis.io/) 实例集成的简单方法。

然而，在某些情况下，使用嵌入式服务器比用真实的服务器创建环境更方便。

因此，我们将学习如何设置和使用嵌入式 Redis 服务器。

## 2.属国

让我们从添加必要的依赖项开始:

```java
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>

<dependency>
  <groupId>it.ozimov</groupId>
  <artifactId>embedded-redis</artifactId>
  <version>0.7.2</version>
  <scope>test</scope>
</dependency>

<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-test</artifactId>
  <scope>test</scope>
</dependency>
```

`[spring-boot-starter-test](https://web.archive.org/web/20221206144637/https://search.maven.org/search?q=g:org.springframework.boot%20AND%20a:spring-boot-starter-test&core=gav)`依赖包含了我们运行集成测试所需的一切。

此外，`[embedded-redis](https://web.archive.org/web/20221206144637/https://search.maven.org/search?q=g:it.ozimov%20AND%20a:embedded-redis&core=gav)`包含我们将使用的嵌入式服务器。

## 3.设置

添加完依赖项后，我们应该定义 Redis 服务器和应用程序之间的连接设置。

让我们首先创建一个保存我们属性的类:

```java
@Configuration
public class RedisProperties {
    private int redisPort;
    private String redisHost;

    public RedisProperties(
      @Value("${spring.redis.port}") int redisPort, 
      @Value("${spring.redis.host}") String redisHost) {
        this.redisPort = redisPort;
        this.redisHost = redisHost;
    }

    // getters
}
```

接下来，我们应该创建一个配置类来定义连接并使用我们的属性:

```java
@Configuration
@EnableRedisRepositories
public class RedisConfiguration {

    @Bean
    public LettuceConnectionFactory redisConnectionFactory(
      RedisProperties redisProperties) {
        return new LettuceConnectionFactory(
          redisProperties.getRedisHost(), 
          redisProperties.getRedisPort());
    }

    @Bean
    public RedisTemplate<?, ?> redisTemplate(LettuceConnectionFactory connectionFactory) {
        RedisTemplate<byte[], byte[]> template = new RedisTemplate<>();
        template.setConnectionFactory(connectionFactory);
        return template;
    }
}
```

配置相当简单。此外，它允许我们在不同的端口上运行嵌入式服务器。

查看我们的[Spring Data Redis 简介](/web/20221206144637/https://www.baeldung.com/spring-data-redis-tutorial)文章，了解更多关于 Spring Boot Redis 的信息。

## 4.嵌入式重定向服务器

现在，我们将配置嵌入式服务器并在我们的一个测试中使用它。

首先，让我们在测试资源目录(`src/test/resources):`中创建一个`application.properties` 文件

```java
spring.redis.host=localhost
spring.redis.port=6370
```

之后，我们将创建一个带`@TestConfiguration`注释的类:

```java
@TestConfiguration
public class TestRedisConfiguration {

    private RedisServer redisServer;

    public TestRedisConfiguration(RedisProperties redisProperties) {
        this.redisServer = new RedisServer(redisProperties.getRedisPort());
    }

    @PostConstruct
    public void postConstruct() {
        redisServer.start();
    }

    @PreDestroy
    public void preDestroy() {
        redisServer.stop();
    }
}
```

一旦上下文启动，服务器将启动。它将在我们的机器上我们已经在属性中定义的端口上启动。例如，我们现在可以在不停止实际 Redis 服务器的情况下运行测试。

理想情况下，我们希望在随机可用的端口上启动它，但是 embedded Redis 还没有这个特性。我们现在可以做的是通过 ServerSocket API 获取随机端口。

此外，一旦上下文被破坏，服务器将停止。

服务器也可以提供我们自己的可执行文件:

```java
this.redisServer = new RedisServer("/path/redis", redisProperties.getRedisPort());
```

此外，可执行文件可以根据操作系统进行定义:

```java
RedisExecProvider customProvider = RedisExecProvider.defaultProvider()
  .override(OS.UNIX, "/path/unix/redis")
  .override(OS.Windows, Architecture.x86_64, "/path/windows/redis")
  .override(OS.MAC_OS_X, Architecture.x86_64, "/path/macosx/redis")

this.redisServer = new RedisServer(customProvider, redisProperties.getRedisPort());
```

最后，让我们创建一个使用我们的`TestRedisConfiguration`类的测试:

```java
@RunWith(SpringRunner.class)
@SpringBootTest(classes = TestRedisConfiguration.class)
public class UserRepositoryIntegrationTest {

    @Autowired
    private UserRepository userRepository;

    @Test
    public void shouldSaveUser_toRedis() {
        UUID id = UUID.randomUUID();
        User user = new User(id, "name");

        User saved = userRepository.save(user);

        assertNotNull(saved);
    }
}
```

用户已经被保存到我们的嵌入式 Redis 服务器。

此外，我们必须手动将`TestRedisConfiguration `添加到`SpringBootTest. `中，正如我们之前所说的，服务器在测试之前已经启动，在测试之后停止。

## 5.结论

嵌入式 Redis 服务器是测试环境中替代实际服务器的完美工具。我们已经看到了如何配置它以及如何在测试中使用它。

与往常一样，GitHub 上的[提供了示例代码。](https://web.archive.org/web/20221206144637/https://github.com/eugenp/tutorials/tree/master/spring-boot-modules/spring-boot-testing)