# 嵌入式 MongoDB 的 Spring Boot 集成测试

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-boot-embedded-mongodb>

## 1。概述

在本教程中，我们将学习如何使用 Flapdoodle 的嵌入式 MongoDB 解决方案以及 Spring Boot 来顺利运行 MongoDB 集成测试。

**MongoDB 是一个流行的 NoSQL 文档数据库**。由于高可扩展性、内置分片和出色的社区支持，它经常被许多开发人员视为“NoSQL 存储”。

与任何其他持久性技术一样，**能够轻松测试数据库与应用程序其余部分的集成至关重要**。谢天谢地，Spring Boot 允许我们轻松地编写这种测试。

## 2。Maven 依赖关系

首先，让我们为引导项目设置 Maven 父项目。

感谢父**，我们不需要为每个 Maven 依赖项手动定义版本**。

我们自然会使用 Spring Boot:

```java
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.6.1</version>
    <relativePath /> <!-- lookup parent from repository -->
</parent>
```

你可以在这里找到最新的引导版本[。](https://web.archive.org/web/20220617075734/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22org.springframework.boot%22%20AND%20a%3A%22spring-boot-starter-parent%22)

因为我们添加了 Spring Boot 父项，所以我们可以添加所需的依赖项，而无需指定它们的版本:

```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-mongodb</artifactId>
</dependency>
```

将启用对 MongoDB 的 Spring 支持:

```java
<dependency>
    <groupId>de.flapdoodle.embed</groupId>
    <artifactId>de.flapdoodle.embed.mongo</artifactId>
    <scope>test</scope>
</dependency>
```

`de.flapdoodle.embed.mongo`为集成测试提供嵌入式 MongoDB。

## 3。使用嵌入式 MongoDB 进行测试

本节涵盖了两个场景:Spring Boot 测试和手动测试。

### 3.1。Spring Boot 测试

在添加了`de.flapdoodle.embed.mongo`依赖关系**之后，Spring Boot 将在运行测试时自动尝试下载并启动嵌入式 MongoDB** 。

每个版本的包只下载一次，这样后续的测试会运行得更快。

在这个阶段，我们应该能够开始并通过示例 JUnit 5 集成测试:

```java
@DataMongoTest
@ExtendWith(SpringExtension.class)
public class MongoDbSpringIntegrationTest {
    @DisplayName("given object to save"
        + " when save object using MongoDB template"
        + " then object is saved")
    @Test
    public void test(@Autowired MongoTemplate mongoTemplate) {
        // given
        DBObject objectToSave = BasicDBObjectBuilder.start()
            .add("key", "value")
            .get();

        // when
        mongoTemplate.save(objectToSave, "collection");

        // then
        assertThat(mongoTemplate.findAll(DBObject.class, "collection")).extracting("key")
            .containsOnly("value");
    }
}
```

正如我们所看到的，嵌入式数据库是由 Spring 自动启动的，这也应该记录在控制台中:

```java
...Starting MongodbExampleApplicationTests on arroyo with PID 10413...
```

### 3.2。手动配置测试

Spring Boot 会自动启动并配置嵌入式数据库，然后为我们注入`MongoTemplate`实例。然而，**有时我们可能需要手动配置嵌入式 Mongo 数据库**(例如，当测试一个特定的 DB 版本时)。

下面的代码片段展示了我们如何手动配置嵌入式 MongoDB 实例。这大致相当于之前的春季测试:

```java
class ManualEmbeddedMongoDbIntegrationTest {
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

        ImmutableMongodConfig mongodConfig = MongodConfig
            .builder()
            .version(Version.Main.PRODUCTION)
            .net(new Net(ip, port, Network.localhostIsIPv6()))
            .build();

        MongodStarter starter = MongodStarter.getDefaultInstance();
        mongodExecutable = starter.prepare(mongodConfig);
        mongodExecutable.start();
        mongoTemplate = new MongoTemplate(MongoClients.create(String.format(CONNECTION_STRING, ip, port)), "test");
    }

    @DisplayName("given object to save"
        + " when save object using MongoDB template"
        + " then object is saved")
    @Test
    void test() throws Exception {
        // given
        DBObject objectToSave = BasicDBObjectBuilder.start()
            .add("key", "value")
            .get();

        // when
        mongoTemplate.save(objectToSave, "collection");

        // then
        assertThat(mongoTemplate.findAll(DBObject.class, "collection")).extracting("key")
            .containsOnly("value");
    }
}
```

请注意，我们可以快速创建配置为使用我们手动配置的嵌入式数据库的`MongoTemplate` bean，并在 Spring 容器中注册它，例如，只需创建一个带有将返回`new MongoTemplate(MongoClients.create(connectionString, “test”)`的`@Bean`方法的`@TestConfiguration`。

更多例子可以在官方 Flapdoodle 的 [GitHub 库](https://web.archive.org/web/20220617075734/https://github.com/flapdoodle-oss/de.flapdoodle.embed.mongo)上找到。

### 3.3.记录

我们可以在运行集成测试时为 MongoDB 配置日志消息，方法是将这两个属性添加到`src/test/resources/application.propertes`文件中:

```java
logging.level.org.springframework.boot.autoconfigure.mongo.embedded
logging.level.org.mongodb
```

例如，要禁用日志记录，我们只需将值设置为`off`:

```java
logging.level.org.springframework.boot.autoconfigure.mongo.embedded=off
logging.level.org.mongodb=off
```

### 3.4.在生产中使用真实的数据库

由于我们使用`<scope>test</scope>` **添加了`de.flapdoodle.embed.mongo`依赖项，因此在生产**上运行时无需禁用嵌入式数据库。我们所要做的就是指定 MongoDB 连接细节(例如，主机和端口),这样就万事俱备了。

为了在测试之外使用嵌入式 DB，我们可以使用 Spring 概要文件，它将根据活动概要文件注册正确的`MongoClient`(嵌入式或生产)。

我们还需要将生产依赖的范围更改为`<scope>runtime</scope>`。

## 4.嵌入式测试争议

使用嵌入式数据库在一开始看起来可能是个好主意。事实上，当我们想要测试我们的应用程序是否在以下方面正常运行时，这是一个很好的方法:

*   对象文档映射配置
*   定制持久性生命周期事件监听器(参见`AbstractMongoEventListener`)
*   任何直接使用持久层的代码的逻辑

不幸的是，**使用嵌入式服务器不能被认为是“完全集成测试”**。Flapdoodle 的嵌入式 MongoDB 不是官方的 MongoDB 产品。因此，我们不能确定它在生产环境中的行为是否完全相同。

如果我们希望在尽可能接近产品的环境中运行通信测试，更好的解决方案是使用 Docker 之类的环境容器。

要了解更多关于 Docker 的信息，请阅读我们之前的文章[这里](/web/20220617075734/https://www.baeldung.com/dockerizing-spring-boot-application)。

## 5。结论

Spring Boot 使得验证文档映射和数据库集成的测试变得非常简单。通过添加正确的 Maven 依赖项，我们可以立即在 Spring Boot 集成测试中使用 MongoDB 组件。

我们需要记住**嵌入式 MongoDB 服务器不能被认为是“真正的”服务器**的替代品。

GitHub 上的[提供了所有示例的完整源代码。](https://web.archive.org/web/20220617075734/https://github.com/eugenp/tutorials/tree/master/persistence-modules/spring-boot-persistence-mongodb)