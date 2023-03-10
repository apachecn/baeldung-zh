# spring Cache——创建定制的密钥生成器

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-cache-custom-keygenerator>

## 1.概观

在这个快速教程中，我们将演示如何用 Spring Cache 创建一个自定义的键生成器。

关于上述模块的介绍，请参考本文中的[。](/web/20221126214040/https://www.baeldung.com/spring-cache-tutorial)

## 2.`KeyGenerator`

它负责为缓存中的每个数据项生成每个键，用于在检索时查找数据项。

这里的默认实现是`SimpleKeyGenerator – `，它使用提供的方法参数来生成一个密钥。这意味着，如果我们有两个方法使用相同的缓存名称和参数类型集，那么很有可能会导致冲突。

这也意味着缓存数据可以被另一种方法覆盖。

## 3.自定义`KeyGenerator`

一个`KeyGenerator`只需要实现一个方法:

```java
Object generate(Object object, Method method, Object... params)
```

如果没有正确实现或使用，可能会导致覆盖缓存数据。

让我们看一下实现:

```java
public class CustomKeyGenerator implements KeyGenerator {

    public Object generate(Object target, Method method, Object... params) {
        return target.getClass().getSimpleName() + "_"
          + method.getName() + "_"
          + StringUtils.arrayToDelimitedString(params, "_");
    }
}
```

之后，我们有两种可能的使用方法；第一个是在`ApplicationConfig`中声明一个 bean。

需要注意的是，该类必须从`CachingConfigurerSupport`扩展或者实现`CacheConfigurer`:

```java
@EnableCaching
@Configuration
public class ApplicationConfig extends CachingConfigurerSupport {

    @Bean
    public CacheManager cacheManager() {
        SimpleCacheManager cacheManager = new SimpleCacheManager();
        Cache booksCache = new ConcurrentMapCache("books");
        cacheManager.setCaches(Arrays.asList(booksCache));
        return cacheManager;
    }

    @Bean("customKeyGenerator")
    public KeyGenerator keyGenerator() {
        return new CustomKeyGenerator();
    }
}
```

第二种方法是将它用于特定的方法:

```java
@Component
public class BookService {

    @Cacheable(value = "books", keyGenerator = "customKeyGenerator")
    public List<Book> getBooks() {
        List<Book> books = new ArrayList<>();
        books.add(new Book("The Counterfeiters", "André Gide"));
        books.add(new Book("Peer Gynt and Hedda Gabler", "Henrik Ibsen"));
        return books;
    }
}
```

## 4.结论

在本文中，我们探索了一种实现自定义 Spring 缓存的`KeyGenerator`的方法。

与往常一样，GitHub 上的[提供了示例的完整源代码。](https://web.archive.org/web/20221126214040/https://github.com/eugenp/tutorials/tree/master/spring-caching)