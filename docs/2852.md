# Spring Webflux 和@Cacheable 注释

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-webflux-cacheable>

## 1.介绍

在本文中，我们将解释 Spring WebFlux 如何与`@Cacheable`注释交互。首先，我们将讨论一些常见问题以及如何避免它们。接下来，我们将介绍可用的解决方法。最后，像往常一样，我们将提供代码示例。

## 2.`@Cacheable`和反应型

这个话题还是比较新的。在写这篇文章的时候，`@Cacheable`和反应式框架之间还没有流畅的集成。**主要问题是没有非阻塞缓存实现(JSR-107 缓存 API 是阻塞的)。**只有[雷迪斯](/web/20220525142026/https://www.baeldung.com/spring-data-redis-tutorial)在提供反应式驱动。

尽管我们在上一段提到了这个问题，我们仍然可以在我们的服务方法上使用`@Cacheable` 。**这将导致缓存我们的包装器对象(`Mono`或`Flux`)，但不会缓存我们方法的实际结果。**

### 2.1.项目设置

让我们用一个测试来说明这一点。在测试之前，我们需要建立我们的项目。我们将用一个反应式 [MongoDB](/web/20220525142026/https://www.baeldung.com/spring-data-mongodb-tutorial) 驱动程序创建一个简单的 [Spring WebFlux](/web/20220525142026/https://www.baeldung.com/spring-webflux) 项目。我们将使用[测试容器](/web/20220525142026/https://www.baeldung.com/spring-boot-testcontainers-integration-test)，而不是将 MongoDB 作为一个单独的进程运行。

我们的测试类将用`@SpringBootTest` 进行注释，并将包含:

```
final static MongoDBContainer mongoDBContainer = new MongoDBContainer(DockerImageName.parse("mongo:4.0.10"));

@DynamicPropertySource
static void mongoDbProperties(DynamicPropertyRegistry registry) {
    mongoDBContainer.start();
    registry.add("spring.data.mongodb.uri",  mongoDBContainer::getReplicaSetUrl);
}
```

这些行将启动一个 MongoDB 实例，并将 URI 传递给 SpringBoot 来自动配置 Mongo 存储库。

对于这个测试，我们将用`save`和`getItem`方法创建`ItemService`类:

```
@Service
public class ItemService {

    private final ItemRepository repository;

    public ItemService(ItemRepository repository) {
        this.repository = repository;
    }
    @Cacheable("items")
    public Mono<Item> getItem(String id){
        return repository.findById(id);
    }
    public Mono<Item> save(Item item){
        return repository.save(item);
    }
}
```

在`application.properties,`中，我们为缓存和存储库设置了记录器，这样我们可以监控测试中发生的事情:

```
logging.level.org.springframework.data.mongodb.core.ReactiveMongoTemplate=DEBUG
logging.level.org.springframework.cache=TRACE
```

### 2.2.初始测试

设置完成后，我们可以运行测试并分析结果:

```
@Test
public void givenItem_whenGetItemIsCalled_thenMonoIsCached() {
    Mono<Item> glass = itemService.save(new Item("glass", 1.00));

    String id = glass.block().get_id();

    Mono<Item> mono = itemService.getItem(id);
    Item item = mono.block();

    assertThat(item).isNotNull();
    assertThat(item.getName()).isEqualTo("glass");
    assertThat(item.getPrice()).isEqualTo(1.00);

    Mono<Item> mono2 = itemService.getItem(id);
    Item item2 = mono2.block();

    assertThat(item2).isNotNull();
    assertThat(item2.getName()).isEqualTo("glass");
    assertThat(item2.getPrice()).isEqualTo(1.00);
}
```

在控制台中，我们可以看到以下输出(为简洁起见，只显示了基本部分):

```
Inserting Document containing fields: [name, price, _class] in collection: item...
Computed cache key '618817a52bffe4526c60f6c0' for operation Builder[public reactor.core.publisher.Mono...
No cache entry for key '618817a52bffe4526c60f6c0' in cache(s) [items]
Computed cache key '618817a52bffe4526c60f6c0' for operation Builder[public reactor.core.publisher.Mono...
findOne using query: { "_id" : "618817a52bffe4526c60f6c0"} fields: Document{{}} for class: class com.baeldung.caching.Item in collection: item...
findOne using query: { "_id" : { "$oid" : "618817a52bffe4526c60f6c0"}} fields: {} in db.collection: test.item
Computed cache key '618817a52bffe4526c60f6c0' for operation Builder[public reactor.core.publisher.Mono...
Cache entry for key '618817a52bffe4526c60f6c0' found in cache 'items'
findOne using query: { "_id" : { "$oid" : "618817a52bffe4526c60f6c0"}} fields: {} in db.collection: test.item 
```

在第一行，我们看到了我们的插入方法。之后，当调用`getItem`时，Spring 检查该项目的缓存，但没有找到，并访问 MongoDB 以获取该记录。在第二次`getItem`调用时，Spring 再次检查缓存并找到那个键的条目，但是仍然要去 MongoDB 获取这个记录。

**这是因为 Spring 缓存了`getItem`方法的结果，也就是`Mono`包装器对象。但是，对于结果本身，它仍然需要从数据库中获取记录。**

在下面几节中，我们将提供这个问题的解决方法。

## 3.缓存`Mono/Flux`的结果

`Mono`和`Flux`有一个内置的缓存机制，我们可以在这种情况下使用它作为解决方法。如前所述， `@Cacheable` 缓存包装器对象，通过内置缓存，我们可以创建对服务方法实际结果的引用:

```
@Cacheable("items")
public Mono<Item> getItem_withCache(String id) {
    return repository.findById(id).cache();
}
```

让我们用这个新的服务方法运行上一章的测试。输出将如下所示:

```
Inserting Document containing fields: [name, price, _class] in collection: item
Computed cache key '6189242609a72e0bacae1787' for operation Builder[public reactor.core.publisher.Mono...
No cache entry for key '6189242609a72e0bacae1787' in cache(s) [items]
Computed cache key '6189242609a72e0bacae1787' for operation Builder[public reactor.core.publisher.Mono...
findOne using query: { "_id" : "6189242609a72e0bacae1787"} fields: Document{{}} for class: class com.baeldung.caching.Item in collection: item
findOne using query: { "_id" : { "$oid" : "6189242609a72e0bacae1787"}} fields: {} in db.collection: test.item
Computed cache key '6189242609a72e0bacae1787' for operation Builder[public reactor.core.publisher.Mono...
Cache entry for key '6189242609a72e0bacae1787' found in cache 'items'
```

我们可以看到几乎相似的输出。只是这一次，当在缓存中找到一个条目时，没有额外的数据库查找。使用这种解决方案，当我们的缓存过期时，会有一个潜在的问题。**由于我们使用的是缓存中的缓存，我们需要在两个缓存上设置适当的到期时间。经验法则是`Flux`缓存 TTL 应该比`@Cacheable.`** 长

## 4.使用反应器附件

Reactor 3 addon 允许我们通过`CacheMono`和`CacheFlux`类流畅地使用不同的缓存实现。对于这个例子，我们将配置[咖啡因](/web/20220525142026/https://www.baeldung.com/spring-boot-caffeine-cache)缓存:

```
public ItemService(ItemRepository repository) {
    this.repository = repository;
    this.cache = Caffeine.newBuilder().build(this::getItem_withAddons);
}
```

在`the ItemService`构造函数中，我们用最小的配置初始化咖啡因缓存，在新的服务方法中，我们使用该缓存:

```
@Cacheable("items")
public Mono<Item> getItem_withAddons(String id) {
    return CacheMono.lookup(cache.asMap(), id)
      .onCacheMissResume(() -> repository.findById(id).cast(Object.class)).cast(Item.class);
} 
```

因为`CacheMono` 内部使用`Signal`类，我们需要做一些转换来返回适当的对象。

当我们重新运行之前的测试时，我们将得到与上一个示例类似的输出。

## 5.结论

在本文中，我们介绍了 Spring WebFlux 如何与`@Cacheable`交互。此外，我们描述了它们的使用方法和一些常见问题。和往常一样，这篇文章的代码可以在 GitHub 上找到。