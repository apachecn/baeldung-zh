# spring Data MongoDB–配置连接

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-data-mongodb-connection>

## 1.介绍

在本教程中，我们将看到配置数据库连接的不同方法。**我们将使用 [Spring Boot](/web/20221004150550/https://www.baeldung.com/spring-boot) 和[春季数据 MongoDB](/web/20221004150550/https://www.baeldung.com/spring-data-mongodb-tutorial) 。**探索 Spring 的灵活配置，我们将为每种方法创建不同的应用程序。因此，我们将能够选择最合适的一个。

## 2.测试我们的连接

在开始构建我们的应用程序之前，我们将创建一个测试类。让我们从几个我们将重复使用的常量开始:

```java
public class MongoConnectionApplicationLiveTest {
    private static final String HOST = "localhost";
    private static final String PORT = "27017";
    private static final String DB = "baeldung";
    private static final String USER = "admin";
    private static final String PASS = "password";

    // test cases
}
```

**我们的测试包括运行我们的应用程序，然后尝试在名为`“items”`的集合中插入一个文档。**插入我们的文档后，我们应该从数据库中收到一个`“_id”`，我们认为测试成功了。让我们为此创建一个助手方法:

```java
private void assertInsertSucceeds(ConfigurableApplicationContext context) {
    String name = "A";

    MongoTemplate mongo = context.getBean(MongoTemplate.class);
    Document doc = Document.parse("{\"name\":\"" + name + "\"}");
    Document inserted = mongo.insert(doc, "items");

    assertNotNull(inserted.get("_id"));
    assertEquals(inserted.get("name"), name);
}
```

我们的方法从应用程序接收 Spring 上下文，这样我们就可以检索到 [`MongoTemplate`](/web/20221004150550/https://www.baeldung.com/spring-data-mongodb-tutorial#mongotemplate-and-mongorepository) 实例。之后，我们用一个带有`Document.parse()`的字符串构建一个简单的 JSON 文档。

这样，我们不需要创建存储库或文档类。然后，在插入之后，我们断言插入的文档中的属性就是我们所期望的。

## 3.通过`application.properties`进行最小设置

我们的第一个例子是配置连接的最常见方式。**我们只需在`application.properties` :** 中提供我们的数据库信息

```java
spring.data.mongodb.host=localhost
spring.data.mongodb.port=27017
spring.data.mongodb.database=baeldung
spring.data.mongodb.username=admin
spring.data.mongodb.password=password
```

所有可用的属性都位于 Spring Boot 的`MongoProperties`类中。我们也可以使用这个类来检查默认值。**我们可以通过应用程序参数在属性文件中定义任何配置。**我们将在下一节看到这是如何工作的。

在我们的应用程序类中，我们不需要任何特殊的东西来启动和运行:

```java
@SpringBootApplication
public class SpringMongoConnectionViaPropertiesApp {

    public static void main(String... args) {
        SpringApplication.run(SpringMongoConnectionViaPropertiesApp.class, args);
    }
}
```

**这是我们连接到数据库实例所需的全部配置。**`@SpringBootApplication`标注包括 [`@EnableAutoConfiguration`](/web/20221004150550/https://www.baeldung.com/spring-boot-custom-auto-configuration) 。它负责发现我们的应用程序是基于我们的类路径的 MongoDB 应用程序。

为了测试它，我们可以使用`SpringApplicationBuilder to`获取对应用程序上下文的引用。**然后，为了断言我们的连接是有效的，我们使用前面创建的`assertInsertSucceeds`方法:**

```java
@Test
public void whenPropertiesConfig_thenInsertSucceeds() {
    SpringApplicationBuilder app = new SpringApplicationBuilder(SpringMongoConnectionViaPropertiesApp.class)
    app.run();

    assertInsertSucceeds(app.context());
}
```

最后，我们的应用程序使用我们的`application.properties`文件成功连接。

### 3.1.用命令行参数覆盖属性

当使用[命令行参数](/web/20221004150550/https://www.baeldung.com/java-command-line-arguments)运行应用程序时，我们可以覆盖我们的属性文件。当用 [java 命令](/web/20221004150550/https://www.baeldung.com/java-run-jar-with-arguments)、 [mvn 命令](/web/20221004150550/https://www.baeldung.com/spring-boot-run-maven-vs-executable-jar)或 IDE 配置运行时，这些被传递给应用程序。提供这些的方法将取决于我们使用的命令。

**让我们看一个例子[使用`mvn` 运行我们的 Spring Boot 应用](/web/20221004150550/https://www.baeldung.com/spring-boot-command-line-arguments) :**

```java
mvn spring-boot:run -Dspring-boot.run.arguments='--spring.data.mongodb.port=7017 --spring.data.mongodb.host=localhost'
```

为了使用它，我们将属性指定为参数`spring-boot.run.arguments`的值。我们使用相同的属性名，但是在它们前面加了两个破折号。**从 Spring Boot 2 号开始，多个属性要用空格隔开。**最后，在运行命令之后，不应该有错误。

以这种方式配置的选项总是优先于属性文件。当我们需要改变我们的应用程序参数而不改变我们的属性文件时，这个选项很有用。例如，如果我们的凭据发生变化，我们无法再连接。

为了在测试中模拟这一点，我们可以在运行应用程序之前设置系统属性。同样，我们可以用`properties`方法覆盖我们的`application.properties`:

```java
@Test
public void givenPrecedence_whenSystemConfig_thenInsertSucceeds() {
    System.setProperty("spring.data.mongodb.host", HOST);
    System.setProperty("spring.data.mongodb.port", PORT);
    System.setProperty("spring.data.mongodb.database", DB);
    System.setProperty("spring.data.mongodb.username", USER);
    System.setProperty("spring.data.mongodb.password", PASS);

    SpringApplicationBuilder app = new SpringApplicationBuilder(SpringMongoConnectionViaPropertiesApp.class);
      .properties(
        "spring.data.mongodb.host=oldValue",
        "spring.data.mongodb.port=oldValue",
        "spring.data.mongodb.database=oldValue",
        "spring.data.mongodb.username=oldValue",
        "spring.data.mongodb.password=oldValue"
      );
    app.run();

    assertInsertSucceeds(app.context());
}
```

因此，属性文件中的旧值不会影响我们的应用程序，因为系统属性有更高的优先级。当我们需要在不改变代码的情况下用新的连接细节重启我们的应用程序时，这很有用。

### 3.2.使用连接 URI 属性

也可以使用单个属性来代替单独的主机、端口等。：

```java
spring.data.mongodb.uri="mongodb://admin:[[email protected]](/web/20221004150550/https://www.baeldung.com/cdn-cgi/l/email-protection):27017/baeldung"
```

这个属性包括初始属性的所有值，所以我们不需要指定所有的五个值。让我们检查一下基本格式:

```java
mongodb://<username>:<password>@<host>:<port>/<database>
```

更确切地说，URI 中的`database`部分是[默认的 auth DB](https://web.archive.org/web/20221004150550/https://www.mongodb.com/docs/master/reference/connection-string/#std-label-connections-standard-connection-string-format) 。**最重要的是，不能为主机、端口和凭证单独指定`spring.data.mongodb.uri`属性。**否则，在运行我们的应用程序时，我们会得到以下错误:

```java
@Test
public void givenConnectionUri_whenAlsoIncludingIndividualParameters_thenInvalidConfig() {
    System.setProperty(
      "spring.data.mongodb.uri", 
      "mongodb://" + USER + ":" + PASS + "@" + HOST + ":" + PORT + "/" + DB
    );

    SpringApplicationBuilder app = new SpringApplicationBuilder(SpringMongoConnectionViaPropertiesApp.class)
      .properties(
        "spring.data.mongodb.host=" + HOST,
        "spring.data.mongodb.port=" + PORT,
        "spring.data.mongodb.username=" + USER,
        "spring.data.mongodb.password=" + PASS
      );

    BeanCreationException e = assertThrows(BeanCreationException.class, () -> {
        app.run();
    });

    Throwable rootCause = e.getRootCause();
    assertTrue(rootCause instanceof IllegalStateException);
    assertThat(rootCause.getMessage()
      .contains("Invalid mongo configuration, either uri or host/port/credentials/replicaSet must be specified"));
}
```

**最后，这个配置选项不仅更短，而且有时是必需的。**这是因为有些选项只能通过连接字符串获得。例如，使用 [`mongodb+srv`](https://web.archive.org/web/20221004150550/https://www.mongodb.com/developer/products/mongodb/srv-connection-strings/) 连接到一个副本集。因此，在接下来的例子中，我们将只使用这个更简单的配置属性。

## 4.使用 MongoClient 进行 Java 设置

**`MongoClient`表示我们与 MongoDB 数据库的连接，并且总是在幕后创建。但是，我们也可以通过编程来设置它。**尽管更加冗长，这种方法还是有一些优点。让我们在接下来的部分中看看它们。

### 4.1.通过`AbstractMongoClientConfiguration`连接

在我们的第一个例子中，我们将在应用程序类中扩展 Spring Data MongoDB 的`AbstractMongoClientConfiguration`类:

```java
@SpringBootApplication
public class SpringMongoConnectionViaClientApp extends AbstractMongoClientConfiguration {
    // main method
} 
```

接下来，让我们注入我们需要的属性:

```java
@Value("${spring.data.mongodb.uri}")
private String uri;

@Value("${spring.data.mongodb.database}")
private String db;
```

澄清一下，这些属性可以是硬编码的。此外，它们可以使用与预期的 Spring 数据变量不同的名称。**最重要的是，这一次，我们使用的是 URI，而不是单个连接属性，它们不能混合使用。**因此，我们不能在这个应用程序中重用我们的`application.properties`，我们应该[将它移到其他地方](/web/20221004150550/https://www.baeldung.com/properties-with-spring)。

`AbstractMongoClientConfiguration`要求我们覆盖`getDatabaseName()`。这是因为 URI 中不需要数据库名称:

```java
protected String getDatabaseName() {
    return db;
}
```

此时，因为我们使用默认的 Spring 数据变量，我们已经能够连接到我们的数据库。此外，如果数据库不存在，MongoDB 会创建它。让我们来测试一下:

```java
@Test
public void whenClientConfig_thenInsertSucceeds() {
    SpringApplicationBuilder app = new SpringApplicationBuilder(SpringMongoConnectionViaClientApp.class);
    app.web(WebApplicationType.NONE)
      .run(
        "--spring.data.mongodb.uri=mongodb://" + USER + ":" + PASS + "@" + HOST + ":" + PORT + "/" + DB,
        "--spring.data.mongodb.database=" + DB
      );

    assertInsertSucceeds(app.context());
}
```

最后，我们可以覆盖`mongoClient()`来获得优于传统配置的优势。这个方法将使用我们的 URI 变量来构建一个 MongoDB 客户端。这样，我们可以直接引用它。例如，这使我们能够列出我们的连接中所有可用的数据库:

```java
@Override
public MongoClient mongoClient() {
    MongoClient client = MongoClients.create(uri);
    ListDatabasesIterable<Document> databases = client.listDatabases();
    databases.forEach(System.out::println);
    return client;
}
```

如果我们想要完全控制 MongoDB 客户机的创建，这样配置连接是很有用的。

### 4.2.创建自定义`MongoClientFactoryBean`

在下一个例子中，我们将创建一个`MongoClientFactoryBean`。**这一次，我们将创建一个名为`custom.uri`的属性来保存我们的连接配置:**

```java
@SpringBootApplication
public class SpringMongoConnectionViaFactoryApp {

    // main method

    @Bean
    public MongoClientFactoryBean mongo(@Value("${custom.uri}") String uri) {
        MongoClientFactoryBean mongo = new MongoClientFactoryBean();
        ConnectionString conn = new ConnectionString(uri);
        mongo.setConnectionString(conn);

        MongoClient client = mongo.getObject();
        client.listDatabaseNames()
          .forEach(System.out::println);
        return mongo;
    }
}
```

**使用这种方法，我们不需要扩展`AbstractMongoClientConfiguration`。**同样，我们可以控制自己`MongoClient`的创作。例如，通过调用`mongo.setSingleton(false)`，我们每次调用`mongo.getObject()`都会得到一个新的客户端，而不是一个单独的客户端。

### 4.3.使用 MongoClientSettingsBuilderCustomizer 设置连接详细信息

在我们的最后一个例子中，我们将使用一个`MongoClientSettingsBuilderCustomizer`:

```java
@SpringBootApplication
public class SpringMongoConnectionViaBuilderApp {

    // main method

    @Bean
    public MongoClientSettingsBuilderCustomizer customizer(@Value("${custom.uri}") String uri) {
        ConnectionString connection = new ConnectionString(uri);
        return settings -> settings.applyConnectionString(connection);
    }
} 
```

我们使用这个类来定制我们的连接的一部分，但是仍然可以自动配置其余部分。当我们需要以编程方式设置一些属性时，这很有帮助。

## 5.结论

在本文中，我们看到了 Spring Data MongoDB 带来的不同工具。我们用它们以不同的方式建立联系。此外，我们构建了测试用例来保证我们的配置按预期工作。同时，我们看到了配置优先级如何影响我们的连接属性。

和往常一样，源代码可以从 GitHub 上的[处获得。](https://web.archive.org/web/20221004150550/https://github.com/eugenp/tutorials/tree/master/persistence-modules/spring-boot-persistence-mongodb-2)