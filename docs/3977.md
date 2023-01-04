# 在 Spring 中使用多个缓存管理器

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-multiple-cache-managers>

## 1.概观

在本教程中，我们将学习如何在一个 Spring 应用程序中配置多个缓存管理器。

## 2.贮藏

Spring 对方法应用缓存，这样我们的应用程序就不会对相同的输入多次执行相同的方法。

在一个 Spring 应用 中实现 [缓存非常容易。](/web/20221126213538/https://www.baeldung.com/spring-cache-tutorial)这可以通过在我们的配置类中添加`@EnableCaching`注释来实现:

```
@Configuration
@EnableCaching
public class MultipleCacheManagerConfig {}
```

然后，我们可以通过在方法上添加`@Cacheable `注释来开始缓存方法的输出:

```
@Cacheable(cacheNames = "customers")
public Customer getCustomerDetail(Integer customerId) {
    return customerDetailRepository.getCustomerDetail(customerId);
}
```

一旦我们添加了上面的配置，Spring Boot 本身就会为我们创建一个缓存管理器。

**默认情况下，如果我们没有明确指定任何其他的**，它使用`ConcurrentHashMap`作为底层缓存。

## 3.配置多个高速缓存管理器

在某些情况下，我们可能需要在应用程序中使用多个缓存管理器。因此，让我们通过一个例子来看看如何在我们的 Spring Boot 应用程序中做到这一点。

**在我们的例子中，我们将使用一个`CaffeineCacheManager`和一个简单的`ConcurrentMapCacheManager`T3。**

`CaffeineCacheManager `是由 [`spring-boot-starter-cache`](https://web.archive.org/web/20221126213538/https://search.maven.org/artifact/org.springframework.boot/spring-boot-starter-cache) 提供的启动器。如果 [`Caffeine`](/web/20221126213538/https://www.baeldung.com/java-caching-caffeine) 存在，它将由 Spring 自动配置，这是一个用 Java 8 编写的缓存库。

`ConcurrentMapCacheManager `使用 C 语言`oncurrentHashMap`实现缓存。

我们可以用下面的方法做到这一点。

### 3.1.使用`@Primary`

我们可以在配置类中创建两个缓存管理器 beans。然后，我们可以将一个 bean 设为主要 bean:

```
@Configuration
@EnableCaching
public class MultipleCacheManagerConfig {

    @Bean
    @Primary
    public CacheManager cacheManager() {
        CaffeineCacheManager cacheManager = new CaffeineCacheManager("customers", "orders");
        cacheManager.setCaffeine(Caffeine.newBuilder()
          .initialCapacity(200)
          .maximumSize(500)
          .weakKeys()
          .recordStats());
        return cacheManager;
    }

    @Bean
    public CacheManager alternateCacheManager() {
        return new ConcurrentMapCacheManager("customerOrders", "orderprice");
    }
}
```

现在，Spring Boot 将使用`CaffeineCacheManager`作为所有方法的默认值，直到我们为一个方法明确指定我们的`alternateCacheManager`:

```
@Cacheable(cacheNames = "customers")
public Customer getCustomerDetail(Integer customerId) {
    return customerDetailRepository.getCustomerDetail(customerId);
}

@Cacheable(cacheNames = "customerOrders", cacheManager = "alternateCacheManager")
public List<Order> getCustomerOrders(Integer customerId) {
    return customerDetailRepository.getCustomerOrders(customerId);
}
```

在上面的例子中，我们的应用程序将使用`CaffeineCacheManager`作为`getCustomerDetail()`方法。对于`getCustomerOrders()`方法，它将使用`alternateCacheManager. `

### 3.2.延伸`CachingConfigurerSupport`

另一种方法是扩展`CachingConfigurerSupport`类并覆盖`cacheManager`()方法。该方法返回一个 bean，它将成为我们的应用程序的默认缓存管理器:

```
@Configuration
@EnableCaching
public class MultipleCacheManagerConfig extends CachingConfigurerSupport {

    @Bean
    public CacheManager cacheManager() {
        CaffeineCacheManager cacheManager = new CaffeineCacheManager("customers", "orders");
        cacheManager.setCaffeine(Caffeine.newBuilder()
          .initialCapacity(200)
          .maximumSize(500)
          .weakKeys()
          .recordStats());
        return cacheManager;
    }

    @Bean
    public CacheManager alternateCacheManager() {
        return new ConcurrentMapCacheManager("customerOrders", "orderprice");
    }
}
```

注意，我们仍然可以创建另一个名为`alternateCacheManager.` 的 bean，我们可以通过显式地指定它来将这个`alternateCacheManager`用于一个方法，就像我们在上一个例子中所做的那样。

### 3.3.使用`CacheResolver`

我们可以实现`CacheResolver`接口并创建一个自定义的`CacheResolver`:

```
public class MultipleCacheResolver implements CacheResolver {

    private final CacheManager simpleCacheManager;
    private final CacheManager caffeineCacheManager;    
    private static final String ORDER_CACHE = "orders";    
    private static final String ORDER_PRICE_CACHE = "orderprice";

    public MultipleCacheResolver(CacheManager simpleCacheManager,CacheManager caffeineCacheManager) {
        this.simpleCacheManager = simpleCacheManager;
        this.caffeineCacheManager=caffeineCacheManager;

    }

    @Override
    public Collection<? extends Cache> resolveCaches(CacheOperationInvocationContext<?> context) {
        Collection<Cache> caches = new ArrayList<Cache>();
        if ("getOrderDetail".equals(context.getMethod().getName())) {
            caches.add(caffeineCacheManager.getCache(ORDER_CACHE));
        } else {
            caches.add(simpleCacheManager.getCache(ORDER_PRICE_CACHE));
        }
        return caches;
    }
}
```

在这种情况下，我们必须覆盖`CacheResolver`接口的`resolveCaches`方法。

在我们的例子中，我们根据方法名选择一个缓存管理器。在这之后，我们需要创建一个自定义的 bean`CacheResolver`:

```
@Configuration
@EnableCaching
public class MultipleCacheManagerConfig extends CachingConfigurerSupport {

    @Bean
    public CacheManager cacheManager() {
        CaffeineCacheManager cacheManager = new CaffeineCacheManager("customers", "orders");
        cacheManager.setCaffeine(Caffeine.newBuilder()
          .initialCapacity(200)
          .maximumSize(500)
          .weakKeys()
          .recordStats());
        return cacheManager;
    }

    @Bean
    public CacheManager alternateCacheManager() {
        return new ConcurrentMapCacheManager("customerOrders", "orderprice");
    }

    @Bean
    public CacheResolver cacheResolver() {
        return new MultipleCacheResolver(alternateCacheManager(), cacheManager());
    }
}
```

现在我们可以使用自定义的`CacheResolver`为我们的方法解析一个缓存管理器:

```
@Component
public class OrderDetailBO {

    @Autowired
    private OrderDetailRepository orderDetailRepository;

    @Cacheable(cacheNames = "orders", cacheResolver = "cacheResolver")
    public Order getOrderDetail(Integer orderId) {
        return orderDetailRepository.getOrderDetail(orderId);
    }

    @Cacheable(cacheNames = "orderprice", cacheResolver = "cacheResolver")
    public double getOrderPrice(Integer orderId) {
        return orderDetailRepository.getOrderPrice(orderId);
    }
}
```

这里，我们在`cacheResolver `元素中传递我们的`CacheResolver` bean 的名称。

## 4.结论

在本文中，我们学习了如何在 Spring Boot 应用程序中启用缓存。然后，我们学习了在应用程序中使用多个缓存管理器的三种方法。

和往常一样，这些例子的代码可以在 GitHub 的[上找到。](https://web.archive.org/web/20221126213538/https://github.com/eugenp/tutorials/tree/master/spring-caching)