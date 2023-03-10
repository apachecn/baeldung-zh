# 用 Spring Boot 连接到 NoSQL 数据库

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-boot-nosql-database>

## 1。概述

在本教程中，我们将学习如何使用 Sprint Boot 连接到 [NoSQL](/web/20220707143855/https://www.baeldung.com/category/persistence/nosql/) 数据库。对于本文的重点，我们将使用 [DataStax Astra DB](/web/20220707143855/https://www.baeldung.com/datastax-post) ，这是一个由 [Apache Cassandra](https://web.archive.org/web/20220707143855/https://cassandra.apache.org/) 支持的 DBaaS，它允许我们使用云原生服务开发和部署数据驱动的应用程序。

首先，我们将从如何使用 Astra DB 设置和配置我们的应用程序开始。然后我们将学习如何使用 [Spring Boot](/web/20220707143855/https://www.baeldung.com/category/spring/spring-boot/) 构建一个简单的应用程序。

## 2。依赖性

让我们从将依赖项添加到我们的`pom.xml`开始。当然，我们将需要`[spring-boot-starter-data-cassandra](https://web.archive.org/web/20220707143855/https://search.maven.org/search?q=g:org.springframework.boot%20AND%20a:spring-boot-starter-data-cassandra)`依赖关系:

```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-cassandra</artifactId>
    <version>2.6.3</version>
</dependency>
```

接下来，我们将添加 [`spring-boot-starter-web`](https://web.archive.org/web/20220707143855/https://search.maven.org/search?q=g:org.springframework.boot%20AND%20a:spring-boot-starter-web) 依赖项:

```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
     <version>2.6.3</version>
</dependency>
```

最后，我们将使用数据税务 [`astra-spring-boot-starter`](https://web.archive.org/web/20220707143855/https://search.maven.org/search?q=g:com.datastax.astra%20AND%20a:astra-spring-boot-starter) :

```java
<dependency>
    <groupId>com.datastax.astra</groupId>
    <artifactId>astra-spring-boot-starter</artifactId>
    <version>0.3.0</version>
</dependency>
```

既然我们已经配置了所有必需的依赖项，我们就可以开始编写我们的 Spring Boot 应用程序了。

## 3。数据库设置

在我们开始定义我们的应用程序之前，**有必要快速重申一下，DataStax Astra 是一个基于云的数据库产品，由 Apache Cassandra** 提供支持。这给了我们一个完全托管、完全管理的 Cassandra 数据库，我们可以用它来存储我们的数据。然而，正如我们将要看到的，我们设置和连接数据库的方式有一些特殊性。

为了与我们的数据库进行交互，我们需要在主机平台上[设置我们的 Astra 数据库](/web/20220707143855/https://www.baeldung.com/cassandra-astra-stargate-dashboard#how-to-set-up-datastax-astra)。然后，我们需要下载我们的[安全连接包](/web/20220707143855/https://www.baeldung.com/cassandra-astra-rest-dashboard-map#1-download-secure-connect-bundle)，它包含 SSL 证书的细节和这个数据库的连接细节，允许我们安全地连接。

出于本教程的目的，我们假设我们已经完成了这两项任务。

## 4。应用程序配置

接下来，我们将为我们的应用程序配置一个简单的`main`类:

```java
@SpringBootApplication
public class AstraDbSpringApplication {

    public static void main(String[] args) {
        SpringApplication.run(AstraDbSpringApplication.class, args);
    }
}
```

正如我们所见，这是一个普通的 [Spring Boot 应用](/web/20220707143855/https://www.baeldung.com/spring-boot-start)。现在让我们开始填充我们的`application.properties`文件:

```java
astra.api.application-token=<token>
astra.api.database-id=<your_db_id>
astra.api.database-region=europe-west1
```

这些是我们的 Cassandra 凭据，可以直接从 Astra 仪表板上获取。

为了通过标准的`CqlSession`使用 [CQL](/web/20220707143855/https://www.baeldung.com/cassandra-data-types) ，我们将添加另外几个属性，包括我们下载的安全连接包的位置:

```java
astra.cql.enabled=true
astra.cql.downloadScb.path=~/.astra/secure-connect-shopping-list.zip 
```

最后，我们将添加几个标准的 Spring 数据属性来使用 [Cassandra](/web/20220707143855/https://www.baeldung.com/spring-data-cassandra-tutorial) :

```java
spring.data.cassandra.keyspace=shopping_list
spring.data.cassandra.schema-action=CREATE_IF_NOT_EXISTS
```

这里，我们指定了数据库的键空间，并告诉 Spring Data 创建我们的表，如果它们不存在的话。

## 5。测试我们的连接

现在我们已经准备好了测试数据库连接的所有部分。所以让我们继续定义一个简单的 [REST 控制器](/web/20220707143855/https://www.baeldung.com/spring-controller-vs-restcontroller):

```java
@RestController
public class AstraDbApiController {

    @Autowired
    private AstraClient astraClient;

    @GetMapping("/ping")
    public String ping() {
        return astraClient.apiDevopsOrganizations()
          .organizationId();
    }

}
```

正如我们所看到的，我们已经使用`AstraClient`类创建了一个简单的 ping 端点，它将返回我们数据库的组织 id。**这是一个包装类，作为 Astra SDK 的一部分，我们可以用它来与各种 Astra API**交互。

最重要的是，这只是一个简单的测试，以确保我们可以建立连接。所以让我们继续使用 Maven 运行我们的应用程序:

```java
mvn clean install spring-boot:run
```

我们应该在控制台上看到与 Astra 数据库建立的连接:

```java
...
13:08:00.656 [main] INFO  c.d.stargate.sdk.StargateClient - + CqlSession   :[ENABLED]
13:08:00.656 [main] INFO  c.d.stargate.sdk.StargateClient - + API Cql      :[ENABLED]
13:08:00.657 [main] INFO  c.d.stargate.sdk.rest.ApiDataClient - + API Data     :[ENABLED]
13:08:00.657 [main] INFO  c.d.s.sdk.doc.ApiDocumentClient - + API Document :[ENABLED]
13:08:00.658 [main] INFO  c.d.s.sdk.gql.ApiGraphQLClient - + API GraphQL  :[ENABLED]
13:08:00.658 [main] INFO  com.datastax.astra.sdk.AstraClient
  - [AstraClient] has been initialized.
13:08:01.515 [main] INFO  o.b.s.a.AstraDbSpringApplication
  - Started AstraDbSpringApplication in 7.653 seconds (JVM running for 8.097) 
```

同样，如果我们在浏览器中找到我们的端点或者使用`curl,`点击它，我们应该得到一个有效的响应:

```java
$ curl http://localhost:8080/ping; echo
d23bf54d-1bc2-4ab7-9bd9-2c628aa54e85
```

太好了！现在我们已经建立了数据库连接，并且实现了一个使用 Spring Boot 的简单应用程序，让我们看看如何存储和检索数据。

## 6。使用弹簧数据

作为 Cassandra 数据库访问的基础，我们有几种风格可供选择。在本教程中，我们选择使用支持 Cassandra 的 Spring 数据。

Spring Data 的存储库抽象的主要目标是显著减少实现我们的数据访问层所需的样板代码的数量，这将有助于保持我们的示例非常简单。

对于我们的数据模型，**我们将定义一个代表简单购物清单的实体**:

```java
@Table
public class ShoppingList {

    @PrimaryKey
    @CassandraType(type = Name.UUID)
    private UUID uid = UUID.randomUUID();

    private String title;
    private boolean completed = false;

    @Column
    private List<String> items = new ArrayList<>();

    // Standard Getters and Setters
}
```

在本例中，我们在 bean 中使用了几个标准注释来将实体映射到 Cassandra 数据表，并定义一个名为`uid`的主键列。

现在让我们创建要在应用程序中使用的`ShoppingListRepository`:

```java
@Repository
public interface ShoppingListRepository extends CassandraRepository<ShoppingList, String> {

    ShoppingList findByTitleAllIgnoreCase(String title);

}
```

这遵循了标准的 Spring 数据仓库抽象。除了包含在`CassandraRepository`接口中的继承方法，比如`findAll`、**之外，我们还添加了一个额外的方法`findByTitleAllIgnoreCase`，我们可以用它来查找标题中的购物清单。**

事实上，使用 Astra [Spring Boot 启动器](/web/20220707143855/https://www.baeldung.com/spring-boot-starters)的真正好处之一是，它使用先前定义的属性为我们创建`CqlSession` bean。

## 7。将所有这些放在一起

现在我们已经有了数据访问存储库，让我们定义一个简单的服务和控制器:

```java
@Service
public class ShoppingListService {

    @Autowired
    private ShoppingListRepository shoppingListRepository;

    public List<ShoppingList> findAll() {
        return shoppingListRepository.findAll(CassandraPageRequest.first(10)).toList();
    }

    public ShoppingList findByTitle(String title) {
        return shoppingListRepository.findByTitleAllIgnoreCase(title);
    }

    @PostConstruct
    public void insert() {
        ShoppingList groceries = new ShoppingList("Groceries");
        groceries.setItems(Arrays.asList("Bread", "Milk, Apples"));

        ShoppingList pharmacy = new ShoppingList("Pharmacy");
        pharmacy.setCompleted(true);
        pharmacy.setItems(Arrays.asList("Nappies", "Suncream, Aspirin"));

        shoppingListRepository.save(groceries);
        shoppingListRepository.save(pharmacy);
    }

}
```

为了我们的测试应用程序，**我们添加了一个 [`@PostContruct`注释](/web/20220707143855/https://www.baeldung.com/spring-postconstruct-predestroy)来将一些测试数据插入我们的数据库。**

对于谜题的最后一部分，我们将添加一个简单的控制器，其一个端点用于检索我们的购物清单:

```java
@RestController
@RequestMapping(value = "/shopping")
public class ShoppingListController {

    @Autowired
    private ShoppingListService shoppingListService;

    @GetMapping("/list")
    public List<ShoppingList> findAll() {
        return shoppingListService.findAll();
    }
}
```

现在，当我们运行应用程序并访问 http://localhost:8080/shopping/list 时，我们将看到一个包含不同购物列表对象的 JSON 响应:

```java
[
  {
    "uid": "363dba2e-17f3-4d01-a44f-a805f74fc43d",
    "title": "Groceries",
    "completed": false,
    "items": [
      "Bread",
      "Milk, Apples"
    ]
  },
  {
    "uid": "9c0f407e-5fc1-41ad-8e46-b3c115de9474",
    "title": "Pharmacy",
    "completed": true,
    "items": [
      "Nappies",
      "Suncream, Aspirin"
    ]
  }
]
```

这证实了我们的应用程序工作正常。厉害！

## 8。使用 Cassandra 模板

另一方面，也可以直接使用 [Cassandra 模板](/web/20220707143855/https://www.baeldung.com/spring-data-cassandratemplate-cqltemplate)，这是经典的 Spring CQL 方法，可能仍然是最流行的。

简而言之，我们可以很容易地扩展我们的`AstraDbApiController`来检索我们的数据中心:

```java
@Autowired
private CassandraTemplate cassandraTemplate;

@GetMapping("/datacenter")
public String datacenter() {
    return cassandraTemplate
        .getCqlOperations()
        .queryForObject("SELECT data_center FROM system.local", String.class);
}
```

这仍将利用我们已经定义的所有配置属性。**正如我们所见，这两种访问方法之间的切换是完全透明的。**

## 9。结论

在本文中，我们学习了如何设置和连接到托管的 Cassandra Astra 数据库。接下来，我们构建了一个简单的购物清单应用程序，使用 Spring 数据存储和检索数据。最后，我们还讨论了如何使用较低级别的访问方法 Cassandra 模板。

和往常一样，GitHub 上的[提供了这篇文章的完整源代码。](https://web.archive.org/web/20220707143855/https://github.com/Baeldung/datastax-cassandra/tree/main/spring-boot-astra-db)