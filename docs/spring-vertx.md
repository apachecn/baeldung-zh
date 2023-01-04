# Vert.x 弹簧集成

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-vertx>

## 1。概述

在这篇简短的文章中，我们将讨论 Spring 与 Vert-x 的集成，并利用两者的优势:强大且众所周知的 Spring 特性，以及 Vert.x 的反应式单事件循环。

要了解更多关于 Vert.x 的信息，请参考我们的介绍文章[这里](/web/20220627075800/https://www.baeldung.com/vertx)。

## 2。设置

首先，让我们把依赖项放在适当的位置:

```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
    <exclusions>
        <exclusion>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-tomcat</artifactId>
        </exclusion>
    </exclusions>
</dependency>
<dependency>
    <groupId>io.vertx</groupId>
    <artifactId>vertx-web</artifactId>
    <version>3.4.1</version>
</dependency>
```

请注意，我们已经从`spring-boot-starter-web s`中排除了嵌入式 Tomcat 依赖，因为我们将使用 verticles 部署我们的服务。

你可以在这里找到最新的依赖关系。

## 3。Spring Vert.x 应用程序

现在，我们将构建一个部署了两个垂直领域的示例应用程序。

第一个 Verticle 将请求路由到处理程序，处理程序将它们作为消息发送到给定的地址。另一个垂直监听给定的地址。

让我们看看这些是如何运作的。

### 3.1。垂直发送方

**`ServerVerticle`接受 HTTP 请求并将它们作为消息发送到指定的地址。**让我们创建一个扩展了`AbstractVerticle,` 的`ServerVerticle` 类，并覆盖`start()` 方法来创建我们的 HTTP 服务器:

```java
@Override
public void start() throws Exception {
    super.start();

    Router router = Router.router(vertx);
    router.get("/api/baeldung/articles")
      .handler(this::getAllArticlesHandler);

    vertx.createHttpServer()
      .requestHandler(router::accept)
      .listen(config().getInteger("http.port", 8080));
}
```

在服务器请求处理程序中，我们传递了一个`router` 对象，它将任何传入的请求重定向到`getAllArticlesHandler`处理程序:

```java
private void getAllArticlesHandler(RoutingContext routingContext) {
    vertx.eventBus().<String>send(ArticleRecipientVerticle.GET_ALL_ARTICLES, "", 
      result -> {
        if (result.succeeded()) {
            routingContext.response()
              .putHeader("content-type", "application/json")
              .setStatusCode(200)
              .end(result.result()
              .body());
        } else {
            routingContext.response()
              .setStatusCode(500)
              .end();
        }
      });
}
```

在 handler 方法中，我们向 Vert.x 事件总线传递一个事件，事件 id 为`GET_ALL_ARTICLES.` ，然后我们相应地处理成功和错误场景的回调。

来自事件总线的消息将由下一节讨论的`ArticleRecipientVerticle` `,` 使用。

### 3.2。垂直收件人

**`ArticleRecipientVerticle`监听传入的消息并注入一个 Spring bean** 。它充当 Spring 和 Vert.x 的集合点。

我们将把 Spring service bean 注入到一个 Verticle 中，并调用各自的方法:

```java
@Override
public void start() throws Exception {
    super.start();
    vertx.eventBus().<String>consumer(GET_ALL_ARTICLES)
      .handler(getAllArticleService(articleService));
} 
```

这里，`articleService` 是注入的春豆:

```java
@Autowired
private ArticleService articleService; 
```

这个 Verticle 将继续监听地址`GET_ALL_ARTICLES.` 上的事件总线，一旦它接收到消息，它就将它委托给`getAllArticleService` 处理程序方法:

```java
private Handler<Message<String>> getAllArticleService(ArticleService service) {
    return msg -> vertx.<String> executeBlocking(future -> {
        try {
            future.complete(
            mapper.writeValueAsString(service.getAllArticle()));
        } catch (JsonProcessingException e) {
            future.fail(e);
        }
    }, result -> {
        if (result.succeeded()) {
            msg.reply(result.result());
        } else {
            msg.reply(result.cause().toString());
        }
    });
}
```

这将执行所需的服务操作，并以状态回复消息。正如我们在前面章节中看到的，消息回复在`ServerVerticle`和回调`result`处被引用。

## 4。服务等级

服务类是一个简单的实现，提供了与存储库层交互的方法:

```java
@Service
public class ArticleService {

    @Autowired
    private ArticleRepository articleRepository;

    public List<Article> getAllArticle() {
        return articleRepository.findAll();
    }
}
```

`ArticleRepository` 扩展了`org.springframework.data.repository.CrudRepository` 并提供基本的 CRUD 功能。

## 5。部署垂直设备

我们将部署该应用程序，就像我们部署常规 Spring Boot 应用程序一样。我们必须创造一个垂直。x 实例，并在 Spring 上下文初始化完成后在其中部署 verticles:

```java
public class VertxSpringApplication {

    @Autowired
    private ServerVerticle serverVerticle;

    @Autowired
    private ArticleRecipientVerticle articleRecipientVerticle;

    public static void main(String[] args) {
        SpringApplication.run(VertxSpringApplication.class, args);
    }

    @PostConstruct
    public void deployVerticle() {
        Vertx vertx = Vertx.vertx();
        vertx.deployVerticle(serverVerticle);
        vertx.deployVerticle(articleRecipientVerticle);
    }
}
```

注意，我们将垂直实例注入到 Spring 应用程序类中。因此，我们必须注释垂直类，

因此，我们必须用`@Component.`来注释垂直类`ServerVerticle` 和`ArticleRecipientVerticle`

让我们测试应用程序:

```java
@Test
public void givenUrl_whenReceivedArticles_thenSuccess() {
    ResponseEntity<String> responseEntity = restTemplate
      .getForEntity("http://localhost:8080/api/baeldung/articles", String.class);

    assertEquals(200, responseEntity.getStatusCodeValue());
}
```

## 6。结论

在本文中，我们学习了如何使用 Spring 和 Vert.x 构建 RESTful WebService。

像往常一样，这个例子可以在 GitHub 上找到。