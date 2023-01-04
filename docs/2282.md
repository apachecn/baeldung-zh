# 用 Spring Boot 记录 MongoDB 查询

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-boot-mongodb-logging>

## 1.概观

使用[Spring Data MongoDB](/web/20221009161527/https://www.baeldung.com/spring-data-mongodb-tutorial)时，我们可能需要登录到比默认级别更高的级别。通常，我们可能需要查看一些附加信息，比如语句执行或查询参数。

在这个简短的教程中，我们将看到如何修改查询的 MongoDB 日志级别。

## 2.配置 MongoDB 查询日志记录

[MongoDB 支持](https://web.archive.org/web/20221009161527/https://docs.spring.io/spring-data/mongodb/docs/3.3.1/reference/html/#introduction)提供了 [`MongoOperations`](https://web.archive.org/web/20221009161527/https://docs.spring.io/spring-data/mongodb/docs/current/api/org/springframework/data/mongodb/core/MongoOperations.html) 接口或者它的主 [`MongoTemplate`](https://web.archive.org/web/20221009161527/https://docs.spring.io/spring-data/mongodb/docs/current/api/org/springframework/data/mongodb/core/MongoTemplate.html) 实现来访问数据，所以我们只需要为`MongoTemplate` 类`.`配置一个调试级别

**像任何 Spring 或 Java 应用程序一样，我们可以使用一个[日志库](/web/20221009161527/https://www.baeldung.com/logback)并为`MongoTemplate`** 定义一个日志级别。

通常，我们可以在配置文件中编写如下内容:

```
<logger name="org.springframework.data.mongodb.core.MongoTemplate" level="DEBUG" />
```

**然而，如果我们正在运行一个 [Spring Boot](/web/20221009161527/https://www.baeldung.com/spring-boot) 应用**，**我们可以在我们的`application.properties`** 文件中进行配置:

```
logging.level.org.springframework.data.mongodb.core.MongoTemplate=DEBUG
```

同样，我们可以使用`YAML`语法:

```
logging:
  level:
    org:
      springframework:
        data:
          mongodb:
            core:
              MongoTemplate: DEBUG 
```

## 3.日志记录的测试类

首先，让我们创建一个`Book`类:

```
@Document(collection = "book")
public class Book {

    @MongoId
    private ObjectId id;
    private String bookName;
    private String authorName;

    // getters and setters
}
```

我们想要创建一个简单的测试类并检查日志。

为了演示这一点，我们使用了[嵌入式 MongoDB](/web/20221009161527/https://www.baeldung.com/spring-boot-embedded-mongodb) 。为了确保安全，让我们先检查一下我们的依赖关系:

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-mongodb</artifactId>
</dependency>
<dependency>
    <groupId>de.flapdoodle.embed</groupId>
    <artifactId>de.flapdoodle.embed.mongo</artifactId>
    <version>${embed.mongo.version}</version>
    <scope>test</scope>
</dependency>
```

最后，让我们使用 [Spring Boot 测试](/web/20221009161527/https://www.baeldung.com/spring-boot-testing)来定义我们的测试类:

```
@SpringBootTest
@TestPropertySource(properties = { "logging.level.org.springframework.data.mongodb.core.MongoTemplate=DEBUG" })
public class LoggingUnitTest {

    private static final String CONNECTION_STRING = "mongodb://%s:%d";

    private MongodExecutable mongodExecutable;
    private MongoTemplate mongoTemplate;

    @AfterEach
    void clean() {
        mongodExecutable.stop();
    }

    @BeforeEach
    void setup() throws Exception {
        String ip = "localhost";
        int port = 27017;

        ImmutableMongodConfig mongodbConfig = MongodConfig.builder()
          .version(Version.Main.PRODUCTION)
          .net(new Net(ip, port, Network.localhostIsIPv6()))
          .build();

        MongodStarter starter = MongodStarter.getDefaultInstance();
        mongodExecutable = starter.prepare(mongodbConfig);
        mongodExecutable.start();
        mongoTemplate = new MongoTemplate(MongoClients.create(String.format(CONNECTION_STRING, ip, port)), "test");
    }
    // tests
} 
```

## 4.日志样本

在这一节中，我们将定义一些简单的测试用例，并展示相关的日志来测试最常见的场景，比如查找、插入、更新或聚合`Document` s。

### 4.1.插入

首先，让我们从插入单个`Document`开始:

```
Book book = new Book();
book.setBookName("Book");
book.setAuthorName("Author");

mongoTemplate.insert(book); 
```

日志显示了我们正在插入哪个集合。当找到一个`Document`时，id 也被记录:

```
[2022-03-20 17:42:47,093]-[main] DEBUG MongoTemplate - Inserting Document containing fields: [bookName, authorName, _class] in collection: book
...
[2022-03-20 17:42:47,144]-[main] DEBUG MongoTemplate - findOne using query: { "id" : { "$oid" : "623759871ff6275fe96a5ecb"}} fields: Document{{}} for class: class com.baeldung.mongodb.models.Book in collection: book
[2022-03-20 17:42:47,149]-[main] DEBUG MongoTemplate - findOne using query: { "_id" : { "$oid" : "623759871ff6275fe96a5ecb"}} fields: {} in db.collection: test.book 
```

### 4.2.更新

同样，当更新一个`Document`时:

```
Book book = new Book();
book.setBookName("Book");
book.setAuthorName("Author");

mongoTemplate.insert(book);

String authorNameUpdate = "AuthorNameUpdate";

book.setAuthorName(authorNameUpdate);
mongoTemplate.updateFirst(query(where("bookName").is("Book")), update("authorName", authorNameUpdate), Book.class); 
```

我们可以在日志中看到实际更新的`Document`字段:

```
[2022-03-20 17:48:31,759]-[main] DEBUG MongoTemplate - Calling update using query: { "bookName" : "Book"} and update: { "$set" : { "authorName" : "AuthorNameUpdate"}} in collection: book
```

### 4.3.批量插入

让我们添加一个批量插入的示例:

```
Book book = new Book();
book.setBookName("Book");
book.setAuthorName("Author");

Book book1 = new Book();
book1.setBookName("Book1");
book1.setAuthorName("Author1");

mongoTemplate.insert(Arrays.asList(book, book1), Book.class);
```

我们可以在日志中看到插入的`Document`的数量:

```
[2022-03-20 17:52:00,564]-[main] DEBUG MongoTemplate - Inserting list of Documents containing 2 items
```

### 4.4.去除

另外，让我们添加一个删除示例:

```
Book book = new Book();
book.setBookName("Book");
book.setAuthorName("Author");

mongoTemplate.insert(book);

mongoTemplate.remove(book); 
```

我们可以在日志中看到，在本例中，被删除的`Document`的 id:

```
[2022-03-20 17:56:42,151]-[main] DEBUG MongoTemplate - Remove using query: { "_id" : { "$oid" : "62375cca2a2cba4db774d8c1"}} in collection: book.
```

### 4.5.聚合

让我们看一个 [`Aggregation`](/web/20221009161527/https://www.baeldung.com/spring-data-mongodb-projections-aggregations) 的例子。在这种情况下，我们需要定义一个结果类。例如，我们将按作者姓名进行聚合:

```
public class GroupByAuthor {

    @Id
    private String authorName;
    private int authCount;

    // getters and setters
}
```

接下来，让我们为分组定义一个测试用例:

```
Book book = new Book();
book.setBookName("Book");
book.setAuthorName("Author");

Book book1 = new Book();
book1.setBookName("Book1");
book1.setAuthorName("Author");

Book book2 = new Book();
book2.setBookName("Book2");
book2.setAuthorName("Author");

mongoTemplate.insert(Arrays.asList(book, book1, book2), Book.class);

GroupOperation groupByAuthor = group("authorName")
  .count()
  .as("authCount");

Aggregation aggregation = newAggregation(groupByAuthor);

AggregationResults<GroupByAuthor> aggregationResults = mongoTemplate.aggregate(aggregation, "book", GroupByAuthor.class);
```

我们可以在日志中看到我们聚合了哪个字段以及聚合管道的类型:

```
[2022-03-20 17:58:51,237]-[main] DEBUG MongoTemplate - Executing aggregation: [{ "$group" : { "_id" : "$authorName", "authCount" : { "$sum" : 1}}}] in collection book
```

## 5.结论

在本文中，我们研究了如何为 Spring 数据 MongoDB 启用调试日志记录级别。

我们已经定义了一些常见的查询场景，并在进行一些现场测试时查看了它们的相关日志。

和往常一样，这些例子的代码可以在 GitHub 的[上找到。](https://web.archive.org/web/20221009161527/https://github.com/eugenp/tutorials/tree/master/persistence-modules/spring-boot-persistence-mongodb-2)