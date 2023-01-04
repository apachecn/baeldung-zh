# 与 MongoDB 的春季会议

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-session-mongodb>

## 1。概述

在这个快速教程中，我们将探索如何使用由 [MongoDB](/web/20220627090416/https://www.baeldung.com/spring-data-mongodb-tutorial) 支持的 Spring 会话，无论有没有 Spring Boot。

春季会议也可以支持其他商店，如 [Redis](/web/20220627090416/https://www.baeldung.com/spring-session) 和 [JDBC](/web/20220627090416/https://www.baeldung.com/spring-session-jdbc) 。

## 2。Spring Boot 配置

首先，让我们看看 Spring Boot 所需的依赖项和配置。首先，让我们将最新版本的 [`spring-session-data-mongodb`](https://web.archive.org/web/20220627090416/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22org.springframework.session%22%20AND%20a%3A%22spring-session-data-mongodb%22) 和`[spring-boot-starter-data-mongodb](https://web.archive.org/web/20220627090416/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22org.springframework.boot%22%20AND%20a%3A%22spring-boot-starter-data-mongodb%22)`添加到我们的项目中:

```
<dependency>
    <groupId>org.springframework.session</groupId>
    <artifactId>spring-session-data-mongodb</artifactId>
    <version>2.2.6.RELEASE</version>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-mongodb</artifactId>
    <version>2.2.6.RELEASE</version>
</dependency>
```

之后，为了启用 Spring Boot 自动配置，我们需要在`application.properties`中添加 Spring Session 存储类型作为`mongodb`:

```
spring.session.store-type=mongodb
```

## 3。无 Spring Boot 的弹簧配置

现在，让我们看看在没有 Spring Boot 的情况下，在 MongoDB 中存储 Spring 会话所需的依赖项和配置。

类似于 Spring Boot 配置，我们将需要`spring-session-data-mongodb`依赖项。然而，这里我们将使用 [`spring-data-mongodb`](https://web.archive.org/web/20220627090416/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22org.springframework.data%22%20AND%20a%3A%22spring-data-mongodb%22) 依赖项来访问我们的 MongoDB 数据库:

```
<dependency>
    <groupId>org.springframework.session</groupId>
    <artifactId>spring-session-data-mongodb</artifactId>
    <version>2.2.6.RELEASE</version>
</dependency>
<dependency>
    <groupId>org.springframework.data</groupId>
    <artifactId>spring-data-mongodb</artifactId>
    <version>2.2.6.RELEASE</version>
</dependency>
```

最后，让我们看看如何配置应用程序:

```
@EnableMongoHttpSession
public class HttpSessionConfig {

    @Bean
    public JdkMongoSessionConverter jdkMongoSessionConverter() {
        return new JdkMongoSessionConverter(Duration.ofMinutes(30));
    }
}
```

@ `EnableMongoHttpSession` 注释支持在 MongoDB 中存储会话数据所需的配置。

另外，请注意，`JdkMongoSessionConverter`负责序列化和反序列化会话数据。

## 4。应用示例

让我们创建一个应用程序来测试配置。我们将使用 Spring Boot，因为它速度更快，需要的配置更少。

我们将从创建控制器来处理请求开始:

```
@RestController
public class SpringSessionMongoDBController {

    @GetMapping("/")
    public ResponseEntity<Integer> count(HttpSession session) {

        Integer counter = (Integer) session.getAttribute("count");

        if (counter == null) {
            counter = 1;
        } else {
            counter++;
        }

        session.setAttribute("count", counter);

        return ResponseEntity.ok(counter);
    }
}
```

正如我们在这个例子中看到的，我们在每次命中端点时递增`counter`，并将其值存储在名为`count`的会话属性中。

## 5。测试应用程序

让我们测试应用程序，看看我们是否真的能够在 MongoDB 中存储会话数据。

为此，我们将访问端点并检查我们将收到的 cookie。这将包含一个会话 id。

之后，我们将查询 MongoDB 集合，使用会话 id 获取会话数据:

```
@Test
public void 
  givenEndpointIsCalledTwiceAndResponseIsReturned_whenMongoDBIsQueriedForCount_thenCountMustBeSame() {

    HttpEntity<String> response = restTemplate
      .exchange("http://localhost:" + 8080, HttpMethod.GET, null, String.class);
    HttpHeaders headers = response.getHeaders();
    String set_cookie = headers.getFirst(HttpHeaders.SET_COOKIE);

    Assert.assertEquals(response.getBody(),
      repository.findById(getSessionId(set_cookie)).getAttribute("count").toString());
}

private String getSessionId(String cookie) {
    return new String(Base64.getDecoder().decode(cookie.split(";")[0].split("=")[1]));
}
```

## 6。它是如何工作的？

让我们来看看春季会议的幕后发生了什么。

`SessionRepositoryFilter`负责大部分工作:

*   将`HttpSession`转换成`MongoSession`
*   检查是否有一个 [`Cookie`](/web/20220627090416/https://www.baeldung.com/cookies-java) 存在，如果存在，从存储中加载会话数据
*   将更新的会话数据保存在存储中
*   检查会话的有效性

另外，`SessionRepositoryFilter` 创建一个名为`SESSION` 的 cookie，它是 HttpOnly 和安全的。这个 cookie 包含会话 id，它是一个 Base64 编码的值。

**为了定制 cookie 的名称或属性，我们必须创建一个类型为`DefaultCookieSerializer.`** 的 Spring bean

例如，这里我们禁用了 cookie 的`httponly`属性:

```
@Bean
public DefaultCookieSerializer customCookieSerializer(){
    DefaultCookieSerializer cookieSerializer = new DefaultCookieSerializer();

    cookieSerializer.setUseHttpOnlyCookie(false);

    return cookieSerializer;
}
```

## 7。存储在 MongoDB 中的会话详细信息

让我们在 MongoDB 控制台中使用以下命令来查询我们的会话集合:

```
db.sessions.findOne()
```

结果，我们将得到一个类似于以下内容的 [BSON](/web/20220627090416/https://www.baeldung.com/mongodb-bson) 文档:

```
{
    "_id" : "5d985be4-217c-472c-ae02-d6fca454662b",
    "created" : ISODate("2019-05-14T16:45:41.021Z"),
    "accessed" : ISODate("2019-05-14T17:18:59.118Z"),
    "interval" : "PT30M",
    "principal" : null,
    "expireAt" : ISODate("2019-05-14T17:48:59.118Z"),
    "attr" : BinData(0,"rO0ABXNyABFqYXZhLnV0aWwuSGFzaE1hcAUH2sHDFmDRAwACRgAKbG9hZEZhY3RvckkACXRocmVzaG9sZHhwP0AAAAAAAAx3CAAAABAAAAABdAAFY291bnRzcgARamF2YS5sYW5nLkludGVnZXIS4qCk94GHOAIAAUkABXZhbHVleHIAEGphdmEubGFuZy5OdW1iZXKGrJUdC5TgiwIAAHhwAAAAC3g=")
}
```

`_id` 是一个 [UUID](/web/20220627090416/https://www.baeldung.com/java-uuid) ，它将由`DefaultCookieSerializer`进行 Base64 编码，并设置为`SESSION` cookie 中的一个值。另外，请注意,`attr`属性包含我们的计数器的实际值。

## 8。结论

在本教程中，我们探索了由 MongoDB 支持的 Spring Session 一个在分布式系统中管理 HTTP 会话的强大工具。考虑到这个目的，它对于**解决跨应用程序的多个实例**复制会话的问题非常有用。

和往常一样，源代码可以在 GitHub 的[上找到。](https://web.archive.org/web/20220627090416/https://github.com/eugenp/tutorials/tree/master/spring-security-modules/spring-session)