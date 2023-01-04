# Spring Boot Ehcache 示例

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-boot-ehcache>

## 1.概观

让我们看一个使用 Ehcache 和 Spring Boot 的例子。我们将使用 Ehcache 版本 3，因为它提供了一个 [JSR-107](https://web.archive.org/web/20220712034740/https://www.jcp.org/en/jsr/detail?id=107) 缓存管理器的实现。

这个例子是一个简单的 REST 服务，它产生一个数的平方。

## 2.属国

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
    <version>2.6.1</version>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-cache</artifactId>
    <version>2.6.1</version></dependency>
<dependency>
    <groupId>javax.cache</groupId>
    <artifactId>cache-api</artifactId>
    <version>1.1.1</version>
</dependency>
<dependency>
    <groupId>org.ehcache</groupId>
    <artifactId>ehcache</artifactId>
    <version>3.8.1</version>
</dependency> 
```

*   [T2`spring-boot-starter-web`](https://web.archive.org/web/20220712034740/https://search.maven.org/classic/#search|ga|1|g%3A"org.springframework.boot"%20AND%20a%3A"spring-boot-starter-web")
*   `[spring-boot-starter-cache](https://web.archive.org/web/20220712034740/https://search.maven.org/classic/#search|ga|1|g%3A"org.springframework.boot"%20AND%20a%3A"spring-boot-starter-cache")`
*   [T2`javax.cache:cache-api`](https://web.archive.org/web/20220712034740/https://search.maven.org/classic/#search|ga|1|g%3A"javax.cache"%20AND%20a%3A"cache-api)
*   [T2`org.ehcache:ehcache`](https://web.archive.org/web/20220712034740/https://search.maven.org/classic/#search|ga|1|g%3A"org.ehcache"%20AND%20a%3A"ehcache")

## 3.例子

让我们创建一个简单的 REST 控制器，它调用一个服务来计算一个数字的平方，并将结果作为 JSON 字符串返回:

```
@RestController
@RequestMapping("/number", MediaType.APPLICATION_JSON_UTF8_VALUE)
public class NumberController {

    // ...

    @Autowired
    private NumberService numberService;

    @GetMapping(path = "/square/{number}")
    public String getSquare(@PathVariable Long number) {
        log.info("call numberService to square {}", number);
        return String.format("{\"square\": %s}", numberService.square(number));
    }
}
```

现在让我们创建服务。

我们用`@Cacheable` 注释该方法，这样 Spring 将处理缓存。**作为这个注释的结果，Spring 将创建一个`NumberService`的代理来拦截对`square`方法的调用并调用 Ehcache。**

我们需要提供要使用的缓存的名称和可选的密钥。我们还可以添加一个条件来限制缓存的内容:

```
@Service
public class NumberService {

    // ...
    @Cacheable(
      value = "squareCache", 
      key = "#number", 
      condition = "#number>10")
    public BigDecimal square(Long number) {
        BigDecimal square = BigDecimal.valueOf(number)
          .multiply(BigDecimal.valueOf(number));
        log.info("square of {} is {}", number, square);
        return square;
    }
}
```

最后，让我们创建主要的 Spring Boot 应用程序:

```
@SpringBootApplication
public class Application {

    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```

## 4.缓存配置

我们需要向 Spring bean 添加 Spring 的`@EnableCaching`注释，以便启用 Spring 的注释驱动的缓存管理。

让我们创建一个`CacheConfig`类:

```
@Configuration
@EnableCaching
public class CacheConfig {
}
```

**Spring 的自动配置找到 Ehcache 的 JSR-107 的实现。但是，默认情况下不会创建缓存。**

因为 Spring 和 Ehcache 都不会寻找默认的`ehcache.xml`文件。我们添加以下属性来告诉 Spring 在哪里可以找到它:

```
spring.cache.jcache.config=classpath:ehcache.xml 
```

让我们用名为`squareCache` `:`的缓存创建一个`ehcache.xml`文件

```
<config xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"

    xmlns:jsr107="http://www.ehcache.org/v3/jsr107"
    xsi:schemaLocation="
            http://www.ehcache.org/v3 http://www.ehcache.org/schema/ehcache-core-3.0.xsd
            http://www.ehcache.org/v3/jsr107 http://www.ehcache.org/schema/ehcache-107-ext-3.0.xsd">

    <cache alias="squareCache">
        <key-type>java.lang.Long</key-type>
        <value-type>java.math.BigDecimal</value-type>
        <expiry>
            <ttl unit="seconds">30</ttl>
        </expiry>

        <listeners>
            <listener>
                <class>com.baeldung.cachetest.config.CacheEventLogger</class>
                <event-firing-mode>ASYNCHRONOUS</event-firing-mode>
                <event-ordering-mode>UNORDERED</event-ordering-mode>
                <events-to-fire-on>CREATED</events-to-fire-on>
                <events-to-fire-on>EXPIRED</events-to-fire-on>
            </listener>
        </listeners>

        <resources>
            <heap unit="entries">2</heap>
            <offheap unit="MB">10</offheap>
        </resources>
    </cache>

</config>
```

我们还要添加缓存事件监听器，它记录`CREATED`和`EXPIRED`缓存事件:

```
public class CacheEventLogger 
  implements CacheEventListener<Object, Object> {

    // ...

    @Override
    public void onEvent(
      CacheEvent<? extends Object, ? extends Object> cacheEvent) {
        log.info(/* message */,
          cacheEvent.getKey(), cacheEvent.getOldValue(), cacheEvent.getNewValue());
    }
}
```

## 5.在活动

我们可以用 Maven 通过运行`mvn spring-boot:run`来启动这个 app。

然后打开浏览器，在端口 8080 上访问 REST 服务。

如果我们转到`[http://localhost:8080/number/square/12](https://web.archive.org/web/20220712034740/http://localhost:8080/number/square/12), `，那么我们将返回`{“square”:144}`，在日志中我们将看到:

```
INFO [nio-8080-exec-1] c.b.cachetest.rest.NumberController : call numberService to square 12
INFO [nio-8080-exec-1] c.b.cachetest.service.NumberService : square of 12 is 144
INFO [e [_default_]-0] c.b.cachetest.config.CacheEventLogger : Cache event CREATED for item with key 12\. Old value = null, New value = 144
```

我们可以看到来自`NumberService`的`square`方法的日志消息，以及来自 EventLogger 的`CREATED`事件。**如果我们刷新浏览器，我们只会看到以下内容被添加到日志中:**

```
INFO [nio-8080-exec-2] c.b.cachetest.rest.NumberController : call numberService to square 12
```

没有调用`NumberService`的`square`方法中的日志消息。这表明缓存的值正在被使用。

**如果我们等待 30 秒，等待缓存项过期并刷新浏览器，我们将看到一个`EXPIRED`事件、**和添加回缓存的值:

```
INFO [nio-8080-exec-1] (...) NumberController : call numberService to square 12
INFO [e [_default_]-1] (...) CacheEventLogger : Cache event EXPIRED for item with key 12\. Old value = 144,New value = null
INFO [nio-8080-exec-1] (... )NumberService : square of 12 is 144
INFO [e [_default_]-1] (...) CacheEventLogger : Cache event CREATED for item with key 12\. Old value = null, New value = 144
```

如果我们在浏览器中输入 [`http://localhost:8080/number/square/3`](https://web.archive.org/web/20220712034740/http://localhost:8080/number/square/3) ，我们得到的正确答案是 9，但是这个值没有被缓存。

这是因为我们在`@Cacheable`注释中使用了只缓存大于 10 的数值的条件。

## 6.结论

在这个快速教程中，我们展示了如何用 Spring Boot 设置 Ehcache。

和往常一样，代码[可以在 GitHub](https://web.archive.org/web/20220712034740/https://github.com/eugenp/tutorials/tree/master/spring-caching) 上找到。