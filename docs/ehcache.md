# Ehcache 简介

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/ehcache>

## 1。概述

在本文中，我们将介绍 [Ehcache](https://web.archive.org/web/20220926202519/http://www.ehcache.org/) ，这是一个广泛使用的、基于 Java 的开源缓存。它具有内存和磁盘存储、监听器、缓存加载器、RESTful 和 SOAP APIs 以及其他非常有用的功能。

为了展示缓存如何优化我们的应用程序，我们将创建一个简单的方法来计算所提供数字的平方值。在每次调用时，该方法将调用`calculateSquareOfNumber(int number)`方法并将信息消息打印到控制台。

在这个简单的例子中，我们想展示平方值的计算只进行一次，并且具有相同输入值的所有其他调用都从缓存中返回结果。

需要注意的是，我们完全专注于 Ehcache 本身(没有 Spring)；如果你想了解 Ehcache 如何与 Spring 一起工作，可以看看 read [这篇文章](/web/20220926202519/https://www.baeldung.com/spring-cache-tutorial)。

## 2。Maven 依赖关系

为了使用 Ehcache，我们需要添加这个 Maven 依赖项:

```java
<dependency>
    <groupId>org.ehcache</groupId>
    <artifactId>ehcache</artifactId>
    <version>3.1.3</version>
</dependency>
```

Ehcache 工件的最新版本可以在[这里](https://web.archive.org/web/20220926202519/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22org.ehcache%22)找到。

## 3。缓存配置

Ehcache 有两种配置方式:

*   第一种方式是通过 Java POJO，其中所有配置参数都是通过 Ehcache API 配置的
*   第二种方式是通过 XML 文件配置，我们可以根据[提供的模式定义](https://web.archive.org/web/20220926202519/http://www.ehcache.org/documentation/3.1/xsds.html#core)配置 Ehcache

在本文中，我们将展示这两种方法——Java 和 XML 配置。

### 3.1。Java 配置

这一小节将展示用 POJOs 配置 Ehcache 是多么容易。此外，我们将创建一个助手类来简化缓存配置和可用性:

```java
public class CacheHelper {

    private CacheManager cacheManager;
    private Cache<Integer, Integer> squareNumberCache;

    public CacheHelper() {
        cacheManager = CacheManagerBuilder
          .newCacheManagerBuilder().build();
        cacheManager.init();

        squareNumberCache = cacheManager
          .createCache("squaredNumber", CacheConfigurationBuilder
            .newCacheConfigurationBuilder(
              Integer.class, Integer.class,
              ResourcePoolsBuilder.heap(10)));
    }

    public Cache<Integer, Integer> getSquareNumberCacheFromCacheManager() {
        return cacheManager.getCache("squaredNumber", Integer.class, Integer.class);
    }

    // standard getters and setters
}
```

为了初始化我们的缓存，首先，我们需要定义 Ehcache `CacheManager`对象。在这个例子中，我们用`newCacheManagerBuilder()` API `.`创建了一个默认缓存 `squaredNumber”`

缓存将简单地将`Integer`键映射到`Integer`值。

注意，在我们开始使用定义的缓存之前，我们需要用`init()`方法初始化`CacheManager`对象。

最后，为了获得我们的缓存，我们可以使用提供了名称、键和缓存值类型的`getCache()` API。

用这几行代码，我们创建了第一个缓存，现在可以用于我们的应用程序。

### 3.2。XML 配置

3.1 小节中的配置对象。等同于使用以下 XML 配置:

```java
<cache-template name="squaredNumber">
    <key-type>java.lang.Integer</key-type>
    <value-type>java.lang.Integer</value-type>
    <heap unit="entries">10</heap>
</cache-template>
```

为了在 Java 应用程序中包含这个缓存，我们需要用 Java 读取 XML 配置文件:

```java
URL myUrl = getClass().getResource(xmlFile); 
XmlConfiguration xmlConfig = new XmlConfiguration(myUrl); 
CacheManager myCacheManager = CacheManagerBuilder
  .newCacheManager(xmlConfig);
```

## 4。Ehcache 测试

在第三节。我们展示了如何为您的目的定义简单缓存。为了展示缓存实际工作，我们将创建`SquaredCalculator`类，它将计算所提供输入的平方值，并将计算值存储在缓存中。

当然，如果缓存已经包含计算值，我们将返回缓存值并避免不必要的计算:

```java
public class SquaredCalculator {
    private CacheHelper cache;

    public int getSquareValueOfNumber(int input) {
        if (cache.getSquareNumberCache().containsKey(input)) {
            return cache.getSquareNumberCache().get(input);
        }

        System.out.println("Calculating square value of " + input + 
          " and caching result.");

        int squaredValue = (int) Math.pow(input, 2);
        cache.getSquareNumberCache().put(input, squaredValue);

        return squaredValue;
    }

    //standard getters and setters;
}
```

为了完成我们的测试场景，我们还需要计算平方值的代码:

```java
@Test
public void whenCalculatingSquareValueAgain_thenCacheHasAllValues() {
    for (int i = 10; i < 15; i++) {
        assertFalse(cacheHelper.getSquareNumberCache().containsKey(i));
        System.out.println("Square value of " + i + " is: "
          + squaredCalculator.getSquareValueOfNumber(i) + "\n");
    }      

    for (int i = 10; i < 15; i++) {
        assertTrue(cacheHelper.getSquareNumberCache().containsKey(i));
        System.out.println("Square value of " + i + " is: "
          + squaredCalculator.getSquareValueOfNumber(i) + "\n");
    }
}
```

如果我们运行我们的测试，我们将在控制台中得到这个结果:

```java
Calculating square value of 10 and caching result.
Square value of 10 is: 100

Calculating square value of 11 and caching result.
Square value of 11 is: 121

Calculating square value of 12 and caching result.
Square value of 12 is: 144

Calculating square value of 13 and caching result.
Square value of 13 is: 169

Calculating square value of 14 and caching result.
Square value of 14 is: 196

Square value of 10 is: 100
Square value of 11 is: 121
Square value of 12 is: 144
Square value of 13 is: 169
Square value of 14 is: 196
```

正如您所注意到的，`calculate()`方法只在第一次调用时进行计算。在第二次调用时，所有值都在缓存中找到并从缓存中返回。

## 5。其他 Ehcache 配置选项

当我们在前一个例子中创建缓存时，它是一个没有任何特殊选项的简单缓存。本节将展示在创建缓存时有用的其他选项。

### 5.1。磁盘持久性

如果有太多的值要存储到缓存中，我们可以在硬盘上存储一些值。

```java
PersistentCacheManager persistentCacheManager = 
  CacheManagerBuilder.newCacheManagerBuilder()
    .with(CacheManagerBuilder.persistence(getStoragePath()
      + File.separator 
      + "squaredValue")) 
    .withCache("persistent-cache", CacheConfigurationBuilder
      .newCacheConfigurationBuilder(Integer.class, Integer.class,
        ResourcePoolsBuilder.newResourcePoolsBuilder()
          .heap(10, EntryUnit.ENTRIES)
          .disk(10, MemoryUnit.MB, true)) 
      )
  .build(true);

persistentCacheManager.close();
```

代替默认的`CacheManager`，我们现在使用`PersistentCacheManager`，它将保存所有不能保存到内存中的值。

从配置中，我们可以看到缓存将 10 个元素保存到内存中，并将在硬盘上分配 10MB 用于持久存储。

### 5.2。数据到期

如果我们缓存大量数据，很自然地我们会将缓存的数据保存一段时间，这样我们就可以避免大量的内存使用。

Ehcache 通过`Expiry`接口控制数据刷新:

```java
CacheConfiguration<Integer, Integer> cacheConfiguration 
  = CacheConfigurationBuilder
    .newCacheConfigurationBuilder(Integer.class, Integer.class, 
      ResourcePoolsBuilder.heap(100)) 
    .withExpiry(Expirations.timeToLiveExpiration(Duration.of(60, 
      TimeUnit.SECONDS))).build();
```

在该缓存中，所有数据将存在 60 秒，在这段时间后，数据将从内存中删除。

## 6。结论

在本文中，我们展示了如何在 Java 应用程序中使用简单的 Ehcache 缓存。

在我们的示例中，我们看到即使是简单配置的缓存也可以节省大量不必要的操作。此外，我们展示了我们可以通过 POJOs 和 XML 配置缓存，Ehcache 有一些很好的特性——比如持久性和数据过期。

和往常一样，这篇文章的代码可以在 GitHub 上找到[。](https://web.archive.org/web/20220926202519/https://github.com/eugenp/tutorials/tree/master/spring-caching)