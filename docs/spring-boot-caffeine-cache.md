# Spring Boot 和咖啡因缓存

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-boot-caffeine-cache>

## 1.概观

[Caffeine cache](/web/20220727020730/https://www.baeldung.com/java-caching-caffeine) 是一个面向 Java 的高性能缓存库。在这个简短的教程中，我们将看到如何使用它与 [Spring Boot](/web/20220727020730/https://www.baeldung.com/spring-boot) 。

## 2.属国

为了开始了解咖啡因和 Spring Boot，我们首先添加了 `[spring-boot-starter-cache](https://web.archive.org/web/20220727020730/https://search.maven.org/artifact/org.springframework.boot/spring-boot-starter-cache)` 和 [`caffeine`](https://web.archive.org/web/20220727020730/https://search.maven.org/artifact/com.github.ben-manes.caffeine/caffeine) 依赖关系:

```java
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-cache</artifactId>
    </dependency>
    <dependency>
        <groupId>com.github.ben-manes.caffeine</groupId>
        <artifactId>caffeine</artifactId>
    </dependency>
</dependencies>
```

这些引入了基本的 Spring 缓存支持，以及咖啡因库。

## 3.配置

现在我们需要在我们的 Spring Boot 应用程序中配置缓存。

首先，我们创建一个`Caffeine` bean。**这是控制缓存行为的主要配置，如过期、缓存大小限制等**:

```java
@Bean
public Caffeine caffeineConfig() {
    return Caffeine.newBuilder().expireAfterWrite(60, TimeUnit.MINUTES);
}
```

接下来，我们需要使用 Spring `CacheManager`接口创建另一个 bean。Caffeine 提供了这个接口的实现，它需要我们上面创建的`Caffeine`对象:

```java
@Bean
public CacheManager cacheManager(Caffeine caffeine) {
    CaffeineCacheManager caffeineCacheManager = new CaffeineCacheManager();
    caffeineCacheManager.setCaffeine(caffeine);
    return caffeineCacheManager;
}
```

最后，我们需要使用`@EnableCaching`注释在 Spring Boot 启用缓存。这可以添加到应用程序中的任何`@Configuration`类中。

## 4.例子

启用缓存并配置为使用 Caffeine 后，让我们看看如何在我们的 Spring Boot 应用程序中使用[缓存](/web/20220727020730/https://www.baeldung.com/spring-cache-tutorial)的几个例子。

**在 Spring Boot 使用缓存的主要方式是通过`@Cacheable`注释**。该注释适用于 Spring bean 的任何方法(甚至是整个类)。它指示注册的缓存管理器将方法调用的结果存储在缓存中。

典型的用法是在服务类内部:

```java
@Service
public class AddressService {
    @Cacheable
    public AddressDTO getAddress(long customerId) {
        // lookup and return result
    }
}
```

使用不带参数的`@Cacheable`注释将迫使 Spring 对缓存和缓存键都使用默认名称。

我们可以通过向注释添加一些参数来覆盖这两种行为:

```java
@Service
public class AddressService {
    @Cacheable(value = "address_cache", key = "customerId")
    public AddressDTO getAddress(long customerId) {
        // lookup and return result
    }
}
```

上面的例子告诉 Spring 使用名为`address_cache`的缓存和缓存键的`customerId`参数。

最后，因为缓存管理器本身是一个 Spring bean，**我们还可以将它自动连接到任何其他 bean 中，并直接使用它**:

```java
@Service
public class AddressService {

    @Autowired
    CacheManager cacheManager;

    public AddressDTO getAddress(long customerId) {
        if(cacheManager.containsKey(customerId)) {
            return cacheManager.get(customerId);
        }

        // lookup address, cache result, and return it
    }
}
```

## 5.结论

在本教程中，我们看到了如何配置 Spring Boot 使用咖啡因缓存，以及如何在我们的应用程序中使用缓存的一些例子。

当然，所有代码示例都位于 GitHub 上的[。](https://web.archive.org/web/20220727020730/https://github.com/eugenp/tutorials/tree/master/spring-boot-modules/spring-boot-libraries)