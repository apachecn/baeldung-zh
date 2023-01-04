# 测试 Quarkus 应用程序

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-quarkus-testing>

## 1.概观

Quarkus 使得开发健壮和干净的应用程序变得非常容易。但是测试呢？

在本教程中，**我们将仔细看看如何测试 Quarkus 应用程序**。我们将探索 Quarkus 和**提供的测试可能性，展示诸如依赖管理和注入、嘲讽、概要文件配置等概念，以及更具体的东西，如 Quarkus 注释和测试本机可执行文件**。

## 2.设置

让我们从我们之前的 QuarkusIO 指南[中配置的基本 Quarkus 项目开始。](/web/20221205203711/https://www.baeldung.com/quarkus-io)

首先，我们将添加[quar kus-reasteasy-Jackson](https://web.archive.org/web/20221205203711/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22io.quarkus%22%20a%3A%22quarkus-resteasy-jackson%22)、[quar kus-hibernate-ORM-panache](https://web.archive.org/web/20221205203711/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22io.quarkus%22%20a%3A%22quarkus-hibernate-orm-panache%22)、 [quarkus-jdbc-h2](https://web.archive.org/web/20221205203711/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22io.quarkus%22%20a%3A%22quarkus-jdbc-h2%22) 、 [quarkus-junit5-mockito](https://web.archive.org/web/20221205203711/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22io.quarkus%22%20a%3A%22quarkus-junit5-mockito%22) 和 [quarkus-test-h2](https://web.archive.org/web/20221205203711/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22io.quarkus%22%20a%3A%22quarkus-test-h2%22) Maven 依赖项:

```
<dependency>
    <groupId>io.quarkus</groupId>
    <artifactId>quarkus-resteasy-jackson</artifactId>
</dependency>
<dependency>
    <groupId>io.quarkus</groupId>
    <artifactId>quarkus-hibernate-orm-panache</artifactId>
</dependency>
<dependency>
    <groupId>io.quarkus</groupId>
    <artifactId>quarkus-jdbc-h2</artifactId>
</dependency>
<dependency>
    <groupId>io.quarkus</groupId>
    <artifactId>quarkus-junit5-mockito</artifactId>
</dependency>
<dependency>
    <groupId>io.quarkus</groupId>
    <artifactId>quarkus-test-h2</artifactId>
</dependency>
```

接下来，让我们创建我们的域实体:

```
public class Book extends PanacheEntity {
    private String title;
    private String author;
}
```

我们继续添加一个简单的 Panache 存储库，以及一个搜索书籍的方法:

```
public class BookRepository implements PanacheRepository {

    public Stream<Book> findBy(String query) {
        return find("author like :query or title like :query", with("query", "%"+query+"%")).stream();
    }
}
```

现在，让我们编写一个`LibraryService` 来容纳任何业务逻辑:

```
public class LibraryService {

    public Set<Book> find(String query) {
        if (query == null) {
            return bookRepository.findAll().stream().collect(toSet());
        }
        return bookRepository.findBy(query).collect(toSet());
    }
}
```

最后，让我们通过创建一个`LibraryResource`来通过 HTTP 公开我们的服务功能:

```
@Path("/library")
public class LibraryResource {

    @GET
    @Path("/book")
    public Set findBooks(@QueryParam("query") String query) {
        return libraryService.find(query);
    }
}
```

## 3.`@Alternative`实施

在编写任何测试之前，让我们确保我们的存储库中有一些书。有了 Quarkus，**我们可以使用 CDI `@Alternative`机制为我们的测试**提供一个定制的 bean 实现。让我们创建一个扩展了`BookRepository`的`TestBookRepository`:

```
@Priority(1)
@Alternative
@ApplicationScoped
public class TestBookRepository extends BookRepository {

    @PostConstruct
    public void init() {
        persist(new Book("Dune", "Frank Herbert"),
          new Book("Foundation", "Isaac Asimov"));
    }

}
```

我们将这个备选 bean 放在我们的`test`包中，因为有了`@Priority(1)`和`@Alternative`注释，我们确信任何测试都会在实际的`BookRepository`实现中发现它。这个**是我们提供全局模拟的一种方式，所有的 Quarkus 测试**都可以使用。我们将很快探索更多的狭窄焦点模拟，但是现在，让我们继续创建我们的第一个测试。

## 4.HTTP 集成测试

让我们从创建一个简单放心的集成测试开始:

```
@QuarkusTest
class LibraryResourceIntegrationTest {

    @Test
    void whenGetBooksByTitle_thenBookShouldBeFound() {

        given().contentType(ContentType.JSON).param("query", "Dune")
          .when().get("/library/book")
          .then().statusCode(200)
          .body("size()", is(1))
          .body("title", hasItem("Dune"))
          .body("author", hasItem("Frank Herbert"));
    }
}
```

这个用`@QuarkusTest,`标注的**测试首先启动 Quarkus 应用程序**，然后针对我们的资源端点执行一系列 HTTP 请求。

现在，让我们利用一些 Quarkus 机制来尝试并进一步改进我们的测试。

### 4.1.用`@TestHTTPResource`注入 URL

让我们注入资源 URL，而不是硬编码我们的 HTTP 端点的路径:

```
@TestHTTPResource("/library/book")
URL libraryEndpoint;
```

然后，让我们在请求中使用它:

```
given().param("query", "Dune")
  .when().get(libraryEndpoint)
  .then().statusCode(200);
```

或者，不使用放心，让我们简单地打开一个到注入的 URL 的连接并测试响应:

```
@Test
void whenGetBooks_thenBooksShouldBeFound() throws IOException {
    assertTrue(IOUtils.toString(libraryEndpoint.openStream(), defaultCharset()).contains("Asimov"));
}
```

正如我们所见，`@TestHTTPResource` URL 注入为我们提供了一种简单灵活的访问端点的方式。

### 4.2.`@TestHTTPEndpoint`

让我们更进一步，使用 Quarkus 提供的`@TestHTTPEndpoint`注释来配置我们的端点:

```
@TestHTTPEndpoint(LibraryResource.class)
@TestHTTPResource("book")
URL libraryEndpoint;
```

这样，如果我们决定改变`LibraryResource`的路径，测试将会选择正确的路径，而不需要我们去接触它。

**`@TestHTTPEndpoint` 也可以应用于类级别，在这种情况下，放心将自动为所有请求加上`LibraryResource`的`Path`** :

```
@QuarkusTest
@TestHTTPEndpoint(LibraryResource.class)
class LibraryHttpEndpointIntegrationTest {

    @Test
    void whenGetBooks_thenShouldReturnSuccessfully() {
        given().contentType(ContentType.JSON)
          .when().get("book")
          .then().statusCode(200);
    }
}
```

## 5.上下文和依赖注入

谈到依赖注入，在 Quarkus 测试中的**，我们可以用`@Inject`来表示任何需要的依赖**。让我们通过为我们的`LibraryService`创建一个测试来看看这一点:

```
@QuarkusTest
class LibraryServiceIntegrationTest {

    @Inject
    LibraryService libraryService;

    @Test
    void whenFindByAuthor_thenBookShouldBeFound() {
        assertFalse(libraryService.find("Frank Herbert").isEmpty());
    }
}
```

现在，让我们试着测试一下我们的派头:

```
class BookRepositoryIntegrationTest {

    @Inject
    BookRepository bookRepository;

    @Test
    void givenBookInRepository_whenFindByAuthor_thenShouldReturnBookFromRepository() {
        assertTrue(bookRepository.findBy("Herbert").findAny().isPresent());
    }
}
```

但是当我们运行测试时，它失败了。这是因为它**需要在一个事务**的上下文中运行，并且没有活动的事务。这可以简单地通过在测试类中添加`@Transactional`来解决。或者，如果我们愿意，我们可以定义我们自己的原型来捆绑`@QuarkusTest`和`@Transactional.` 让我们通过创建`@QuarkusTransactionalTest`注释来实现这一点:

```
@QuarkusTest
@Stereotype
@Transactional
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
public @interface QuarkusTransactionalTest {
}
```

现在，让我们将它应用到我们的测试中:

```
@QuarkusTransactionalTest
class BookRepositoryIntegrationTest
```

正如我们所见，因为 **Quarkus 测试是完整的 CDI bean**，我们可以利用所有的 CDI 优势，比如依赖注入、事务上下文和 CDI 拦截器。

## 6.嘲弄的

模仿是任何测试工作的一个重要方面。正如我们在上面已经看到的，Quarkus 测试可以利用 CDI `@Alternative`机制。现在让我们更深入地了解 Quarkus 提供的嘲讽功能。

### 6.1.`@Mock`

作为**对`@Alternative`方法**的略微简化，我们可以使用`@Mock`原型注释。这将`@Alternative`和`@Primary(1)`注释捆绑在一起。

### 6.2.`@QuarkusMock`

如果我们不想要一个全局定义的模拟，而是希望**只在一个测试**的范围内进行模拟，我们可以使用`@QuarkusMock`:

```
@QuarkusTest
class LibraryServiceQuarkusMockUnitTest {

    @Inject
    LibraryService libraryService;

    @BeforeEach
    void setUp() {
        BookRepository mock = Mockito.mock(TestBookRepository.class);
        Mockito.when(mock.findBy("Asimov"))
          .thenReturn(Arrays.stream(new Book[] {
            new Book("Foundation", "Isaac Asimov"),
            new Book("I Robot", "Isaac Asimov")}));
        QuarkusMock.installMockForType(mock, BookRepository.class);
    }

    @Test
    void whenFindByAuthor_thenBooksShouldBeFound() {
        assertEquals(2, libraryService.find("Asimov").size());
    }
}
```

### 6.3.`@InjectMock`

让我们稍微简化一下，**使用 Quarkus `@InjectMock` 注释代替`@QuarkusMock`** :

```
@QuarkusTest
class LibraryServiceInjectMockUnitTest {

    @Inject
    LibraryService libraryService;

    @InjectMock
    BookRepository bookRepository;

    @BeforeEach
    void setUp() {
        when(bookRepository.findBy("Frank Herbert"))
          .thenReturn(Arrays.stream(new Book[] {
            new Book("Dune", "Frank Herbert"),
            new Book("Children of Dune", "Frank Herbert")}));
    }

    @Test
    void whenFindByAuthor_thenBooksShouldBeFound() {
        assertEquals(2, libraryService.find("Frank Herbert").size());
    }
}
```

### 6.4.`@InjectSpy`

如果我们只对监视感兴趣，而不是替换 bean 行为，我们可以使用提供的`@InjectSpy`注释:

```
@QuarkusTest
class LibraryResourceInjectSpyIntegrationTest {

    @InjectSpy
    LibraryService libraryService;

    @Test
    void whenGetBooksByAuthor_thenBookShouldBeFound() {
        given().contentType(ContentType.JSON).param("query", "Asimov")
          .when().get("/library/book")
          .then().statusCode(200);

        verify(libraryService).find("Asimov");
    }

}
```

## 7.测试配置文件

我们可能希望**在不同的配置下运行我们的测试**。为此， **Quarkus 提出了测试配置文件**的概念。让我们创建一个测试，使用定制版本的`BookRepository`在不同的数据库引擎上运行，这也将在不同于已经配置的路径上公开我们的 HTTP 资源。

为此，我们从实现一个`QuarkusTestProfile`开始:

```
public class CustomTestProfile implements QuarkusTestProfile {

    @Override
    public Map<String, String> getConfigOverrides() {
        return Collections.singletonMap("quarkus.resteasy.path", "/custom");
    }

    @Override
    public Set<Class<?>> getEnabledAlternatives() {
        return Collections.singleton(TestBookRepository.class);
    }

    @Override
    public String getConfigProfile() {
        return "custom-profile";
    }
}
```

现在让我们通过添加一个`custom-profile` config 属性来配置我们的`application.properties`，该属性将把我们的 H2 存储从内存更改为文件:

```
%custom-profile.quarkus.datasource.jdbc.url = jdbc:h2:file:./testdb
```

最后，有了所有的资源和配置，让我们编写测试:

```
@QuarkusTest
@TestProfile(CustomBookRepositoryProfile.class)
class CustomLibraryResourceManualTest {

    public static final String BOOKSTORE_ENDPOINT = "/custom/library/book";

    @Test
    void whenGetBooksGivenNoQuery_thenAllBooksShouldBeReturned() {
        given().contentType(ContentType.JSON)
          .when().get(BOOKSTORE_ENDPOINT)
          .then().statusCode(200)
          .body("size()", is(2))
          .body("title", hasItems("Foundation", "Dune"));
    }
}
```

正如我们从`@TestProfile`注释中看到的，这个测试将使用`CustomTestProfile` `.` ，它将向在概要文件的`getConfigOverrides`方法中被覆盖的自定义端点发出 HTTP 请求。此外，它将使用在`getEnabledAlternatives`方法中配置的替代图书仓库实现。最后，通过使用`getConfigProfile`中定义的`custom-profile` ，它将把数据保存在文件中，而不是内存中。

需要注意的一点是 **Quarkus 将关闭，然后在测试执行**之前用新的配置文件重启。这增加了关机/重启的时间，但这是额外灵活性的代价。

## 8.测试本机可执行文件

Quarkus 提供了测试本地可执行文件的可能性。让我们创建一个本机映像测试:

```
@NativeImageTest
@QuarkusTestResource(H2DatabaseTestResource.class)
class NativeLibraryResourceIT extends LibraryHttpEndpointIntegrationTest {
}
```

现在，通过运行:

```
mvn verify -Pnative
```

我们将看到正在构建的本机映像以及针对它运行的测试。

`@NativeImageTest` 注释指示 Quarkus 对本机映像运行这个测试，而`@QuarkusTestResource` 将在测试开始前在一个单独的进程中启动一个 H2 实例。后者是针对本机可执行文件运行测试所需要的，因为数据库引擎没有嵌入到本机映像中。

@ `QuarkusTestResource`注释也可以用来启动定制服务，比如 Testcontainers。我们所需要做的就是实现`QuarkusTestResourceLifecycleManager`接口，并用以下代码注释我们的测试:

```
@QuarkusTestResource(OurCustomResourceImpl.class)
```

您将需要一个 [GraalVM 来构建本机映像](https://web.archive.org/web/20221205203711/https://quarkus.io/guides/building-native-image#configuring-graalvm)。

此外，请注意，目前，注入不能用于本机映像测试。唯一本地运行的是 Quarkus 应用程序，而不是测试本身。

## 9.结论

在本文中，我们看到了 **Quarkus 如何为测试**我们的应用程序提供出色的支持。从简单的依赖管理、注入和模仿，到更复杂的配置文件和本机映像，Quarkus 为我们提供了许多工具来创建强大而简洁的测试。

和往常一样，完整的代码可以在 GitHub 上找到[。](https://web.archive.org/web/20221205203711/https://github.com/eugenp/tutorials/tree/master/quarkus-modules/quarkus)