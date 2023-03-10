# GraphQL SPQR 和 Spring Boot 入门

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-boot-graphql-spqr>

## 1.介绍

GraphQL 是一种用于 web APIs 的查询和操作语言。起源于让 GraphQL 工作更加无缝的库之一是 [SPQR](https://web.archive.org/web/20220707145328/https://github.com/leangen/graphql-spqr) 。

在本教程中，我们将学习 GraphQL SPQR 的基础知识，并在一个简单的 Spring Boot 项目中看到它的实际应用。

## 2.什么是 GraphQL SPQR？

GraphQL 是由脸书创建的著名查询语言。其核心是模式——我们在其中定义自定义类型和函数的文件。

在传统方法中，如果我们想要将 GraphQL 添加到我们的项目中，我们必须遵循两个步骤。首先，我们必须将 GraphQL 模式文件添加到项目中。其次，我们需要编写各自的 Java POJOs 来代表模式中的每种类型。这意味着我们将在两个地方维护相同的信息:模式文件和 Java 类。这种方法容易出错，并且在维护项目时需要更多的努力。

GraphQL Schema Publisher & Query Resolver，简称为 **SPQR，是为了减少上述问题而产生的**——它只是从带注释的 Java 类中生成 graph QL 模式。

## 3.Spring Boot 推出 GraphQL SPQR

为了查看 SPQR 的运行情况，我们将设置一个简单的服务。我们将使用 [Spring Boot GraphQL Starter](https://web.archive.org/web/20220707145328/https://mvnrepository.com/artifact/com.graphql-java/graphql-spring-boot-starter/5.0.2) 和 GraphQL SPQR。

### 3.1.设置

让我们从将 SPQR 和 Spring Boot 的依赖项添加到 POM 开始:

```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-test</artifactId>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>io.leangen.graphql</groupId>
    <artifactId>spqr</artifactId>
    <version>0.11.2</version>
</dependency>
```

### 3.2.编写模型`Book`类

现在我们已经添加了必要的依赖项，让我们创建一个简单的`Book`类:

```java
public class Book {
    private Integer id;
    private String author;
    private String title;
}
```

正如我们在上面看到的，它不包括任何 SPQR 注释。如果我们没有源代码，但想从这个库中受益，这将非常有用。

### 3.3.编写图书服务

为了管理藏书，让我们创建一个`IBookService`界面:

```java
public interface IBookService {
    Book getBookWithTitle(String title);

    List<Book> getAllBooks();

    Book addBook(Book book);

    Book updateBook(Book book);

    boolean deleteBook(Book book);
} 
```

然后，我们将提供接口的实现:

```java
@Service
public class BookService implements IBookService {

    Set<Book> books = new HashSet<>();

    public Book getBookWithTitle(String title) {
        return books.stream()
            .filter(book -> book.getTitle()
                .equals(title))
            .findFirst()
            .orElse(null);
    }

    public List<Book> getAllBooks() {
        return books.stream()
            .collect(Collectors.toList());
    }

    public Book addBook(Book book) {
        books.add(book);
        return book;
    }

    public Book updateBook(Book book) {
        books.remove(book);
        books.add(book);
        return book;
    }

    public boolean deleteBook(Book book) {
        return books.remove(book);
    }
}
```

### 3.4.用 graphql-spqr 公开服务

唯一剩下的事情是创建一个解析器，它将暴露 GraphQL 的变化和查询。**为此，我们将使用两个重要的 SPQR 注释—`@GraphQLMutation`和`@GraphQLQuery` :**

```java
@Service
public class BookResolver {

    @Autowired
    IBookService bookService;

    @GraphQLQuery(name = "getBookWithTitle")
    public Book getBookWithTitle(@GraphQLArgument(name = "title") String title) {
        return bookService.getBookWithTitle(title);
    }

    @GraphQLQuery(name = "getAllBooks", description = "Get all books")
    public List<Book> getAllBooks() {
        return bookService.getAllBooks();
    }

    @GraphQLMutation(name = "addBook")
    public Book addBook(@GraphQLArgument(name = "newBook") Book book) {
        return bookService.addBook(book);
    }

    @GraphQLMutation(name = "updateBook")
    public Book updateBook(@GraphQLArgument(name = "modifiedBook") Book book) {
        return bookService.updateBook(book);
    }

    @GraphQLMutation(name = "deleteBook")
    public void deleteBook(@GraphQLArgument(name = "book") Book book) {
        bookService.deleteBook(book);
    }
}
```

如果我们不想在每个方法中都写`@GraphQLArgument`并且满足于 GraphQL 参数被命名为输入参数，我们可以用`-parameters`参数编译代码。

### 3.5.休息控制器

最后，我们将定义一个 Spring `@RestController.` **为了用 SPQR 公开服务，我们将配置`GraphQLSchema`和`GraphQL`对象:**

```java
@RestController
public class GraphqlController {

    private final GraphQL graphQL;

    @Autowired
    public GraphqlController(BookResolver bookResolver) {
        GraphQLSchema schema = new GraphQLSchemaGenerator()
          .withBasePackages("com.baeldung")
          .withOperationsFromSingleton(bookResolver)
          .generate();
        this.graphQL = new GraphQL.Builder(schema)
          .build();
    }
```

需要注意的是**我们必须将`BookResolver`注册为单例**。

SPQR 旅程中的最后一项任务是创建一个`/graphql`端点。它将作为与我们服务的单点联系，并将执行请求的查询和变更:

```java
@PostMapping(value = "/graphql")
    public Map<String, Object> execute(@RequestBody Map<String, String> request, HttpServletRequest raw)
      throws GraphQLException {
        ExecutionResult result = graphQL.execute(request.get("query"));
        return result.getData();
    }
}
```

### 3.6.结果

我们可以通过检查`/graphql`端点来检查结果。例如，让我们通过执行以下 cURL 命令来检索所有的`Book`记录:

```java
curl -g \
  -X POST \
  -H "Content-Type: application/json" \
  -d '{"query":"{getAllBooks {id author title }}"}' \
  http://localhost:8080/graphql
```

### 3.7.试验

一旦我们完成了配置，我们就可以测试我们的项目。我们将使用 [`MockMvc`](https://web.archive.org/web/20220707145328/https://spring.io/guides/gs/testing-web/) 来测试我们的新端点并验证响应。让我们定义 JUnit 测试并自动连接所需的服务:

```java
@RunWith(SpringRunner.class)
@SpringBootTest
@AutoConfigureMockMvc
public class GraphqlControllerIntegrationTest {

    @Autowired
    private MockMvc mockMvc;

    @Autowired
    BookService bookService;

    private static final String GRAPHQL_PATH = "/graphql";

    @Test
    public void givenNoBooks_whenReadAll_thenStatusIsOk() throws Exception {

        String getAllBooksQuery = "{ getAllBooks {id author title } }";

        this.mockMvc.perform(post(GRAPHQL_PATH).content(toJSON(getAllBooksQuery))
            .contentType(
                MediaType.APPLICATION_JSON))
            .andExpect(status().isOk())
            .andExpect(jsonPath("$.getAllBooks").isEmpty());
    }

    @Test
    public void whenAddBook_thenStatusIsOk() throws Exception {

        String addBookMutation = "mutation { addBook(newBook: {id: 123, author: \"J.R.R. Tolkien\", "
            + "title: \"The Lord of the Rings\"}) { id author title } }";

        this.mockMvc.perform(post(GRAPHQL_PATH).content(toJSON(addBookMutation))
            .contentType(MediaType.APPLICATION_JSON))
            .andExpect(status().isOk())
            .andExpect(jsonPath("$.addBook.id").value("123"))
            .andExpect(jsonPath("$.addBook.author").value("J.R.R. Tolkien"))
            .andExpect(jsonPath("$.addBook.title").value("The Lord of the Rings"));
    }

    private String toJSON(String query) throws JSONException {
        JSONObject jsonObject = new JSONObject();
        jsonObject.put("query", query);
        return jsonObject.toString();
    }
}
```

## 4.使用 GraphQL SPQR Spring Boot 启动器

致力于 SPQR 的团队已经创建了一个 Spring Boot 启动器，这使得使用它更加容易。我们去看看吧！

### 4.1.设置

我们将从向 POM 添加`spqr-spring-boot-starter`开始:

```java
<dependency>
    <groupId>io.leangen.graphql</groupId>
    <artifactId>graphql-spqr-spring-boot-starter</artifactId>
    <version>0.0.6</version>
</dependency>
```

### 4.2.图书服务

然后，我们需要给我们的`BookService`添加两个修改。首先，它必须用`@GraphQLApi`标注。此外，我们希望在 API 中公开的每个方法都必须有各自的注释:

```java
@Service
@GraphQLApi
public class BookService implements IBookService {

    Set<Book> books = new HashSet<>();

    @GraphQLQuery(name = "getBookWithTitle")
    public Book getBookWithTitle(@GraphQLArgument(name = "title") String title) {
        return books.stream()
            .filter(book -> book.getTitle()
                .equals(title))
            .findFirst()
            .orElse(null);
    }

    @GraphQLQuery(name = "getAllBooks", description = "Get all books")
    public List<com.baeldung.sprq.Book> getAllBooks() {
        return books.stream()
            .toList();
    }

    @GraphQLMutation(name = "addBook")
    public Book addBook(@GraphQLArgument(name = "newBook") Book book) {
        books.add(book);
        return book;
    }

    @GraphQLMutation(name = "updateBook")
    public Book updateBook(@GraphQLArgument(name = "modifiedBook") Book book) {
        books.remove(book);
        books.add(book);
        return book;
    }

    @GraphQLMutation(name = "deleteBook")
    public boolean deleteBook(@GraphQLArgument(name = "book") Book book) {
        return books.remove(book);
    }
}
```

正如我们所看到的，我们基本上将代码从`BookResolver`移到了`BookService`。另外，**我们不需要`GraphqlController`类——一个`/graphql`端点会被自动添加**。

## 5.摘要

GraphQL 是一个令人兴奋的框架，是传统 RESTful 端点的替代方案。虽然提供了很大的灵活性，但是它也增加了一些繁琐的任务，比如维护模式文件。SPQR 致力于使 GraphQL 的使用更简单，更少出错。

在本文中，我们看到了如何将 SPQR 添加到现有的 POJOs 中，并将其配置为服务于查询和突变。然后，我们在 GraphiQL 中看到了一个新的端点。最后，我们使用 Spring 的 MockMvc 和 JUnit 测试了我们的代码。

和往常一样，这里使用的示例代码可以从 GitHub 上的[处获得。此外，GraphQL Spring Boot 初学者工具包的代码可以在 GitHub](https://web.archive.org/web/20220707145328/https://github.com/eugenp/tutorials/tree/master/spring-boot-modules/spring-boot-libraries-2) 的[上找到。](https://web.archive.org/web/20220707145328/https://github.com/eugenp/tutorials/tree/master/graphql-modules/graphql-spqr)