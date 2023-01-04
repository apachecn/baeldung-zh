# spring boot cache with redis 春季启动快取记忆体

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-boot-redis-cache>

## 1.概观

在这个简短的教程中，我们将了解如何将 [Redis](/web/20220707143815/https://www.baeldung.com/spring-data-redis-tutorial) 配置为 Spring Boot 缓存的数据存储。

## 延伸阅读:

## [Spring Boot 和咖啡因缓存](/web/20220707143815/https://www.baeldung.com/spring-boot-caffeine-cache)

Learn how to use Caffeine cache with Spring Boot[Read more](/web/20220707143815/https://www.baeldung.com/spring-boot-caffeine-cache) →

## [在 Spring 中使用多个缓存管理器](/web/20220707143815/https://www.baeldung.com/spring-multiple-cache-managers)

Learn how we can enable multiple cache managers in our Spring Boot application.[Read more](/web/20220707143815/https://www.baeldung.com/spring-multiple-cache-managers) →

## [Spring Boot 的缓存回收](/web/20220707143815/https://www.baeldung.com/spring-boot-evict-cache)

Learn how to invalidate caches with Spring Boot.[Read more](/web/20220707143815/https://www.baeldung.com/spring-boot-evict-cache) →

## 2.属国

要开始，让我们添加 [`spring-boot-starter-cache`](https://web.archive.org/web/20220707143815/https://search.maven.org/artifact/org.springframework.boot/spring-boot-starter-cache) 和 [`spring-boot-starter-data-redis`](https://web.archive.org/web/20220707143815/https://search.maven.org/artifact/org.springframework.boot/spring-boot-starter-cache) 神器:

```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-cache</artifactId>
    <version>2.4.3</version>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
    <version>2.4.3</version>
</dependency>
```

这些增加了缓存支持，并引入了所有必需的依赖项。

## 3.配置

通过添加上述依赖项和`@EnableCaching`注释，Spring Boot 将使用默认缓存配置自动配置一个`RedisCacheManager`。但是，在缓存管理器初始化之前，我们可以用一些有用的方法修改这个配置。

首先，让我们创建一个`RedisCacheConfiguration` bean:

```java
@Bean
public RedisCacheConfiguration cacheConfiguration() {
    return RedisCacheConfiguration.defaultCacheConfig()
      .entryTtl(Duration.ofMinutes(60))
      .disableCachingNullValues()
      .serializeValuesWith(SerializationPair.fromSerializer(new GenericJackson2JsonRedisSerializer()));
}
```

这让我们对默认配置有了更多的控制。例如，我们可以设置期望的生存时间(TTL)值，并定制动态缓存创建的默认序列化策略。

为了完全控制缓存设置，让我们注册自己的`RedisCacheManagerBuilderCustomizer` bean:

```java
@Bean
public RedisCacheManagerBuilderCustomizer redisCacheManagerBuilderCustomizer() {
    return (builder) -> builder
      .withCacheConfiguration("itemCache",
        RedisCacheConfiguration.defaultCacheConfig().entryTtl(Duration.ofMinutes(10)))
      .withCacheConfiguration("customerCache",
        RedisCacheConfiguration.defaultCacheConfig().entryTtl(Duration.ofMinutes(5)));
}
```

这里我们使用了`RedisCacheManagerBuilder`和`RedisCacheConfiguration`来分别为`itemCache`和`customerCache`配置 10 分钟和 5 分钟的 TTL 值。这有助于进一步**微调每个缓存的缓存行为**，包括`null`值、关键字前缀和二进制序列化。

值得一提的是，Redis 实例的默认连接细节是`localhost:6379`。 [Redis 配置](/web/20220707143815/https://www.baeldung.com/spring-data-redis-tutorial#the-redis-configuration)可用于进一步调整主机和端口的底层连接细节。

## 4.例子

在我们的例子中，我们有一个从数据库中检索商品信息的`ItemService`组件。实际上，这是一个潜在的高成本操作，也是一个很好的缓存选择。

首先，让我们使用一个[嵌入式 Redis](/web/20220707143815/https://www.baeldung.com/spring-embedded-redis) 服务器为这个组件创建集成测试:

```java
@Import({ CacheConfig.class, ItemService.class})
@ExtendWith(SpringExtension.class)
@EnableCaching
@ImportAutoConfiguration(classes = { 
  CacheAutoConfiguration.class, 
  RedisAutoConfiguration.class 
})
class ItemServiceCachingIntegrationTest {

    @MockBean
    private ItemRepository mockItemRepository;

    @Autowired
    private ItemService itemService;

    @Autowired
    private CacheManager cacheManager;

    @Test
    void givenRedisCaching_whenFindItemById_thenItemReturnedFromCache() {
        Item anItem = new Item(AN_ID, A_DESCRIPTION);
        given(mockItemRepository.findById(AN_ID))
          .willReturn(Optional.of(anItem));

        Item itemCacheMiss = itemService.getItemForId(AN_ID);
        Item itemCacheHit = itemService.getItemForId(AN_ID);

        assertThat(itemCacheMiss).isEqualTo(anItem);
        assertThat(itemCacheHit).isEqualTo(anItem);

        verify(mockItemRepository, times(1)).findById(AN_ID);
        assertThat(itemFromCache()).isEqualTo(anItem);
    }
}
```

这里我们为缓存行为创建一个测试片，并调用两次`getItemForId`。第一次调用应该从存储库中获取项目，但是第二次调用应该从缓存中返回项目，而不调用存储库。

最后，让我们使用 Spring 的`@Cacheable`注释来启用缓存行为:

```java
@Cacheable(value = "itemCache")
public Item getItemForId(String id) {
    return itemRepository.findById(id)
      .orElseThrow(RuntimeException::new);
}
```

这应用了缓存逻辑，同时依赖于我们之前配置的 Redis 缓存基础设施。关于控制 Spring 缓存抽象的属性和行为的更多细节，包括数据更新和驱逐，在我们的[Spring 缓存指南](/web/20220707143815/https://www.baeldung.com/spring-cache-tutorial)中有所介绍。

## 5.结论

在本文中，我们看到了如何使用 Redis 进行 Spring 缓存。

我们首先描述了如何用最少的配置自动配置 Redis 缓存。然后我们看了如何通过注册配置 beans 来进一步定制缓存行为。

最后，我们创建了一个样例用例来演示实际的缓存。

和往常一样，完整的源代码可以在 GitHub 上找到[。](https://web.archive.org/web/20220707143815/https://github.com/eugenp/tutorials/tree/master/spring-caching-2)