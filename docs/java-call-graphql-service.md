# 从 Java 应用程序调用 GraphQL 服务

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-call-graphql-service>

## 1.概观

GraphQL 是一个相对**的新概念，用于构建 web 服务作为 REST** 的替代。最近出现了一些用于创建和调用 GraphQL 服务的 Java 库。

在本教程中，我们将研究 GraphQL 模式、查询和变异。我们将看到如何用普通 Java 创建和模拟一个简单的 GraphQL 服务器。然后，我们将探索如何使用众所周知的 HTTP 库调用 GraphQL 服务。

最后，我们还将探索可用于进行 GraphQL 服务调用的第三方库。

## 2.GraphQL

GraphQL 是一种用于 web 服务的**查询语言，也是使用类型系统**执行查询的服务器端运行时。

GraphQL 服务器使用 GraphQL 模式指定 API 功能。这允许 GraphQL 客户端准确地指定从 API 中检索哪些数据。这可能包括子资源和单个请求中的多个查询。

### 2.1.GraphQL 模式

GraphQL 服务器用一组类型定义服务。这些类型**描述了您可以使用服务**查询的一组可能的数据。

GraphQL 服务可以用任何语言编写。然而，GraphQL 模式需要使用称为 GraphQL 模式语言的 DSL 来定义。

在我们的示例 GraphQL 模式中，我们将定义两种类型(`Book`和`Author`)和一个获取所有书籍的查询操作(`allBooks`):

```java
type Book {
    title: String!
    author: Author
}

type Author {
    name: String!
    surname: String!
}

type Query {
    allBooks: [Book]
}

schema {
    query: Query
}
```

`Query`类型很特殊，因为它定义了 GraphQL 查询的入口点。

### 2.2.查询和突变

GraphQL 服务是通过**定义类型和字段，以及为不同的字段**提供函数来创建的。

最简单的形式是，GraphQL 询问对象上的特定字段。例如，我们可以查询获取所有书名:

```java
{
    "allBooks" {
        "title"
    }
}
```

尽管看起来很相似，但这不是 JSON。这是一种特殊的 GraphQL 查询格式，支持参数、别名、变量等等。

GraphQL 服务将使用 JSON 格式的响应来响应上述查询，如下所示:

```java
{
    "data": {
        "allBooks": [
            {
                "title": "Title 1"
            },
            {
                "title": "Title 2"
            }
        ]
    }
}
```

在本教程中，我们将关注使用查询获取数据。然而，重要的是要提到 GraphQL 中的另一个特殊概念——变异。

任何可能导致修改的操作都使用变异类型发送。

## 3.GraphQL 服务器

让我们使用上面定义的模式用 Java 创建一个简单的 GraphQL 服务器。我们将利用 **[GraphQL Java](https://web.archive.org/web/20220613105332/https://www.graphql-java.com/) 库来实现 GraphQL 服务器**。

我们将从定义 GraphQL 查询开始，并实现我们的示例 GraphQL 模式中指定的`allBooks`方法:

```java
public class GraphQLQuery implements GraphQLQueryResolver {

    private BookRepository repository;

    public GraphQLQuery(BookRepository repository) {
        this.repository = repository;
    }

    public List<Book> allBooks() {
        return repository.getAllBooks();
    }

}
```

接下来，为了公开我们的 GraphQL 端点，我们将创建一个 web servlet:

```java
@WebServlet(urlPatterns = "/graphql")
public class GraphQLEndpoint extends HttpServlet {

    private SimpleGraphQLHttpServlet graphQLServlet;

    @Override
    protected void service(HttpServletRequest req, HttpServletResponse resp) 
      throws ServletException, IOException {
        graphQLServlet.service(req, resp);
    }

    @Override
    public void init() {
        GraphQLSchema schema = SchemaParser.newParser()
          .resolvers(new GraphQLQuery(new BookRepository()))
          .file("schema.graphqls")
          .build()
          .makeExecutableSchema();
        graphQLServlet = SimpleGraphQLHttpServlet
          .newBuilder(schema)
          .build();
    }
}
```

在 servlet `init`方法中，我们将解析位于 resources 文件夹中的 GraphQL 模式。最后，使用解析的模式，我们可以创建一个`SimpleGraphQLHttpServlet`的实例。

我们将使用 [`maven-war-plugin`](https://web.archive.org/web/20220613105332/https://mvnrepository.com/artifact/org.apache.maven.plugins/maven-war-plugin) 来打包我们的应用程序，并使用 [`jetty-maven-plugin`](https://web.archive.org/web/20220613105332/https://mvnrepository.com/artifact/org.eclipse.jetty/jetty-maven-plugin) 来运行它:

```java
mvn jetty:run
```

现在，我们已经准备好运行和测试我们的 GraphQL 服务了，方法是发送一个请求到:

```java
http://localhost:8080/graphql?query={allBooks{title}}
```

## 4.HTTP 客户端

与 REST 服务一样，GraphQL 服务是通过 HTTP 协议公开的。因此，我们可以使用任何 Java HTTP 客户端来调用 GraphQL 服务。

### 4.1.发送请求

让我们试着**发送一个请求到我们在上一节**中创建的 GraphQL 服务:

```java
public static HttpResponse callGraphQLService(String url, String query) 
  throws URISyntaxException, IOException {
    HttpClient client = HttpClientBuilder.create().build();
    HttpGet request = new HttpGet(url);
    URI uri = new URIBuilder(request.getURI())
      .addParameter("query", query)
      .build();
    request.setURI(uri);
    return client.execute(request);
}
```

在我们的例子中，我们使用了 [Apache HttpClient](/web/20220613105332/https://www.baeldung.com/httpclient4) 。但是，任何 Java HTTP 客户端都可以使用。

### 4.2.解析响应

接下来，让我们解析来自 GraphQL 服务的响应。 **GraphQL 服务发送 JSON 格式的响应**，与 REST 服务相同:

```java
HttpResponse httpResponse = callGraphQLService(serviceUrl, "{allBooks{title}}");
String actualResponse = IOUtils.toString(httpResponse.getEntity().getContent(), StandardCharsets.UTF_8.name());
Response parsedResponse = objectMapper.readValue(actualResponse, Response.class);
assertThat(parsedResponse.getData().getAllBooks()).hasSize(2);
```

在我们的例子中，我们使用了流行的 [Jackson](/web/20220613105332/https://www.baeldung.com/jackson-annotations) 库中的`[ObjectMapper](/web/20220613105332/https://www.baeldung.com/jackson-object-mapper-tutorial)` 。然而，我们可以使用任何 Java 库进行 JSON 序列化/反序列化。

### 4.3.嘲弄的回应

与任何其他通过 HTTP 公开的服务一样，**我们可以模拟 GraphQL 服务器的响应来进行测试**。

我们可以利用 [MockServer](/web/20220613105332/https://www.baeldung.com/mockserver) 库来存根外部 GraphQL HTTP 服务:

```java
String requestQuery = "{allBooks{title}}";
String responseJson = "{\"data\":{\"allBooks\":[{\"title\":\"Title 1\"},{\"title\":\"Title 2\"}]}}";

new MockServerClient(SERVER_ADDRESS, serverPort)
    .when(
      request()
        .withPath(PATH)
        .withQueryStringParameter("query", requestQuery),
      exactly(1)
    )
    .respond(
      response()
        .withStatusCode(HttpStatusCode.OK_200.code())
        .withBody(responseJson)
    );
```

我们的示例模拟服务器将接受一个 GraphQL 查询作为参数，并在主体中用一个 JSON 响应进行响应。

## 5.外部库

最近出现了一些 Java GraphQL 库，它们允许更简单的 GraphQL 服务调用。

### 5.1.美国运通`Nodes`

`Nodes`是美国运通的 GraphQL 客户端，设计用于**从标准模型定义**构建查询。要开始使用它，我们应该首先添加所需的[依赖项](https://web.archive.org/web/20220613105332/https://jitpack.io/p/americanexpress/nodes):

```java
<dependency>
    <groupId>com.github.americanexpress.nodes</groupId>
    <artifactId>nodes</artifactId>
    <version>0.5.0</version>>
</dependency>
```

该库目前托管在`JitPack`上，我们也应该将它添加到我们的 Maven 安装库:

```java
<repository>
    <id>jitpack.io</id>
    <url>https://jitpack.io</url>
</repository>
```

一旦依赖关系得到解决，我们就可以利用`GraphQLTemplate`来构造一个查询并调用我们的 GraphQL 服务:

```java
public static GraphQLResponseEntity<Data> callGraphQLService(String url, String query)
  throws IOException {
    GraphQLTemplate graphQLTemplate = new GraphQLTemplate();

    GraphQLRequestEntity requestEntity = GraphQLRequestEntity.Builder()
      .url(StringUtils.join(url, "?query=", query))
      .request(Data.class)
      .build();

    return graphQLTemplate.query(requestEntity, Data.class);
}
```

`Nodes`将使用我们指定的类解析来自 GraphQL 服务的响应:

```java
GraphQLResponseEntity<Data> responseEntity = callGraphQLService(serviceUrl, "{allBooks{title}}");
assertThat(responseEntity.getResponse().getAllBooks()).hasSize(2);
```

我们应该注意到,`Nodes`仍然要求我们构造自己的 DTO 类来解析响应。

### 5.2.GraphQL Java 生成器

`[GraphQL Java Generator](https://web.archive.org/web/20220613105332/https://github.com/graphql-java-generator/graphql-maven-plugin-project)`库利用**的能力生成基于 GraphQL 模式**的 Java 代码。

这种方法类似于 SOAP 服务中使用的 WSDL 代码生成器。要开始使用它，我们应该首先添加所需的[依赖关系](https://web.archive.org/web/20220613105332/https://search.maven.org/search?q=com.graphql-java-generator):

```java
<dependency>
    <groupId>com.graphql-java-generator</groupId>
    <artifactId>graphql-java-runtime</artifactId>
    <version>1.18</version>
</dependency>
```

接下来，我们可以配置 `graphql-maven-plugin`来执行一个`generateClientCode`目标:

```java
<plugin>
    <groupId>com.graphql-java-generator</groupId>
    <artifactId>graphql-maven-plugin</artifactId>
    <version>1.18</version>
    <executions>
        <execution>
            <goals>
                <goal>generateClientCode</goal>
            </goals>
        </execution>
    </executions>
    <configuration>
        <packageName>com.baeldung.graphql.generated</packageName>
        <copyRuntimeSources>false</copyRuntimeSources>
        <generateDeprecatedRequestResponse>false</generateDeprecatedRequestResponse>
        <separateUtilityClasses>true</separateUtilityClasses>
    </configuration>
</plugin>
```

一旦我们运行 Maven build 命令，插件将生成调用 GraphQL 服务所需的 dto 和实用程序类。

生成的`QueryExecutor`组件将包含调用我们的 GraphQL 服务并解析其响应的方法:

```java
public List<Book> allBooks(String queryResponseDef, Object... paramsAndValues)
  throws GraphQLRequestExecutionException, GraphQLRequestPreparationException {
    logger.debug("Executing query 'allBooks': {} ", queryResponseDef);
    ObjectResponse objectResponse = getAllBooksResponseBuilder()
      .withQueryResponseDef(queryResponseDef).build();
    return allBooksWithBindValues(objectResponse, 
      graphqlClientUtils.generatesBindVariableValuesMap(paramsAndValues));
}
```

然而，它是为使用 [Spring](/web/20220613105332/https://www.baeldung.com/spring-tutorial) 框架而构建的。

## 6.结论

在本文中，我们探讨了如何从 Java 应用程序调用 GraphQL 服务。

我们学习了如何创建和模拟一个简单的 GraphQL 服务器。接下来，我们看到了如何使用标准 HTTP 库发送请求并从 GraphQL 服务器检索响应。我们还看到了如何将来自 JSON 的 GraphQL 服务响应解析为 Java 对象。

最后，我们查看了两个可用于进行 GraphQL 服务调用的第三方库，`Nodes`和`GraphQL Java Generator`。

和往常一样，完整的源代码可以在 GitHub 上找到[。](https://web.archive.org/web/20220613105332/https://github.com/eugenp/tutorials/tree/master/graphql-modules/graphql-java)