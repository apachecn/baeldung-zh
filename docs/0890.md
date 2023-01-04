# Spring 缓存指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-cache-tutorial>

## 1。缓存抽象？

在本教程中，我们将学习如何**使用 Spring** 中的缓存抽象，并从总体上提高我们系统的性能。

我们将为一些真实世界的方法示例启用简单的缓存，并且我们将讨论如何通过智能缓存管理来实际提高这些调用的性能。

## 延伸阅读:

## [Spring Boot Ehcache 示例](/web/20221024020308/https://www.baeldung.com/spring-boot-ehcache)

A quick and practical guide to using Spring with Ehcache.[Read more](/web/20221024020308/https://www.baeldung.com/spring-boot-ehcache) →

## [Spring Boot 的缓存回收](/web/20221024020308/https://www.baeldung.com/spring-boot-evict-cache)

Learn how to invalidate caches with Spring Boot.[Read more](/web/20221024020308/https://www.baeldung.com/spring-boot-evict-cache) →

## 2。入门

Spring 提供的核心缓存抽象驻留在`[spring-context](https://web.archive.org/web/20221024020308/https://search.maven.org/search?q=g:org.springframework%20a:spring-context) `模块中。所以在使用 Maven 时，我们的`pom.xml`应该包含以下依赖关系:

```
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-context</artifactId>
    <version>5.3.3</version>
</dependency>
```

有趣的是，还有另一个名为`[spring-context-support](https://web.archive.org/web/20221024020308/https://search.maven.org/search?q=g:org.springframework%20a:spring-context-support),` 的模块，它位于`spring-context `模块之上，提供更多的`CacheManagers `，由类似 [EhCache](/web/20221024020308/https://www.baeldung.com/spring-boot-ehcache) 或[咖啡因](/web/20221024020308/https://www.baeldung.com/java-caching-caffeine)支持。如果我们想将它们用作缓存存储，那么我们需要使用`spring-context-support `模块:

```
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-context-support</artifactId>
    <version>5.3.3</version>
</dependency>
```

因为`spring-context-support` 模块依赖于`spring-context `模块，所以不需要为`spring-context.`单独声明依赖关系

### 2.1。Spring Boot

如果我们使用 Spring Boot，那么我们可以利用`[spring-boot-starter-cache](https://web.archive.org/web/20221024020308/https://search.maven.org/search?q=g:org.springframework.boot%20a:spring-boot-starter-cache) ` starter 包轻松添加缓存依赖项:

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-cache</artifactId>
    <version>2.4.0</version>
</dependency>
```

引擎盖下，启动器带来了`spring-context-support `模块。

## 3。启用缓存

为了启用缓存，Spring 很好地利用了注释，就像启用框架中任何其他配置级别的特性一样。

我们可以简单地通过向任何配置类添加`@EnableCaching`注释来启用缓存特性:

```
@Configuration
@EnableCaching
public class CachingConfig {

    @Bean
    public CacheManager cacheManager() {
        return new ConcurrentMapCacheManager("addresses");
    }
}
```

当然，我们也可以通过 XML 配置来实现缓存管理:

```
<beans>
    <cache:annotation-driven />

    <bean id="cacheManager" class="org.springframework.cache.support.SimpleCacheManager">
        <property name="caches">
            <set>
                <bean 
                  class="org.springframework.cache.concurrent.ConcurrentMapCacheFactoryBean" 
                  name="addresses"/>
            </set>
        </property>
    </bean>
</beans>
```

注意:**在我们启用缓存后，对于最小设置，我们必须注册一个`cacheManager`。**

### 3.1。使用 Spring Boot

当使用 Spring Boot 时，只要 starter 包出现在类路径中的`EnableCaching `注释旁边，就会注册同一个`ConcurrentMapCacheManager.`，所以不需要单独的 bean 声明。

此外，我们可以使用一个或多个`CacheManagerCustomizer<T> `bean 定制自动配置的`CacheManager `[:](https://web.archive.org/web/20221024020308/https://github.com/spring-projects/spring-boot/blob/master/spring-boot-project/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/cache/CacheAutoConfiguration.java)

```
@Component
public class SimpleCacheCustomizer 
  implements CacheManagerCustomizer<ConcurrentMapCacheManager> {

    @Override
    public void customize(ConcurrentMapCacheManager cacheManager) {
        cacheManager.setCacheNames(asList("users", "transactions"));
    }
}
```

在完成初始化之前，`[CacheAutoConfiguration](https://web.archive.org/web/20221024020308/https://github.com/spring-projects/spring-boot/blob/master/spring-boot-project/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/cache/CacheAutoConfiguration.java) `自动配置选择这些定制器并将其应用到当前的`CacheManager `。

## 4。使用带注释的缓存

一旦我们启用了缓存，下一步就是用声明性注释将缓存行为绑定到方法。

### 4.1。@ `Cacheable`

为一个方法启用缓存行为的最简单的方法是用`@Cacheable`来区分它，并用存储结果的缓存的名称来参数化它:

```
@Cacheable("addresses")
public String getAddress(Customer customer) {...} 
```

在实际调用方法之前，`getAddress()`调用将首先检查缓存`addresses`,然后缓存结果。

虽然在大多数情况下一个缓存就足够了，但是 Spring 框架也支持将多个缓存作为参数传递:

```
@Cacheable({"addresses", "directory"})
public String getAddress(Customer customer) {...}
```

在这种情况下，如果任何缓存包含所需的结果，则返回结果，并且不调用该方法。

### 4.2。**@`CacheEvict`**

现在，让所有方法都成为`@Cacheable`会有什么问题呢？

问题在于规模。 **W** **我们不想在缓存中填入我们不经常需要的值**。缓存可能会变得非常大，非常快，我们可能会保留大量陈旧或未使用的数据。

我们可以使用`@CacheEvict`注释来指示删除一个或多个/所有值，以便新值可以再次加载到缓存中:

```
@CacheEvict(value="addresses", allEntries=true)
public String getAddress(Customer customer) {...}
```

这里我们使用了附加参数`allEntries`和要清空的缓存；这将清除缓存`addresses`中的所有条目，并为新数据做准备。

### *4.3。@ `CachePut`*

 *虽然`@CacheEvict`通过删除陈旧和未使用的条目来减少在大型缓存中查找条目的开销，但我们希望**避免从缓存中驱逐太多数据**。

相反，每当我们改变条目时，我们有选择地更新它们。

使用`@CachePut`注释，我们可以在不干扰方法执行的情况下更新缓存的内容。也就是说，将始终执行该方法，并缓存结果:

```
@CachePut(value="addresses")
public String getAddress(Customer customer) {...}
```

`@Cacheable`和`@CachePut`的区别在于`@Cacheable`将**跳过运行方法**，而`@CachePut`将**实际运行方法**，然后将其结果放入缓存。

### 4.4。@ `Caching`

如果我们想使用同一类型的多个注释来缓存一个方法，该怎么办？让我们看一个不正确的例子:

```
@CacheEvict("addresses")
@CacheEvict(value="directory", key=customer.name)
public String getAddress(Customer customer) {...}
```

上面的代码将无法编译，因为 Java 不允许为一个给定的方法声明多个相同类型的注释。

上述问题的解决方法是:

```
@Caching(evict = { 
  @CacheEvict("addresses"), 
  @CacheEvict(value="directory", key="#customer.name") })
public String getAddress(Customer customer) {...}
```

如上面的代码片段所示，我们可以用**将多个缓存注释**与`@Caching`分组，并使用它来实现我们自己定制的缓存逻辑。

### 4.5。@ `CacheConfig`

有了`@CacheConfig`注释，我们可以**将一些缓存配置精简到类级别的一个地方，**，这样我们就不必多次声明:

```
@CacheConfig(cacheNames={"addresses"})
public class CustomerDataService {

    @Cacheable
    public String getAddress(Customer customer) {...}
```

## 5。条件缓存

有时，缓存可能并不适用于所有情况下的方法。

重用我们在`@CachePut`注释中的例子，这将执行方法并每次缓存结果:

```
@CachePut(value="addresses")
public String getAddress(Customer customer) {...} 
```

### 5.1。条件参数

如果我们希望对注释何时激活有更多的控制，我们可以用一个条件参数来参数化`@CachePut`,该条件参数采用一个 SpEL 表达式，并确保基于对该表达式的求值来缓存结果:

```
@CachePut(value="addresses", condition="#customer.name=='Tom'")
public String getAddress(Customer customer) {...}
```

### 5.2。除非参数

我们也可以通过`unless`参数基于方法的输出而不是输入来控制缓存**:**

```
@CachePut(value="addresses", unless="#result.length()<64")
public String getAddress(Customer customer) {...}
```

除非地址短于 64 个字符，否则上面的注释会缓存地址。

重要的是要知道`condition`和`unless`参数可以与所有缓存注释结合使用。

事实证明，这种条件缓存对于管理大型结果非常有效。它对于基于输入参数定制行为也很有用，而不是对所有操作强制一个通用的行为。

## 6。基于 XML 的声明式缓存

如果我们无法访问应用程序的源代码，或者想从外部注入缓存行为，我们也可以使用声明式的基于 XML 的缓存。

下面是我们的 XML 配置:

```
<!-- the service that you wish to make cacheable -->
<bean id="customerDataService" 
  class="com.your.app.namespace.service.CustomerDataService"/>

<bean id="cacheManager" 
  class="org.springframework.cache.support.SimpleCacheManager"> 
    <property name="caches"> 
        <set> 
            <bean 
              class="org.springframework.cache.concurrent.ConcurrentMapCacheFactoryBean" 
              name="directory"/> 
            <bean 
              class="org.springframework.cache.concurrent.ConcurrentMapCacheFactoryBean" 
              name="addresses"/> 
        </set> 
    </property> 
</bean>
<!-- define caching behavior -->
<cache:advice id="cachingBehavior" cache-manager="cacheManager">
    <cache:caching cache="addresses">
        <cache:cacheable method="getAddress" key="#customer.name"/>
    </cache:caching>
</cache:advice>

<!-- apply the behavior to all the implementations of CustomerDataService interface->
<aop:config>
    <aop:advisor advice-ref="cachingBehavior"
      pointcut="execution(* com.your.app.namespace.service.CustomerDataService.*(..))"/>
</aop:config>
```

## 7。基于 Java 的缓存

下面是等效的 Java 配置:

```
@Configuration
@EnableCaching
public class CachingConfig {

    @Bean
    public CacheManager cacheManager() {
        SimpleCacheManager cacheManager = new SimpleCacheManager();
        cacheManager.setCaches(Arrays.asList(
          new ConcurrentMapCache("directory"), 
          new ConcurrentMapCache("addresses")));
        return cacheManager;
    }
}
```

这是我们的`CustomerDataService`:

```
@Component
public class CustomerDataService {

    @Cacheable(value = "addresses", key = "#customer.name")
    public String getAddress(Customer customer) {
        return customer.getAddress();
    }
}
```

## 8。总结

在本文中，我们讨论了 Spring 中缓存的基础，以及如何通过注释很好地利用这种抽象。

本文的完整实现可以在 GitHub 项目中找到。*