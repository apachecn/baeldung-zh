# 设置缓存的生存时间值

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-setting-ttl-value-cache>

## 1.概观

在本教程中，我们为一些基本的现实世界的例子做缓存。特别是，我们将演示如何将这种缓存机制配置为有时间限制的。我们还将这种时间限制称为缓存的生存时间(TTL)。

## 2.Spring 缓存的配置

之前，我们已经用[展示了如何使用来自 Spring](/web/20221023070030/https://www.baeldung.com/spring-cache-tutorial) 的`@Cacheable`注释。同时，缓存的一个实际用例是当一个`Hotel`预订网站的主页被频繁打开时。这意味着提供一个`Hotels`列表的 REST 端点经常被请求，频繁地调用数据库。与直接从内存中提供数据相比，数据库调用要慢一些。

首先，我们将创建`SpringCachingConfig`:

```
@Configuration
@EnableCaching
public class SpringCachingConfig {

    @Bean
    public CacheManager cacheManager() {
        return new ConcurrentMapCacheManager("hotels");
    }
}
```

我们还需要`SpringCacheCustomizer`:

```
@Component
public class SpringCacheCustomizer implements CacheManagerCustomizer<ConcurrentMapCacheManager> {

    @Override
    public void customize(ConcurrentMapCacheManager cacheManager) {
        cacheManager.setCacheNames(asList("hotels"));
    }
}
```

## 3.@ `Cacheable`缓存

设置完成后，我们就可以使用 Spring 配置了。我们可以通过在内存中存储`Hotel`对象来减少 REST 端点响应时间。我们使用`@Cacheable`注释来缓存一列`Hotel`对象，如下面的代码片段所示:

```
@Cacheable("hotels")
public List<Hotel> getAllHotels() {
    return hotelRepository.getAllHotels();
}
```

## 4.为`@Cacheable`设置 TLL

但是，由于更新、删除或添加，数据库中缓存的`Hotels`列表可能会随着时间而改变。我们希望通过设置生存时间间隔(TTL)来刷新缓存，在此之后，现有的缓存条目将被删除，并在第一次调用上面第 3 节中的方法时重新填充。我们可以通过使用`@CacheEvict`注释来做到这一点。例如，在下面的例子中，我们通过`caching.hotelCacheTTL`变量设置 TTL:

```
@CacheEvict(value = "hotels", allEntries = true)
@Scheduled(fixedRateString = "${caching.hotelListTTL}")
public void emptyHotelsCache() {
    logger.info("emptying Hotels cache");
}
```

我们希望 TTL 为 12 小时。以毫秒为单位的值是 12 x 3600 x 1000 = 43200000。我们在环境属性中定义了这一点。此外，如果我们使用基于属性的环境配置，我们可以如下设置缓存 TTL:

```
caching.spring.hotelListTTL=43200000
```

或者，如果我们使用基于 YAML 的设计，我们可以将其设置为:

```
caching:
  spring:
    hotelListTTL: 43200000
```

## 5.结论

在本文中，我们探讨了如何为基于 Spring 的缓存设置 TTL 缓存。和往常一样，我们可以在 GitHub 上找到完整的代码[。](https://web.archive.org/web/20221023070030/https://github.com/eugenp/tutorials/tree/master/spring-caching-2)