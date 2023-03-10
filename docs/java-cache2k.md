# cache2k 简介

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-cache2k>

## 1.概观

在本教程中，我们将了解一下 [cache2k](https://web.archive.org/web/20221129011727/https://cache2k.org/) —一个轻量级、高性能、内存中的 Java 缓存库。

## 2.关于`cache2k`

由于对缓存值的非阻塞和无等待访问，cache2k 库提供了快速的访问时间。它还支持与 Spring Framework、Scala Cache、Datanucleus 和 Hibernate 的集成。

该库提供了许多功能，包括一组**线程安全原子操作**，一个具有阻塞 **通读**、**自动到期**、**提前刷新、事件监听器**，以及对 JSR107 API 的 [JCache](/web/20221129011727/https://www.baeldung.com/jcache) 实现的支持。我们将在本教程中讨论其中的一些特性。

需要注意的是，cache2k 不是像 [Infispan](/web/20221129011727/https://www.baeldung.com/infinispan) 或 [Hazelcast](/web/20221129011727/https://www.baeldung.com/java-hazelcast) 那样的分布式缓存解决方案。

## 3.Maven 依赖性

要使用 cache2k，我们需要首先将 [`cache2k-base-bom`依赖](https://web.archive.org/web/20221129011727/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.cache2k%22%20AND%20a%3A%22cache2k-base-bom%22)添加到我们的`pom.xml`中:

```java
<dependency>
    <groupId>org.cache2k</groupId>
    <artifactId>cache2k-base-bom</artifactId>
    <version>1.2.3.Final</version>
    <type>pom</type>
</dependency>
```

## 4.一个简单的`cache2k`例子

现在，让我们借助一个简单的例子来看看如何在 Java 应用程序中使用 cache2k。

让我们考虑一个在线购物网站的例子。让我们假设该网站对所有体育产品提供八折优惠，对其他产品提供九折优惠。我们的目标是缓存折扣，这样我们就不用每次都计算它。

因此，首先，我们将创建一个`ProductHelper`类，并创建一个简单的缓存实现:

```java
public class ProductHelper {

    private Cache<String, Integer> cachedDiscounts;
    private int cacheMissCount = 0;

    public ProductHelper() {
        cachedDiscounts = Cache2kBuilder.of(String.class, Integer.class)
          .name("discount")
          .eternal(true)
          .entryCapacity(100)
          .build();
    }

    public Integer getDiscount(String productType) {
        Integer discount = cachedDiscounts.get(productType);
        if (Objects.isNull(discount)) {
            cacheMissCount++;
            discount = "Sports".equalsIgnoreCase(productType) ? 20 : 10;
            cachedDiscounts.put(productType, discount);
        }
        return discount;
    }

    // Getters and setters

}
```

如我们所见，我们使用了一个`cacheMissCount`变量来计算在缓存中找不到折扣的次数。所以，如果`getDiscount`方法使用缓存来获得折扣，`cacheMissCount`不会改变。

接下来，我们将编写一个测试用例并验证我们的实现:

```java
@Test
public void whenInvokedGetDiscountTwice_thenGetItFromCache() {
    ProductHelper productHelper = new ProductHelper();
    assertTrue(productHelper.getCacheMissCount() == 0);

    assertTrue(productHelper.getDiscount("Sports") == 20);
    assertTrue(productHelper.getDiscount("Sports") == 20);

    assertTrue(productHelper.getCacheMissCount() == 1);
}
```

最后，让我们快速看一下我们使用过的配置。

第一个是`name`方法，该方法**设置我们的缓存**的唯一名称。缓存名称是可选的，如果我们不提供它，它就会生成。

然后，我们将 **`eternal`设置为`true`** 以指示**缓存的值不会随着时间而到期**。因此，在这种情况下，我们可以选择显式地从缓存中删除元素。否则，一旦缓存达到其容量，这些元素将被自动逐出。

此外，我们已经使用了`entryCapacity` 方法来**指定缓存所拥有的条目**的最大数量。当缓存达到最大值时，缓存回收算法将删除一个或多个条目，以保持指定的容量。

我们可以进一步探索 [`Cache2kBuilder`](https://web.archive.org/web/20221129011727/https://cache2k.org/docs/latest/apidocs/cache2k-api/org/cache2k/Cache2kBuilder.html) 类中的其他可用配置。

## 5.`cache2k`特性

现在，让我们增强我们的示例来探索一些 cache2k 特性。

### 5.1.配置缓存过期

到目前为止，我们对所有的体育产品都给予固定的折扣。然而，我们的网站现在希望折扣只在固定的时间内有效。

为了满足这一新需求，我们将使用`expireAfterWrite`方法配置缓存到期时间:

```java
cachedDiscounts = Cache2kBuilder.of(String.class, Integer.class)
  // other configurations
  .expireAfterWrite(10, TimeUnit.MILLISECONDS)
  .build();
```

现在让我们编写一个测试用例来检查缓存是否过期:

```java
@Test
public void whenInvokedGetDiscountAfterExpiration_thenDiscountCalculatedAgain() 
  throws InterruptedException {
    ProductHelper productHelper = new ProductHelper();
    assertTrue(productHelper.getCacheMissCount() == 0);
    assertTrue(productHelper.getDiscount("Sports") == 20);
    assertTrue(productHelper.getCacheMissCount() == 1);

    Thread.sleep(20);

    assertTrue(productHelper.getDiscount("Sports") == 20);
    assertTrue(productHelper.getCacheMissCount() == 2);
}
```

在我们的测试案例中，我们试图在配置的持续时间过后再次获得折扣。我们可以看到，与前面的例子不同，`cacheMissCount`已经增加了。这是因为缓存中的项目已过期，折扣会重新计算。

对于高级缓存到期配置，我们也可以配置一个`[ExpiryPolicy](https://web.archive.org/web/20221129011727/https://cache2k.org/docs/latest/apidocs/cache2k-api/org/cache2k/expiry/ExpiryPolicy.html)`。

### 5.2.缓存加载或通读

在我们的例子中，我们使用了缓存备用模式来加载缓存。这意味着我们已经在`getDiscount`方法中计算并在按需缓存中添加了折扣。

或者，我们可以简单地**使用 cache2k 支持读通操作**。在这个操作中，**缓存将在加载器**的帮助下自己加载丢失的值。这也称为缓存加载。

现在，让我们进一步增强我们的示例，以自动计算和加载缓存:

```java
cachedDiscounts = Cache2kBuilder.of(String.class, Integer.class)
  // other configurations
  .loader((key) -> {
      cacheMissCount++;
      return "Sports".equalsIgnoreCase(key) ? 20 : 10;
  })
  .build();
```

此外，我们将从`getDiscount`中删除计算和更新折扣的逻辑:

```java
public Integer getDiscount(String productType) {
    return cachedDiscounts.get(productType);
}
```

之后，让我们编写一个测试用例来确保加载器按预期工作:

```java
@Test
public void whenInvokedGetDiscount_thenPopulateCacheUsingLoader() {
    ProductHelper productHelper = new ProductHelper();
    assertTrue(productHelper.getCacheMissCount() == 0);

    assertTrue(productHelper.getDiscount("Sports") == 20);
    assertTrue(productHelper.getCacheMissCount() == 1);

    assertTrue(productHelper.getDiscount("Electronics") == 10);
    assertTrue(productHelper.getCacheMissCount() == 2);
}
```

### 5.3.事件监听器

我们还可以为不同的缓存操作配置事件侦听器，如缓存元素的插入、更新、移除和过期。

假设我们想要记录缓存中添加的所有条目。因此，让我们在缓存构建器中添加一个事件监听器配置:

```java
.addListener(new CacheEntryCreatedListener<String, Integer>() {
    @Override
    public void onEntryCreated(Cache<String, Integer> cache, CacheEntry<String, Integer> entry) {
        LOGGER.info("Entry created: [{}, {}].", entry.getKey(), entry.getValue());
    }
})
```

现在，我们可以执行我们已经创建的任何测试用例，并验证日志:

```java
Entry created: [Sports, 20].
```

值得注意的是，除了到期事件之外，**事件监听器同步执行。如果我们想要一个异步监听器，我们可以使用`addAsyncListener`方法。**

### 5.4.原子操作

[`Cache`](https://web.archive.org/web/20221129011727/https://cache2k.org/docs/latest/apidocs/cache2k-api/org/cache2k/Cache.html) 类有许多支持原子操作的方法。这些方法仅适用于单个条目的操作。

这些方法包括`containsAndRemove``,``putIfAbsent``,``removeIfEquals``,``replaceIfEquals` `peekAndReplace`和 `peekAndPut`。

## 6.结论

在本教程中，我们研究了 cache2k 库及其一些有用的特性。我们可以参考 [cache2k 用户指南](https://web.archive.org/web/20221129011727/https://cache2k.org/docs/latest/user-guide.html)来进一步探索这个库。

和往常一样，本教程的完整代码可以在 GitHub 上找到[。](https://web.archive.org/web/20221129011727/https://github.com/eugenp/tutorials/tree/master/libraries-3)