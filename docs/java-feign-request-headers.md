# 使用 Feign 设置请求头

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-feign-request-headers>

## 1.概观

当使用 [Feign](/web/20221128035320/https://www.baeldung.com/intro-to-feign) 时，有时我们需要在 HTTP 调用中设置请求头。Feign 允许我们简单地用声明性语法构建 HTTP 客户端。

在这个简短的教程中，我们将看到如何使用注释来配置请求头。我们还将看到如何通过使用拦截器来包含公共请求头。

## 2.例子

在整个教程中，我们将使用一个例子 [书店应用](https://web.archive.org/web/20221128035320/https://github.com/Baeldung/spring-hypermedia-api) 来公开 REST API 端点。

我们可以轻松地克隆项目并在本地运行它:

```java
$ mvn install spring-boot:run
```

让我们深入了解客户端实现。

## 3.使用`Header`注释

让我们考虑一个场景，其中特定的 API 调用应该总是包含一个静态头。在这种情况下，我们可以将请求头配置为客户机的一部分。一个典型的例子是包含一个`Content-Type`头。

使用`@Header` 注释，我们可以很容易地配置一个静态请求头。我们可以静态或动态地定义这个头值。

### 3.1.设置静态头值

让我们在`BookClient`中配置两个静态头，即`Accept-Language`和`Content-Type,`:

```java
@Headers("Accept-Language: en-US")
public interface BookClient {

    @RequestLine("GET /{isbn}")
    BookResource findByIsbn(@Param("isbn") String isbn);

    @RequestLine("POST")
    @Headers("Content-Type: application/json")
    void create(Book book);
}
```

在上面的代码中，头文件 `Accept-Language` 被应用到 `BookClient` 中，所以包含在所有的 API 中。但是， `create` 方法有一个额外的 `Content-Type` 头。

接下来，让我们看看如何使用 Feign 的`Builder`方法创建`BookClient`，并传递 [`HEADERS`](/web/20221128035320/https://www.baeldung.com/java-feign-logging) 日志级别:

```java
Feign.builder()
  .encoder(new GsonEncoder())
  .decoder(new GsonDecoder())
  .logger(new Slf4jLogger(type))
  .logLevel(Logger.Level.HEADERS)
  .target(BookClient.class, "http://localhost:8081/api/books"); 
```

现在，让我们测试一下`create`方法:

```java
String isbn = UUID.randomUUID().toString();
Book book = new Book(isbn, "Me", "It's me!", null, null);

bookClient.create(book);

book = bookClient.findByIsbn(isbn).getBook();
```

然后，让我们验证输出记录器中的标题:

```java
18:01:15.039 [main] DEBUG c.b.f.c.h.staticheader.BookClient - [BookClient#create] Accept-Language: en-US
18:01:15.039 [main] DEBUG c.b.f.c.h.staticheader.BookClient - [BookClient#create] Content-Type: application/json
18:01:15.096 [main] DEBUG c.b.f.c.h.staticheader.BookClient - [BookClient#findByIsbn] Accept-Language: en-US
```

我们应该注意，如果客户端接口和 API 方法中的**头名称相同，那么它们不会互相覆盖。相反，请求将包括所有这样的值。**

### 3.2.设置动态标题值

使用`@Header`注释，我们还可以设置一个动态头值。为此，我们需要将值表示为占位符。

让我们用占位符`requester` :将`x-requester-id`头包含到`BookClient`中

```java
@Headers("x-requester-id: {requester}")
public interface BookClient {

    @RequestLine("GET /{isbn}")
    BookResource findByIsbn(@Param("requester") String requester, @Param("isbn") String isbn);
}
```

这里我们将 `x-requester-id` 作为一个变量传递给每个方法。我们使用了 **`@Param `注释来匹配变量**的名称。它在运行时被扩展，以满足 `@Headers ` 注释指定的头。

现在，让我们用`x-requester-id` 头调用`BookClient` API:

```java
String requester = "test";
book = bookClient.findByIsbn(requester, isbn).getBook();
```

然后，让我们验证输出记录器中的请求头:

```java
18:04:27.515 [main] DEBUG c.b.f.c.h.s.parameterized.BookClient - [BookClient#findByIsbn] x-requester-id: test
```

## 4.使用`HeaderMaps`注释

让我们想象一个场景，标题键和值都是动态的。在这种情况下，可能的键的范围事先是未知的。此外，同一客户端上不同方法调用的头可能会有所不同。一个典型的例子是设置某些元数据头。

使用标注有 `@HeaderMap` 的`Map`参数设置动态头:

```java
@RequestLine("POST")
void create(@HeaderMap Map<String, Object> headers, Book book);
```

现在，让我们用头文件测试一下`create`方法:

```java
Map<String,Object> headerMap = new HashMap<>();

headerMap.put("metadata-key1", "metadata-value1");
headerMap.put("metadata-key2", "metadata-value2");

bookClient.create(headerMap, book);
```

然后，让我们验证输出记录器中的标题:

```java
18:05:03.202 [main] DEBUG c.b.f.c.h.dynamicheader.BookClient - [BookClient#create] metadata-key1: metadata-value1
18:05:03.202 [main] DEBUG c.b.f.c.h.dynamicheader.BookClient - [BookClient#create] metadata-key2: metadata-value2
```

## 5.请求拦截器

拦截器可以为每个请求或响应执行各种隐式任务，比如日志记录或身份验证。

Feign 提供了一个`RequestInterceptor`接口。这样，我们可以添加请求头。

当知道每个调用中都应该包含头部时，添加请求拦截器是有意义的。这种模式消除了调用代码实现非功能性需求(如身份验证或跟踪)的依赖性。

让我们通过实现一个用于生成授权令牌的`AuthorisationService` 来尝试一下:

```java
public class ApiAuthorisationService implements AuthorisationService {

    @Override
    public String getAuthToken() {
        return "Bearer " + UUID.randomUUID();
    }
}
```

现在，让我们实现我们的定制请求拦截器:

```java
public class AuthRequestInterceptor implements RequestInterceptor {

    private AuthorisationService authTokenService;

    public AuthRequestInterceptor(AuthorisationService authTokenService) {
        this.authTokenService = authTokenService;
    }

    @Override
    public void apply(RequestTemplate template) {
        template.header("Authorisation", authTokenService.getAuthToken());
    }
} 
```

我们应该注意到，**请求拦截器可以读取、删除或改变**请求模板的任何部分。

现在，让我们使用`builder`方法`:`将`AuthInterceptor` 添加到 `BookClient`

```java
Feign.builder()
  .requestInterceptor(new AuthInterceptor(new ApiAuthorisationService()))
  .encoder(new GsonEncoder())
  .decoder(new GsonDecoder())
  .logger(new Slf4jLogger(type))
  .logLevel(Logger.Level.HEADERS)
  .target(BookClient.class, "http://localhost:8081/api/books"); 
```

然后，让我们测试带有`Authorisation`头的`BookClient` API:

```java
bookClient.findByIsbn("0151072558").getBook();
```

现在，让我们验证输出记录器中的标题:

```java
18:06:06.135 [main] DEBUG c.b.f.c.h.staticheader.BookClient - [BookClient#findByIsbn] Authorisation: Bearer 629e0af7-513d-4385-a5ef-cb9b341cedb5
```

**多个请求拦截器也可以应用于**Feign 客户端。尽管没有保证它们的应用顺序。

## 6.结论

在本文中，我们讨论了 Feign client 如何支持设置请求头。我们使用`@Headers`、`@HeaderMaps`注释和请求拦截器实现了这个功能。

和往常一样，本教程中展示的所有代码示例都可以从 GitHub 上的[处获得。](https://web.archive.org/web/20221128035320/https://github.com/eugenp/tutorials/tree/master/feign)