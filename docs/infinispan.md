# Java 中的 Infinispan 指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/infinispan>

## 1。概述

在本指南中，我们将了解 [Infinispan](https://web.archive.org/web/20220627082753/http://infinispan.org/) ，这是一个内存中的键/值数据存储，它提供了一组比其他同类工具更强大的功能。

为了理解它是如何工作的，我们将构建一个简单的项目来展示最常见的特性，并检查如何使用它们。

## 2。项目设置

为了能够这样使用它，我们需要在我们的`pom.xml`中添加它的依赖项。

最新版本可以在 [Maven Central](https://web.archive.org/web/20220627082753/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.infinispan%22%20AND%20a%3A%22infinispan-core%22) 资源库中找到:

```java
<dependency>
    <groupId>org.infinispan</groupId>
    <artifactId>infinispan-core</artifactId>
    <version>9.1.5.Final</version>
</dependency>
```

从现在开始，所有必要的底层基础设施都将以编程方式处理。

## 3。`CacheManager`设置

`CacheManager`是我们将使用的大多数特性的基础。它充当所有声明的缓存的容器，控制它们的生命周期，并负责全局配置。

Infinispan 提供了一种非常简单的方法来构建`CacheManager`:

```java
public DefaultCacheManager cacheManager() {
    return new DefaultCacheManager();
}
```

现在我们可以用它来建造我们的贮藏所。

## 4。缓存设置

缓存由名称和配置定义。可以使用类`ConfigurationBuilder`构建必要的配置，该类已经在我们的类路径中可用。

为了测试我们的缓存，我们将构建一个简单的方法来模拟一些繁重的查询:

```java
public class HelloWorldRepository {
    public String getHelloWorld() {
        try {
            System.out.println("Executing some heavy query");
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            // ...
            e.printStackTrace();
        }
        return "Hello World!";
    }
}
```

此外，为了能够检查缓存中的变化，Infinispan 提供了一个简单的注释`@Listener`。

在定义我们的缓存时，我们可以传递一些对其内部发生的任何事件感兴趣的对象，Infinispan 会在处理缓存时通知它:

```java
@Listener
public class CacheListener {
    @CacheEntryCreated
    public void entryCreated(CacheEntryCreatedEvent<String, String> event) {
        this.printLog("Adding key '" + event.getKey() 
          + "' to cache", event);
    }

    @CacheEntryExpired
    public void entryExpired(CacheEntryExpiredEvent<String, String> event) {
        this.printLog("Expiring key '" + event.getKey() 
          + "' from cache", event);
    }

    @CacheEntryVisited
    public void entryVisited(CacheEntryVisitedEvent<String, String> event) {
        this.printLog("Key '" + event.getKey() + "' was visited", event);
    }

    @CacheEntryActivated
    public void entryActivated(CacheEntryActivatedEvent<String, String> event) {
        this.printLog("Activating key '" + event.getKey() 
          + "' on cache", event);
    }

    @CacheEntryPassivated
    public void entryPassivated(CacheEntryPassivatedEvent<String, String> event) {
        this.printLog("Passivating key '" + event.getKey() 
          + "' from cache", event);
    }

    @CacheEntryLoaded
    public void entryLoaded(CacheEntryLoadedEvent<String, String> event) {
        this.printLog("Loading key '" + event.getKey() 
          + "' to cache", event);
    }

    @CacheEntriesEvicted
    public void entriesEvicted(CacheEntriesEvictedEvent<String, String> event) {
        StringBuilder builder = new StringBuilder();
        event.getEntries().forEach(
          (key, value) -> builder.append(key).append(", "));
        System.out.println("Evicting following entries from cache: " 
          + builder.toString());
    }

    private void printLog(String log, CacheEntryEvent event) {
        if (!event.isPre()) {
            System.out.println(log);
        }
    }
}
```

在打印我们的消息之前，我们检查被通知的事件是否已经发生，因为对于某些事件类型，Infinispan 发送两个通知:一个在处理之前，一个在处理之后。

现在，让我们构建一个方法来为我们处理缓存创建:

```java
private <K, V> Cache<K, V> buildCache(
  String cacheName, 
  DefaultCacheManager cacheManager, 
  CacheListener listener, 
  Configuration configuration) {

    cacheManager.defineConfiguration(cacheName, configuration);
    Cache<K, V> cache = cacheManager.getCache(cacheName);
    cache.addListener(listener);
    return cache;
}
```

注意我们是如何将一个配置传递给`CacheManager`，然后使用同一个`cacheName`来获取对应于所需缓存的对象。还要注意我们如何通知监听器缓存对象本身。

我们现在将检查五种不同的缓存配置，看看如何设置它们并充分利用它们。

### 4.1。简单缓存

最简单的缓存类型可以在一行中定义，使用我们的方法`buildCache`:

```java
public Cache<String, String> simpleHelloWorldCache(
  DefaultCacheManager cacheManager, 
  CacheListener listener) {
    return this.buildCache(SIMPLE_HELLO_WORLD_CACHE, 
      cacheManager, listener, new ConfigurationBuilder().build());
}
```

我们现在可以建造一个`Service`:

```java
public String findSimpleHelloWorld() {
    String cacheKey = "simple-hello";
    return simpleHelloWorldCache
      .computeIfAbsent(cacheKey, k -> repository.getHelloWorld());
}
```

注意我们如何使用缓存，首先检查想要的条目是否已经被缓存。如果不是，我们需要调用我们的`Repository`，然后缓存它。

让我们在测试中添加一个简单的方法来计时我们的方法:

```java
protected <T> long timeThis(Supplier<T> supplier) {
    long millis = System.currentTimeMillis();
    supplier.get();
    return System.currentTimeMillis() - millis;
}
```

测试它，我们可以检查执行两个方法调用之间的时间:

```java
@Test
public void whenGetIsCalledTwoTimes_thenTheSecondShouldHitTheCache() {
    assertThat(timeThis(() -> helloWorldService.findSimpleHelloWorld()))
      .isGreaterThanOrEqualTo(1000);

    assertThat(timeThis(() -> helloWorldService.findSimpleHelloWorld()))
      .isLessThan(100);
}
```

### 4.2。过期缓存

我们可以定义一个缓存，其中所有的条目都有一个生命周期，换句话说，在给定的时间段后，元素将从缓存中移除。配置非常简单:

```java
private Configuration expiringConfiguration() {
    return new ConfigurationBuilder().expiration()
      .lifespan(1, TimeUnit.SECONDS)
      .build();
}
```

现在，我们使用上面的配置来构建缓存:

```java
public Cache<String, String> expiringHelloWorldCache(
  DefaultCacheManager cacheManager, 
  CacheListener listener) {

    return this.buildCache(EXPIRING_HELLO_WORLD_CACHE, 
      cacheManager, listener, expiringConfiguration());
}
```

最后，从上面的简单缓存中以类似的方法使用它:

```java
public String findSimpleHelloWorldInExpiringCache() {
    String cacheKey = "simple-hello";
    String helloWorld = expiringHelloWorldCache.get(cacheKey);
    if (helloWorld == null) {
        helloWorld = repository.getHelloWorld();
        expiringHelloWorldCache.put(cacheKey, helloWorld);
    }
    return helloWorld;
}
```

让我们再次测试我们的时代:

```java
@Test
public void whenGetIsCalledTwoTimesQuickly_thenTheSecondShouldHitTheCache() {
    assertThat(timeThis(() -> helloWorldService.findExpiringHelloWorld()))
      .isGreaterThanOrEqualTo(1000);

    assertThat(timeThis(() -> helloWorldService.findExpiringHelloWorld()))
      .isLessThan(100);
}
```

运行它，我们看到在快速连续的缓存命中。为了展示过期是相对于它的条目`put`时间的，让我们在我们的条目中强制它:

```java
@Test
public void whenGetIsCalledTwiceSparsely_thenNeitherHitsTheCache()
  throws InterruptedException {

    assertThat(timeThis(() -> helloWorldService.findExpiringHelloWorld()))
      .isGreaterThanOrEqualTo(1000);

    Thread.sleep(1100);

    assertThat(timeThis(() -> helloWorldService.findExpiringHelloWorld()))
      .isGreaterThanOrEqualTo(1000);
}
```

运行测试后，注意在给定时间后我们的条目是如何从缓存中过期的。我们可以通过查看侦听器的打印日志行来确认这一点:

```java
Executing some heavy query
Adding key 'simple-hello' to cache
Expiring key 'simple-hello' from cache
Executing some heavy query
Adding key 'simple-hello' to cache
```

请注意，当我们尝试访问该条目时，它已经过期。Infinispan 会在两个时刻检查过期的条目:当我们试图访问它时，或者当 reaper 线程扫描缓存时。

我们甚至可以在主配置中没有过期的缓存中使用它。方法`put`接受更多的参数:

```java
simpleHelloWorldCache.put(cacheKey, helloWorld, 10, TimeUnit.SECONDS);
```

或者，我们可以给我们的条目一个最大值`idleTime`，而不是一个固定的生命周期:

```java
simpleHelloWorldCache.put(cacheKey, helloWorld, -1, TimeUnit.SECONDS, 10, TimeUnit.SECONDS);
```

对寿命属性使用-1，缓存不会因此而过期，但是当我们将它与 10 秒的`idleTime`结合起来时，我们告诉 Infinispan 让这个条目过期，除非它在这个时间范围内被访问。

### 4.3。缓存驱逐

在 Infinispan 中，我们可以使用`eviction configuration:`来限制给定缓存中的条目数量

```java
private Configuration evictingConfiguration() {
    return new ConfigurationBuilder()
      .memory().evictionType(EvictionType.COUNT).size(1)
      .build();
}
```

在本例中，我们将缓存中的最大条目数限制为一个，这意味着，如果我们试图输入另一个条目，它将被从缓存中清除。

同样，该方法类似于这里已经介绍的方法:

```java
public String findEvictingHelloWorld(String key) {
    String value = evictingHelloWorldCache.get(key);
    if(value == null) {
        value = repository.getHelloWorld();
        evictingHelloWorldCache.put(key, value);
    }
    return value;
}
```

让我们构建我们的测试:

```java
@Test
public void whenTwoAreAdded_thenFirstShouldntBeAvailable() {

    assertThat(timeThis(
      () -> helloWorldService.findEvictingHelloWorld("key 1")))
      .isGreaterThanOrEqualTo(1000);

    assertThat(timeThis(
      () -> helloWorldService.findEvictingHelloWorld("key 2")))
      .isGreaterThanOrEqualTo(1000);

    assertThat(timeThis(
      () -> helloWorldService.findEvictingHelloWorld("key 1")))
      .isGreaterThanOrEqualTo(1000);
}
```

运行测试，我们可以查看我们的监听器活动日志:

```java
Executing some heavy query
Adding key 'key 1' to cache
Executing some heavy query
Evicting following entries from cache: key 1, 
Adding key 'key 2' to cache
Executing some heavy query
Evicting following entries from cache: key 2, 
Adding key 'key 1' to cache
```

检查当我们插入第二个键时，第一个键是如何自动从缓存中移除的，然后，第二个键也被移除，以便再次为第一个键腾出空间。

### 4.4。钝化缓存

`cache passivation`是 Infinispan 的强大特性之一。通过结合钝化和驱逐，我们可以创建一个不占用大量内存的缓存，而不会丢失信息。

让我们来看看钝化配置:

```java
private Configuration passivatingConfiguration() {
    return new ConfigurationBuilder()
      .memory().evictionType(EvictionType.COUNT).size(1)
      .persistence() 
      .passivation(true)    // activating passivation
      .addSingleFileStore() // in a single file
      .purgeOnStartup(true) // clean the file on startup
      .location(System.getProperty("java.io.tmpdir")) 
      .build();
}
```

我们再次强制缓存中只有一个条目，但是告诉 Infinispan 钝化剩余的条目，而不是仅仅删除它们。

让我们看看当我们试图填充多个条目时会发生什么:

```java
public String findPassivatingHelloWorld(String key) {
    return passivatingHelloWorldCache.computeIfAbsent(key, k -> 
      repository.getHelloWorld());
}
```

让我们构建并运行我们的测试:

```java
@Test
public void whenTwoAreAdded_thenTheFirstShouldBeAvailable() {

    assertThat(timeThis(
      () -> helloWorldService.findPassivatingHelloWorld("key 1")))
      .isGreaterThanOrEqualTo(1000);

    assertThat(timeThis(
      () -> helloWorldService.findPassivatingHelloWorld("key 2")))
      .isGreaterThanOrEqualTo(1000);

    assertThat(timeThis(
      () -> helloWorldService.findPassivatingHelloWorld("key 1")))
      .isLessThan(100);
}
```

现在让我们看看我们的听众活动:

```java
Executing some heavy query
Adding key 'key 1' to cache
Executing some heavy query
Passivating key 'key 1' from cache
Evicting following entries from cache: key 1, 
Adding key 'key 2' to cache
Passivating key 'key 2' from cache
Evicting following entries from cache: key 2, 
Loading key 'key 1' to cache
Activating key 'key 1' on cache
Key 'key 1' was visited
```

请注意，只保留一个条目的缓存需要多少步骤。此外，请注意步骤的顺序–钝化、驱逐、加载，然后激活。让我们看看这些步骤意味着什么:

*   **钝化–**我们的条目存储在另一个地方，远离 Infinispan 的主存储器(在本例中是内存)
*   **Eviction–**删除条目，以释放内存并在高速缓存中保留配置的最大条目数
*   **加载–**当尝试访问钝化条目时，Infinispan 会检查其存储的内容，并再次将条目加载到内存中
*   **激活–**现在可以再次在 Infinispan 中访问该条目

### 4.5。交易缓存

Infinispan 附带了强大的事务控制。与数据库类似，当多个线程试图写入同一个条目时，它有助于保持完整性。

让我们看看如何定义具有事务处理能力的缓存:

```java
private Configuration transactionalConfiguration() {
    return new ConfigurationBuilder()
      .transaction().transactionMode(TransactionMode.TRANSACTIONAL)
      .lockingMode(LockingMode.PESSIMISTIC)
      .build();
}
```

为了能够测试它，让我们构建两个方法——一个快速完成事务，另一个需要一段时间:

```java
public Integer getQuickHowManyVisits() {
    TransactionManager tm = transactionalCache
      .getAdvancedCache().getTransactionManager();
    tm.begin();
    Integer howManyVisits = transactionalCache.get(KEY);
    howManyVisits++;
    System.out.println("I'll try to set HowManyVisits to " + howManyVisits);
    StopWatch watch = new StopWatch();
    watch.start();
    transactionalCache.put(KEY, howManyVisits);
    watch.stop();
    System.out.println("I was able to set HowManyVisits to " + howManyVisits + 
      " after waiting " + watch.getTotalTimeSeconds() + " seconds");

    tm.commit();
    return howManyVisits;
}
```

```java
public void startBackgroundBatch() {
    TransactionManager tm = transactionalCache
      .getAdvancedCache().getTransactionManager();
    tm.begin();
    transactionalCache.put(KEY, 1000);
    System.out.println("HowManyVisits should now be 1000, " +
      "but we are holding the transaction");
    Thread.sleep(1000L);
    tm.rollback();
    System.out.println("The slow batch suffered a rollback");
}
```

现在，让我们创建一个执行这两种方法的测试，并检查 Infinispan 将如何运行:

```java
@Test
public void whenLockingAnEntry_thenItShouldBeInaccessible() throws InterruptedException {
    Runnable backGroundJob = () -> transactionalService.startBackgroundBatch();
    Thread backgroundThread = new Thread(backGroundJob);
    transactionalService.getQuickHowManyVisits();
    backgroundThread.start();
    Thread.sleep(100); //lets wait our thread warm up

    assertThat(timeThis(() -> transactionalService.getQuickHowManyVisits()))
      .isGreaterThan(500).isLessThan(1000);
}
```

执行它时，我们将再次在控制台中看到以下活动:

```java
Adding key 'key' to cache
Key 'key' was visited
Ill try to set HowManyVisits to 1
I was able to set HowManyVisits to 1 after waiting 0.001 seconds
HowManyVisits should now be 1000, but we are holding the transaction
Key 'key' was visited
Ill try to set HowManyVisits to 2
I was able to set HowManyVisits to 2 after waiting 0.902 seconds
The slow batch suffered a rollback
```

检查主线程上的时间，等待 slow 方法创建的事务结束。

## 5。结论

在本文中，我们已经了解了什么是 Infinispan，以及它作为应用程序中的缓存的主要特性和功能。

和往常一样，代码可以在 Github 上找到[。](https://web.archive.org/web/20220627082753/https://github.com/eugenp/tutorials/tree/master/libraries-data-2)