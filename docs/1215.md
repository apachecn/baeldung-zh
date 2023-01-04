# 使用 Spring Data MongoDB 连接到多个数据库

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/mongodb-multiple-databases-spring-data>

## 1.概观

使用 Spring Data MongoDB，我们可以创建一个`MongoClient`来对数据库进行操作。然而，有时，我们可能需要在应用程序中使用多个数据库。

在本教程中，**我们将创建到 MongoDB 的多个连接。我们还将添加一些 Spring Boot 测试来模拟这个场景。**

## 2.Spring Data MongoDB 的多数据库应用

当使用 [MongoDB](/web/20221029180113/https://www.baeldung.com/spring-data-mongodb-tutorial) 时，我们创建一个`MongoTemplate`来访问数据。因此，我们可以创建多个模板来连接各种数据库。

**然而，我们能得到** [`NoUniqueBeanDefinitionException`](https://web.archive.org/web/20221029180113/http://baeldung.com/spring-nosuchbeandefinitionexception#cause-2.) **是因为春天没有找到一颗独特的豆子。**

记住这一点，让我们看看如何构建 Spring Boot 配置。

### 2.1.相关性设置

首先，我们需要一个[弹簧启动程序](https://web.archive.org/web/20221029180113/https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter) `:`

```
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.6.4</version>
    <relativePath />
</parent>
```

然后，我们需要一个 [web starter](https://web.archive.org/web/20221029180113/https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter-web) 和 [data MongoDB](https://web.archive.org/web/20221029180113/https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter-data-mongodb) 的依赖关系:

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-mongodb</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

类似地，如果我们使用 Gradle，我们添加到`build.gradle`:

```
plugins {
    id 'org.springframework.boot' version '2.6.4'
}
```

```
compile 'org.springframework.boot:spring-boot-starter-data-mongodb'
compile 'org.springframework.boot:spring-boot-starter-web'
```

作为替代，我们可以使用 [Spring 初始化器](https://web.archive.org/web/20221029180113/https://start.spring.io/)。

### 2.2.模型

首先，让我们添加我们的模型。我们将创建两个文档，它们将被两个不同的数据库使用。

例如，我们将创建一个`User`文档:

```
@Document(collection = "user")
public class User {

    @MongoId
    private ObjectId id;

    private String name;

    private String surname;
    private String email;

    private int age;

    // getters and setters
}
```

然后，我们再添加一个`Account`文档:

```
@Document(collection = "account")
public class Account {

    @MongoId
    private ObjectId id;

    private String userEmail;

    private String nickName;

    private String accountDomain;

    private String password;

    // getters and setters
} 
```

### 2.3.贮藏室ˌ仓库

然后，我们用一些 Spring 数据方法为每个模型类创建一个存储库。

首先，我们来加一个`UserRepository`:

```
@Repository
public interface UserRepository extends MongoRepository<User, String> {

    User findByEmail(String email);
}
```

下面，我们添加一个`AccountRepository`:

```
@Repository
public interface AccountRepository extends MongoRepository<Account, String> {

    Account findByAccountDomain(String account);
}
```

### 2.4.连接属性

让我们为正在使用的多个数据库定义属性:

```
mongodb.primary.host=localhost
mongodb.primary.database=db1
mongodb.primary.authenticationDatabase=admin
mongodb.primary.username=user1
mongodb.primary.password=password
mongodb.primary.port=27017

mongodb.secondary.host=localhost
mongodb.secondary.database=db2
mongodb.secondary.authenticationDatabase=admin
mongodb.secondary.username=user2
mongodb.secondary.password=password
mongodb.secondary.port=27017
```

值得注意的是，我们有一个用于身份验证的特定数据库的属性。

### 2.5.主要配置

现在，我们需要我们的配置。我们将为每个数据库创建一个。

让我们来看看用于`UserRepository`的主类定义。：

```
@Configuration
@EnableMongoRepositories(basePackageClasses = UserRepository.class, mongoTemplateRef = "primaryMongoTemplate")
@EnableConfigurationProperties
public class PrimaryConfig {
    // beans
}
```

为了演示，让我们分解所有的 beans 和注释。

首先，我们将使用`[MongoProperties](https://web.archive.org/web/20221029180113/https://docs.spring.io/spring-boot/docs/current/api/org/springframework/boot/autoconfigure/mongo/MongoProperties.html).`检索和设置属性。这样，我们将所有属性直接映射到一个 bean:

```
@Bean(name = "primaryProperties")
@ConfigurationProperties(prefix = "mongodb.primary")
@Primary
public MongoProperties primaryProperties() {
    return new MongoProperties();
} 
```

为了向多个用户授予访问权限，我们使用一个 MongoDB [认证机制](https://web.archive.org/web/20221029180113/https://www.mongodb.com/docs/drivers/java/sync/v4.6/fundamentals/auth/)和`[MongoCredential](https://web.archive.org/web/20221029180113/https://mongodb.github.io/mongo-java-driver/3.6/javadoc/com/mongodb/MongoCredential.html).`，我们通过添加一个认证数据库`admin`来构造我们的凭证对象，在本例中:

```
@Bean(name = "primaryMongoClient")
public MongoClient mongoClient(@Qualifier("primaryProperties") MongoProperties mongoProperties) {

    MongoCredential credential = MongoCredential
      .createCredential(mongoProperties.getUsername(), mongoProperties.getAuthenticationDatabase(), mongoProperties.getPassword());

    return MongoClients.create(MongoClientSettings.builder()
      .applyToClusterSettings(builder -> builder
        .hosts(singletonList(new ServerAddress(mongoProperties.getHost(), mongoProperties.getPort()))))
      .credential(credential)
      .build());
} 
```

**根据最新版本的建议，我们使用** `[SimpleMongoClientDatabaseFactory](https://web.archive.org/web/20221029180113/https://docs.spring.io/spring-data/data-mongo/docs/4.0.x/reference/html/#mongo.mongo-db-factory)` **，而不是从连接字符串**创建`MongoTemplate`

```
@Primary
@Bean(name = "primaryMongoDBFactory")
public MongoDatabaseFactory mongoDatabaseFactory(
  @Qualifier("primaryMongoClient") MongoClient mongoClient, 
  @Qualifier("primaryProperties") MongoProperties mongoProperties) {
    return new SimpleMongoClientDatabaseFactory(mongoClient, mongoProperties.getDatabase());
} 
```

**我们需要我们的 beans 在这里`@Primary`，因为我们将添加更多的数据库配置**。否则，我们将陷入我们之前讨论过的唯一性约束。

**当我们映射[多个`EntityManager`](/web/20221029180113/https://www.baeldung.com/spring-data-jpa-multiple-databases) 时，我们用 JPA 做同样的事情。同样，我们需要引用我们的`@EnableMongoRepositories:`** 中的`Mongotemplate`

```
@EnableMongoRepositories(basePackageClasses = UserRepository.class, mongoTemplateRef = "primaryMongoTemplate")
```

### 2.6.二级配置

最后，为了仔细检查，让我们看一下第二个数据库配置:

```
@Configuration
@EnableMongoRepositories(basePackageClasses = AccountRepository.class, mongoTemplateRef = "secondaryMongoTemplate")
@EnableConfigurationProperties
public class SecondaryConfig {

    @Bean(name = "secondaryProperties")
    @ConfigurationProperties(prefix = "mongodb.secondary")
    public MongoProperties secondaryProperties() {
        return new MongoProperties();
    }

    @Bean(name = "secondaryMongoClient")
    public MongoClient mongoClient(@Qualifier("secondaryProperties") MongoProperties mongoProperties) {

        MongoCredential credential = MongoCredential
          .createCredential(mongoProperties.getUsername(), mongoProperties.getAuthenticationDatabase(), mongoProperties.getPassword());

        return MongoClients.create(MongoClientSettings.builder()
          .applyToClusterSettings(builder -> builder
            .hosts(singletonList(new ServerAddress(mongoProperties.getHost(), mongodProperties.getPort()))))
          .credential(credential)
          .build());
    }

    @Bean(name = "secondaryMongoDBFactory")
    public MongoDatabaseFactory mongoDatabaseFactory(
      @Qualifier("secondaryMongoClient") MongoClient mongoClient, 
      @Qualifier("secondaryProperties") MongoProperties mongoProperties) {
        return new SimpleMongoClientDatabaseFactory(mongoClient, mongoProperties.getDatabase());
    }

    @Bean(name = "secondaryMongoTemplate")
    public MongoTemplate mongoTemplate(@Qualifier("secondaryMongoDBFactory") MongoDatabaseFactory mongoDatabaseFactory) {
        return new MongoTemplate(mongoDatabaseFactory);
    }
}
```

在这种情况下，它将引用`AccountRepository`或者属于同一个包`.`的所有类

## 3.试验

我们将针对 MongoDB 实例测试应用程序。我们可以将 [MongoDB 与 Docker](https://web.archive.org/web/20221029180113/https://www.mongodb.com/compatibility/docker) 一起使用。

### 3.1.启动一个 MongoDB 容器

让我们使用 [Docker Compose](/web/20221029180113/https://www.baeldung.com/ops/docker-compose) 运行一个 MongoDB 容器。让我们看看我们的 YAML 模板:

```
services:
  mongo:
    hostname: localhost
    container_name: 'mongo'
    image: 'mongo:latest'
    expose:
      - 27017
    ports:
      - 27017:27017
    environment:
      - MONGO_INITDB_DATABASE=admin
      - MONGO_INITDB_ROOT_USERNAME=admin
      - MONGO_INITDB_ROOT_PASSWORD=admin
    volumes:
      - ./mongo-init.js:/docker-entrypoint-initdb.d/mongo-init.js
```

如果我们想进行身份验证，我们需要用 root 用户进行初始化。为了用更多的用户填充数据库，我们将一个[绑定挂载](https://web.archive.org/web/20221029180113/https://docs.docker.com/storage/bind-mounts/)添加到一个 JavaScript 初始化文件:

```
db.createUser(
    {
        user: "user1",
        pwd: "password",
        roles: [ { role: "readWrite", db: "db1" } ]
    }
)

db.createUser(
    {
        user: "user2",
        pwd: "password",
        roles: [ { role: "readWrite", db: "db2" } ]
    }
)
```

让我们运行我们的容器:

```
docker-compose up -d
```

当容器启动时，它为`mongo-init.js`文件创建一个卷，并将其复制到容器的入口点。

### 3.2.Spring Boot 测试

让我们用一些基本的 [Spring Boot 测试](/web/20221029180113/https://www.baeldung.com/spring-boot-testing)来总结一下:

```
@SpringBootTest(classes = { SpringBootMultipeDbApplication.class })
@TestPropertySource("/multipledb/multidb.properties")
public class MultipleDbUnitTest {

    // set up

    @Test
    void whenFindUserByEmail_thenNameOk() {
        assertEquals("name", userRepository.findByEmail("[[email protected]](/web/20221029180113/https://www.baeldung.com/cdn-cgi/l/email-protection).com")
          .getName());
    }

    @Test
    void whenFindAccountByDomain_thenNickNameOk() {
        assertEquals("nickname", accountRepository.findByAccountDomain("[[email protected]](/web/20221029180113/https://www.baeldung.com/cdn-cgi/l/email-protection)")
          .getNickName());
    }
}
```

首先，我们希望建立一个数据库连接并获得身份验证。如果我们不这样做，我们可能没有填充数据库或者没有启动 MongoDb 实例。

在这些情况下，我们可以查看数据库容器的日志，例如:

```
docker logs 30725c8635d4
```

值得注意的是，最初的 JavaScript 只在容器第一次运行时执行。因此，如果我们需要使用不同的脚本运行，我们可能需要删除该卷:

```
docker-compose down -v
```

## 4.结论

在本文中，我们看到了如何使用 Spring Data MongoDB 创建多个连接。我们看到了如何添加凭证以获得认证。最重要的是，我们已经看到了如何创建配置，其中一个配置必须使用主 beans。最后，我们使用 Docker 和 Spring Boot 测试针对一个正在运行的 MongoDB 实例添加了一些测试用例，

和往常一样，这篇文章的全部代码可以在 GitHub 上找到[。](https://web.archive.org/web/20221029180113/https://github.com/eugenp/tutorials/tree/master/persistence-modules/spring-boot-persistence-mongodb-2)