# Spring Data Redis 基于属性的配置

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-data-redis-properties>

## 1.概观

Spring Boot 的主要吸引力之一是它经常将第三方配置减少到几个属性。

在本教程中，我们将看到 **Spring Boot 如何简化 Redis 的工作。**

## 2.干嘛再说一遍？

Redis 是最流行的内存数据结构存储之一。因此，它可以用作数据库、缓存和消息代理。

性能方面，因其[快速响应时间](https://web.archive.org/web/20220525011721/https://redis.io/topics/benchmarks)而众所周知。因此，它每秒可以处理数十万次操作，并且易于扩展。

**而且，它与 Spring Boot 应用程序**配合得很好。例如，我们可以在微服务架构中将其用作缓存。我们也可以用它作为 NoSQL 的数据库。

## 3.跑啊跑啊跑啊跑啊跑啊跑啊跑啊跑啊跑啊跑啊跑啊跑啊跑啊跑啊跑啊跑啊跑啊跑啊跑啊跑啊跑啊跑啊跑啊跑啊跑

首先，让我们使用他们的官方 Docker 映像创建一个 Redis 实例。

```
$ docker run -p 16379:6379 -d redis:6.0 redis-server --requirepass "mypass"
```

上面，我们刚刚在端口`16379`上用密码`mypass`启动了一个 Redis 实例。

## 4.启动器

Spring 为我们使用 [Spring Data Redis](/web/20220525011721/https://www.baeldung.com/spring-data-redis-tutorial) 连接 Spring Boot 应用程序和 Redis 提供了强大的支持。

因此，接下来，让我们确保在我们的`pom.xml`中有 [`spring-boot-starter-data-redis`](https://web.archive.org/web/20220525011721/https://search.maven.org/search?q=g:org.springframework.boot%20AND%20a:spring-boot-starter-data-redis) 依赖项:

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
    <version>2.2.6.RELEASE</version>    
</dependency>
```

## 5.生菜；莴苣

接下来，让我们配置客户端。

我们将使用的 Java Redis 客户端**是[莴苣](/web/20220525011721/https://www.baeldung.com/java-redis-lettuce)，因为 Spring Boot 默认使用它。然而，我们也可以使用[杰迪斯](/web/20220525011721/https://www.baeldung.com/jedis-java-redis-client-library)。**

无论哪种方式，结果都是一个`RedisTemplate`的实例:

```
@Bean
public RedisTemplate<Long, Book> redisTemplate(RedisConnectionFactory connectionFactory) {
    RedisTemplate<Long, Book> template = new RedisTemplate<>();
    template.setConnectionFactory(connectionFactory);
    // Add some specific configuration here. Key serializers, etc.
    return template;
}
```

## 6.性能

当我们使用莴苣时，我们不需要配置`RedisConnectionFactory.` Spring Boot 为我们做了。

那么，我们剩下的就是在我们的`application.properties`文件中指定一些属性:

```
spring.redis.database=0
spring.redis.host=localhost
spring.redis.port=16379
spring.redis.password=mypass
spring.redis.timeout=60000
```

分别是:

*   `database`设置连接工厂使用的数据库索引
*   `host`是服务器主机所在的位置
*   `port`表示服务器监听的端口
*   `password`是服务器的登录密码，以及
*   `timeout`建立连接超时

当然，我们还可以配置许多其他属性。Spring Boot 文档中提供了配置属性的[完整列表。](https://web.archive.org/web/20220525011721/https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#data-properties)

## 7.演示

最后，让我们尝试在我们的应用程序中使用它。如果我们想象一个`Book` 类和一个`BookRepository,` 类，我们可以创建和检索`Book`，使用我们的`[RedisTemplate](https://web.archive.org/web/20220525011721/https://docs.spring.io/spring-data/redis/docs/current/api/org/springframework/data/redis/core/RedisTemplate.html)`与 Redis 交互作为我们的后端:

```
@Autowired
private RedisTemplate<Long, Book> redisTemplate;

public void save(Book book) {
    redisTemplate.opsForValue().set(book.getId(), book);
}

public Book findById(Long id) {
    return redisTemplate.opsForValue().get(id);
}
```

默认情况下，生菜将为我们管理序列化和反序列化，所以在这一点上没有什么要做的。然而，很高兴知道这也可以配置。

另一个重要的特性是，由于**redistemp****是线程安全的**，所以它将在[多线程环境](/web/20220525011721/https://www.baeldung.com/java-thread-safety)中正常工作。

## 8.结论

在本文中，我们配置 Spring Boot 通过莴苣与 Redis 对话。而且，我们用一个启动器、一个单一的`@Bean`配置和一些属性实现了它。

最后，我们使用`RedisTemplate`让 Redis 充当一个简单的后端。

完整的例子可以在 GitHub 上找到[。](https://web.archive.org/web/20220525011721/https://github.com/eugenp/tutorials/tree/master/persistence-modules/redis)