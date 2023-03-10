# 与 Spring Cloud 网飞和 Feign 的集成测试

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-cloud-feign-integration-tests>

## 1.概观

在本文中，我们将探索一个虚拟客户端的**集成测试。**

我们将创建一个基本的 [Open Feign 客户端](/web/20221126220215/https://www.baeldung.com/spring-cloud-openfeign)，为此**我们将在 [WireMock](/web/20221126220215/https://www.baeldung.com/introduction-to-wiremock) 的帮助下编写一个简单的集成测试**。

之后，**我们将为我们的客户端**添加一个 [功能区](/web/20221126220215/https://www.baeldung.com/spring-cloud-rest-client-with-netflix-ribbon) **配置，并为其构建一个集成测试。最后，**我们将配置一个** [Eureka](/web/20221126220215/https://www.baeldung.com/spring-cloud-netflix-eureka) **测试容器，并测试这个设置**以确保我们的整个配置按预期工作。**

## 2.假装的客户

要设置我们的 Feign 客户端，我们应该首先添加[Spring Cloud open Feign](https://web.archive.org/web/20221126220215/https://search.maven.org/artifact/org.springframework.cloud/spring-cloud-starter-openfeign)Maven 依赖项:

```java
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-openfeign</artifactId>
</dependency> 
```

之后，让我们为我们的模型创建一个`Book`类:

```java
public class Book {
    private String title;
    private String author;
}
```

最后，让我们创建我们的虚拟客户端接口:

```java
@FeignClient(value="simple-books-client", url="${book.service.url}")
public interface BooksClient {

    @RequestMapping("/books")
    List<Book> getBooks();

}
```

现在，**我们有了一个从 REST 服务中检索`Books`列表的虚拟客户端。**现在，让我们继续前进，编写一些集成测试。

## 3.WireMock

### 3.1.设置 WireMock 服务器

如果我们想要测试我们的`BooksClient,`，我们需要一个提供`/books`端点的模拟服务。**我们的客户会打电话反对这项模拟服务。**为此，我们将使用 WireMock。

因此，让我们添加 [WireMock](https://web.archive.org/web/20221126220215/https://search.maven.org/artifact/com.github.tomakehurst/wiremock) Maven 依赖关系:

```java
<dependency>
    <groupId>com.github.tomakehurst</groupId>
    <artifactId>wiremock</artifactId>
    <scope>test</scope>
</dependency> 
```

并配置模拟服务器:

```java
@TestConfiguration
public class WireMockConfig {

    @Autowired
    private WireMockServer wireMockServer;

    @Bean(initMethod = "start", destroyMethod = "stop")
    public WireMockServer mockBooksService() {
        return new WireMockServer(9561);
    }

}
```

我们现在有一个正在运行的模拟服务器，它在端口 9651 上接受连接。

### 3.2.设置模拟

让我们将属性`book.service.url`添加到指向`WireMockServer`端口的`application-test.yml` 中:

```java
book:
  service:
    url: http://localhost:9561
```

让我们也为`/books`端点准备一个模拟响应`get-books-response.json`:

```java
[
  {
    "title": "Dune",
    "author": "Frank Herbert"
  },
  {
    "title": "Foundation",
    "author": "Isaac Asimov"
  }
]
```

现在让我们为`/books`端点上的`GET`请求配置模拟响应:

```java
public class BookMocks {

    public static void setupMockBooksResponse(WireMockServer mockService) throws IOException {
        mockService.stubFor(WireMock.get(WireMock.urlEqualTo("/books"))
          .willReturn(WireMock.aResponse()
            .withStatus(HttpStatus.OK.value())
            .withHeader("Content-Type", MediaType.APPLICATION_JSON_VALUE)
            .withBody(
              copyToString(
                BookMocks.class.getClassLoader().getResourceAsStream("payload/get-books-response.json"),
                defaultCharset()))));
    }

}
```

至此，所有需要的配置都已就绪。让我们继续编写我们的第一个测试。

## 4.我们的第一次集成测试

让我们创建一个集成测试`BooksClientIntegrationTest`:

```java
@SpringBootTest
@ActiveProfiles("test")
@EnableConfigurationProperties
@ExtendWith(SpringExtension.class)
@ContextConfiguration(classes = { WireMockConfig.class })
class BooksClientIntegrationTest {

    @Autowired
    private WireMockServer mockBooksService;

    @Autowired
    private BooksClient booksClient;

    @BeforeEach
    void setUp() throws IOException {
        BookMocks.setupMockBooksResponse(mockBooksService);
    }

    // ...
}
```

此时，我们有一个配置了一个`WireMockServer`的`SpringBootTest`，当`/books`端点被`BooksClient`调用时，它准备返回一个预定义的`Books`列表。

最后，让我们添加我们的测试方法:

```java
@Test
public void whenGetBooks_thenBooksShouldBeReturned() {
    assertFalse(booksClient.getBooks().isEmpty());
}

@Test
public void whenGetBooks_thenTheCorrectBooksShouldBeReturned() {
    assertTrue(booksClient.getBooks()
      .containsAll(asList(
        new Book("Dune", "Frank Herbert"),
        new Book("Foundation", "Isaac Asimov"))));
}
```

## 5.与功能区集成

现在**让我们通过添加 Ribbon 提供的负载平衡功能**来改进我们的客户端。

在客户端界面中，我们需要做的就是删除硬编码的服务 URL，代之以通过服务名`book-service`引用服务:

```java
@FeignClient("books-service")
public interface BooksClient {
...
```

接下来，添加[网飞丝带](https://web.archive.org/web/20221126220215/https://search.maven.org/artifact/org.springframework.cloud/spring-cloud-starter-netflix-ribbon) Maven 依赖:

```java
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-ribbon</artifactId>
</dependency>
```

最后，在`application-test.yml`文件中，我们现在应该删除`book.service.url`并定义功能区`listOfServers`:

```java
books-service:
  ribbon:
    listOfServers: http://localhost:9561
```

现在让我们再次运行`BooksClientIntegrationTest`。它应该会通过，确认新的设置按预期工作。

### 5.1.动态端口配置

如果我们不想硬编码服务器的端口，我们可以配置 WireMock 在启动时使用一个动态端口。

为此，让我们创建另一个测试配置，`RibbonTestConfig:`

```java
@TestConfiguration
@ActiveProfiles("ribbon-test")
public class RibbonTestConfig {

    @Autowired
    private WireMockServer mockBooksService;

    @Autowired
    private WireMockServer secondMockBooksService;

    @Bean(initMethod = "start", destroyMethod = "stop")
    public WireMockServer mockBooksService() {
        return new WireMockServer(options().dynamicPort());
    }

    @Bean(name="secondMockBooksService", initMethod = "start", destroyMethod = "stop")
    public WireMockServer secondBooksMockService() {
        return new WireMockServer(options().dynamicPort());
    }

    @Bean
    public ServerList ribbonServerList() {
        return new StaticServerList<>(
          new Server("localhost", mockBooksService.port()),
          new Server("localhost", secondMockBooksService.port()));
    }

}
```

这个配置设置了两个 WireMock 服务器，每个运行在运行时动态分配的不同端口上。此外，它还使用两个模拟服务器配置 Ribbon 服务器列表。

### 5.2.负载平衡测试

现在我们已经配置了我们的带状负载平衡器，**让我们确保我们的`BooksClient`在两个模拟服务器:**之间正确地交替

```java
@SpringBootTest
@ActiveProfiles("ribbon-test")
@EnableConfigurationProperties
@ExtendWith(SpringExtension.class)
@ContextConfiguration(classes = { RibbonTestConfig.class })
class LoadBalancerBooksClientIntegrationTest {

    @Autowired
    private WireMockServer mockBooksService;

    @Autowired
    private WireMockServer secondMockBooksService;

    @Autowired
    private BooksClient booksClient;

    @BeforeEach
    void setUp() throws IOException {
        setupMockBooksResponse(mockBooksService);
        setupMockBooksResponse(secondMockBooksService);
    }

    @Test
    void whenGetBooks_thenRequestsAreLoadBalanced() {
        for (int k = 0; k < 10; k++) {
            booksClient.getBooks();
        }

        mockBooksService.verify(
          moreThan(0), getRequestedFor(WireMock.urlEqualTo("/books")));
        secondMockBooksService.verify(
          moreThan(0), getRequestedFor(WireMock.urlEqualTo("/books")));
    }

    @Test
    public void whenGetBooks_thenTheCorrectBooksShouldBeReturned() {
        assertTrue(booksClient.getBooks()
          .containsAll(asList(
            new Book("Dune", "Frank Herbert"),
            new Book("Foundation", "Isaac Asimov"))));
    }
}
```

## 6.尤里卡整合

到目前为止，我们已经看到了如何测试使用 Ribbon 进行负载平衡的客户机。但是，如果我们的设置使用像 Eureka 这样的服务发现系统会怎么样呢？我们应该编写一个集成测试，确保我们的`BooksClient`在这样的环境中也能像预期的一样工作。

为此，**我们将运行一个 Eureka 服务器作为测试容器**。然后我们启动并用我们的 Eureka 容器注册一个模拟`book-service`。最后，安装完成后，我们可以对其进行测试。

在继续之前，让我们添加一下[测试容器](https://web.archive.org/web/20221126220215/https://search.maven.org/artifact/org.testcontainers/testcontainers)和[网飞尤里卡客户端](https://web.archive.org/web/20221126220215/https://search.maven.org/artifact/org.springframework.cloud/spring-cloud-starter-netflix-eureka-client) Maven 依赖项:

```java
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>
<dependency>
    <groupId>org.testcontainers</groupId>
    <artifactId>testcontainers</artifactId>
    <scope>test</scope>
</dependency>
```

### 6.1.测试容器设置

让我们创建一个 TestContainer 配置来启动我们的 Eureka 服务器:

```java
public class EurekaContainerConfig {

    public static class Initializer implements ApplicationContextInitializer {

        public static GenericContainer eurekaServer = 
          new GenericContainer("springcloud/eureka").withExposedPorts(8761);

        @Override
        public void initialize(@NotNull ConfigurableApplicationContext configurableApplicationContext) {

            Startables.deepStart(Stream.of(eurekaServer)).join();

            TestPropertyValues
              .of("eureka.client.serviceUrl.defaultZone=http://localhost:" 
                + eurekaServer.getFirstMappedPort().toString() 
                + "/eureka")
              .applyTo(configurableApplicationContext);
        }
    }
}
```

正如我们所看到的，上面的初始化器启动了容器。然后它暴露端口 8761，Eureka 服务器正在监听该端口。

最后，在 Eureka 服务启动后，我们需要更新`eureka.client.serviceUrl.defaultZone` 属性。这定义了用于服务发现的 Eureka 服务器的地址。

### 6.2.注册模拟服务器

既然我们的 Eureka 服务器已经启动并运行，我们需要注册一个模拟的`books-service`。我们通过简单地创建一个 RestController 来实现这一点:

```java
@Configuration
@RestController
@ActiveProfiles("eureka-test")
public class MockBookServiceConfig {

    @RequestMapping("/books")
    public List getBooks() {
        return Collections.singletonList(new Book("Hitchhiker's Guide to the Galaxy", "Douglas Adams"));
    }
}
```

为了注册这个控制器，我们现在要做的就是确保我们的`application-eureka-test.yml`中的`spring.application.name`属性与`BooksClient`接口中使用的服务名`books-service,` 相同。

注意:既然`netflix-eureka-client`库在我们的依赖列表中，默认情况下，Eureka 将用于服务发现。因此，如果我们希望我们之前的测试不使用 Eureka，以保持通过，**我们将需要手动设置 eureka.client.enabled 为** `false`。这样，即使库在路径上，`BooksClient`也不会尝试使用 Eureka 来定位服务，而是使用 Ribbon 配置。

### 6.3.整合测试

同样，我们已经有了所有需要的配置，所以让我们将它们放在一起进行测试:

```java
@ActiveProfiles("eureka-test")
@EnableConfigurationProperties
@ExtendWith(SpringExtension.class)
@SpringBootTest(classes = Application.class, webEnvironment =  SpringBootTest.WebEnvironment.RANDOM_PORT)
@ContextConfiguration(classes = { MockBookServiceConfig.class }, 
  initializers = { EurekaContainerConfig.Initializer.class })
class ServiceDiscoveryBooksClientIntegrationTest {

    @Autowired
    private BooksClient booksClient;

    @Lazy
    @Autowired
    private EurekaClient eurekaClient;

    @BeforeEach
    void setUp() {
        await().atMost(60, SECONDS).until(() -> eurekaClient.getApplications().size() > 0);
    }

    @Test
    public void whenGetBooks_thenTheCorrectBooksAreReturned() {
        List books = booksClient.getBooks();

        assertEquals(1, books.size());
        assertEquals(
          new Book("Hitchhiker's guide to the galaxy", "Douglas Adams"), 
          books.stream().findFirst().get());
    }

}
```

在这个测试中发生了一些事情。我们一个一个来看。

首先，`EurekaContainerConfig`中的上下文初始化器启动了 Eureka 服务。

然后，`SpringBootTest`启动`books-service`应用程序，公开在`MockBookServiceConfig`中定义的控制器。

因为**Eureka 容器和 web 应用程序的启动可能需要几秒钟**，所以我们需要等到`books-service`被注册。这发生在测试的`setUp`中。

最后，tests 方法验证了 BooksClient 确实可以与 Eureka 配置一起正常工作。

## 7.结论

在本文中，**我们探索了为 Spring Cloud Feign Client** 编写集成测试的不同方法。我们从一个基本客户端开始，在 WireMock 的帮助下进行测试。之后，我们继续使用 Ribbon 添加负载平衡。我们编写了一个集成测试，并确保我们的 Feign 客户端能够与 Ribbon 提供的客户端负载平衡一起正常工作。最后，我们在组合中添加了 Eureka 服务发现。再一次，我们确保我们的客户仍然像预期的那样工作。

和往常一样，完整的代码可以在 GitHub 上找到[。](https://web.archive.org/web/20221126220215/https://github.com/eugenp/tutorials/tree/master/spring-cloud-modules/spring-cloud-eureka)