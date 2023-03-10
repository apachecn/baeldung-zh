# 假装入门

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/intro-to-feign>

## 1。概述

在本教程中，我们将介绍由网飞开发的声明式 HTTP 客户端 [Feign](https://web.archive.org/web/20220627092521/https://github.com/OpenFeign/feign) 。

Feign 的目标是简化 HTTP API 客户端。简单地说，开发人员只需要声明和注释一个接口，而实际的实现是在运行时提供的。

## 2。示例

在本教程中，我们将使用一个示例[书店应用程序](https://web.archive.org/web/20220627092521/https://github.com/Baeldung/spring-hypermedia-api)，它公开了 REST API 端点。

我们可以轻松地克隆项目并在本地运行它:

```java
mvn install spring-boot:run
```

## 3。设置

首先，让我们添加所需的依赖项:

```java
<dependency>
    <groupId>io.github.openfeign</groupId>
    <artifactId>feign-okhttp</artifactId>
    <version>10.11</version>
</dependency>
<dependency>
    <groupId>io.github.openfeign</groupId>
    <artifactId>feign-gson</artifactId>
    <version>10.11</version>
</dependency>
<dependency>
    <groupId>io.github.openfeign</groupId>
    <artifactId>feign-slf4j</artifactId>
    <version>10.11</version>
</dependency>
```

除了 [feign-core](https://web.archive.org/web/20220627092521/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22io.github.openfeign%22%20AND%20a%3A%22feign-core%22) 依赖(这也是引入的)，我们将使用一些插件，特别是: [feign-okhttp](https://web.archive.org/web/20220627092521/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22io.github.openfeign%22%20AND%20a%3A%22feign-okhttp%22) 用于内部使用 Square 的 [OkHttp](https://web.archive.org/web/20220627092521/https://square.github.io/okhttp/) 客户端发出请求， [feign-gson](https://web.archive.org/web/20220627092521/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22io.github.openfeign%22%20AND%20a%3A%22feign-gson%22) 用于使用 Google 的 gson 作为 JSON 处理器， [feign-slf4j](https://web.archive.org/web/20220627092521/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22io.github.openfeign%22%20AND%20a%3A%22feign-slf4j%22) 用于使用`Simple Logging Facade`记录请求。

为了实际获得一些日志输出，我们需要在类路径上实现我们最喜欢的、SLF4J 支持的日志记录器。

在我们继续创建我们的客户端接口之前，首先我们将建立一个用于保存数据的`Book`模型:

```java
public class Book {
    private String isbn;
    private String author;
    private String title;
    private String synopsis;
    private String language;

    // standard constructor, getters and setters
}
```

**注意:**JSON 处理器至少需要一个“无参数构造函数”。

事实上，我们的 REST 提供者是一个[超媒体驱动的 API](/web/20220627092521/https://www.baeldung.com/spring-hateoas-tutorial) `,`，所以我们还需要一个简单的包装类:

```java
public class BookResource {
    private Book book;

    // standard constructor, getters and setters
}
```

**注意:**我们**’**将保持`BookResource`的简单，因为我们的样本假装客户端没有从超媒体特性中受益！

## 4。服务器端

为了理解如何定义一个虚拟客户端，我们将首先研究 REST 提供者支持的一些方法和响应。

让我们用一个简单的 curl shell 命令来列出所有的书。我们需要记住给所有的调用加上前缀`/api`，这是应用程序的 servlet 上下文:

```java
curl http://localhost:8081/api/books
```

因此，我们将获得一个完整的图书仓库，表示为 JSON:

```java
[
  {
    "book": {
      "isbn": "1447264533",
      "author": "Margaret Mitchell",
      "title": "Gone with the Wind",
      "synopsis": null,
      "language": null
    },
    "links": [
      {
        "rel": "self",
        "href": "http://localhost:8081/api/books/1447264533"
      }
    ]
  },

  ...

  {
    "book": {
      "isbn": "0451524934",
      "author": "George Orwell",
      "title": "1984",
      "synopsis": null,
      "language": null
    },
    "links": [
      {
        "rel": "self",
        "href": "http://localhost:8081/api/books/0451524934"
      }
    ]
  }
]
```

我们还可以通过将 ISBN 附加到 get 请求来查询单个的`Book` 资源:

```java
curl http://localhost:8081/api/books/1447264533
```

## 5。假装客户端

最后，让我们定义一下我们的虚拟客户。

我们将使用`@RequestLine`注释来指定 HTTP 动词和路径部分作为参数。将使用`@Param` 注释对参数进行建模:

```java
public interface BookClient {
    @RequestLine("GET /{isbn}")
    BookResource findByIsbn(@Param("isbn") String isbn);

    @RequestLine("GET")
    List<BookResource> findAll();

    @RequestLine("POST")
    @Headers("Content-Type: application/json")
    void create(Book book);
}
```

**注意:** Feign 客户端只能用于消费基于文本的 HTTP APIs，这意味着它们不能处理二进制数据，例如文件上传或下载。

**就这些！**现在我们将使用`Feign.builder()`来配置我们基于界面的客户端。实际的实现将在运行时提供:

```java
BookClient bookClient = Feign.builder()
  .client(new OkHttpClient())
  .encoder(new GsonEncoder())
  .decoder(new GsonDecoder())
  .logger(new Slf4jLogger(BookClient.class))
  .logLevel(Logger.Level.FULL)
  .target(BookClient.class, "http://localhost:8081/api/books");
```

Feign 支持各种插件，比如 JSON/XML 编码器和解码器，或者用于发出请求的底层 HTTP 客户端。

## 6。单元测试

让我们创建三个测试用例来测试我们的客户端。注意，我们对`org.hamcrest.CoreMatchers.*`和`org.junit.Assert.*`使用静态导入:

```java
@Test
public void givenBookClient_shouldRunSuccessfully() throws Exception {
   List<Book> books = bookClient.findAll().stream()
     .map(BookResource::getBook)
     .collect(Collectors.toList());

   assertTrue(books.size() > 2);
}

@Test
public void givenBookClient_shouldFindOneBook() throws Exception {
    Book book = bookClient.findByIsbn("0151072558").getBook();
    assertThat(book.getAuthor(), containsString("Orwell"));
}

@Test
public void givenBookClient_shouldPostBook() throws Exception {
    String isbn = UUID.randomUUID().toString();
    Book book = new Book(isbn, "Me", "It's me!", null, null);
    bookClient.create(book);
    book = bookClient.findByIsbn(isbn).getBook();

    assertThat(book.getAuthor(), is("Me"));
} 
```

## 7。延伸阅读

如果我们在服务不可用的情况下需要某种回退，我们可以将 [HystrixFeign](https://web.archive.org/web/20220627092521/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22io.github.openfeign%22%20AND%20a%3A%22feign-hystrix%22) 添加到类路径中，并用`HystrixFeign.builder()`构建我们的客户端。

查看[这个专门的教程系列](/web/20220627092521/https://www.baeldung.com/introduction-to-hystrix)以了解更多关于 Hystrix 的信息。

此外，如果我们想将 Spring Cloud 网飞 Hystrix 与 Feign 集成，这里有一篇关于[的专门文章](/web/20220627092521/https://www.baeldung.com/spring-cloud-netflix-hystrix)。

此外，还可以为我们的客户端添加客户端负载平衡和/或服务发现。

我们可以通过将 [Ribbon](https://web.archive.org/web/20220627092521/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22io.github.openfeign%22%20AND%20a%3A%22feign-ribbon%22) 添加到我们的类路径中来实现这一点，并像这样使用构建器:

```java
BookClient bookClient = Feign.builder()
  .client(RibbonClient.create())
  .target(BookClient.class, "http://localhost:8081/api/books");
```

对于服务发现，我们必须通过启用 Spring Cloud 网飞尤里卡来构建我们的服务。然后网飞佯称与春云融为一体。因此，我们可以免费获得带状负载平衡。关于这方面的更多信息可以在这里找到[。](/web/20220627092521/https://www.baeldung.com/spring-cloud-netflix-eureka)

## 8。结论

在本文中，我们解释了如何使用 Feign 构建一个声明性的 HTTP 客户端来使用基于文本的 API。

像往常一样，本教程中显示的所有代码示例都可以在 GitHub 上获得。