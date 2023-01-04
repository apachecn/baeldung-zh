# Spring Data Azure Cosmos DB 简介

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-data-cosmos-db>

## 1.概观

在本教程中，我们将了解 Azure Cosmos DB 以及如何使用 Spring 数据与它进行交互。

## 2.天蓝色宇宙数据库

**[Azure Cosmos DB](https://web.archive.org/web/20220625230703/https://github.com/Azure/azure-sdk-for-java/tree/master/sdk/cosmos/azure-cosmos) 是微软的全球分布式数据库服务。**

**这是一个 NoSQL 数据库**，为吞吐量、延迟、可用性和一致性保证提供全面的服务级别协议。此外，它还确保了 [99.999%的读写可用性](https://web.archive.org/web/20220625230703/https://docs.microsoft.com/en-us/azure/cosmos-db/introduction#guaranteed-low-latency-at-99th-percentile-worldwide)。

Azure Cosmos DB 不仅仅提供两种一致性选择，即一致或不一致。相反，**我们得到五个一致性选择:`strong`、`bounded staleness`、`session`、`consistent prefix`和`eventual`。**

我们可以灵活地扩展 Azure Cosmos DB 的吞吐量和存储。

此外，它在所有 Azure 地区都可用，并提供交钥匙全球分发，因为我们只需点击一个按钮，就可以在任何 Azure 地区复制我们的数据。这有助于我们让我们的数据更接近我们的用户，以便我们能够更快地满足他们的请求。

它是模式不可知的 **，因为它没有模式**。此外，我们不需要对 Azure Cosmos Db 进行任何索引管理。它会自动为我们建立数据索引。

我们可以使用不同的标准 API(如 SQL、MongoDB、Cassandra 等)与 Azure CosmosDb 合作。

## 3.Spring Data Azure Cosmos DB

微软还提供了一个模块，允许我们使用 Spring 数据与 Cosmos DB 一起工作。在下一节中，我们将看到如何在 Spring Boot 应用程序中使用 Azure Cosmos DB。

在我们的示例中，我们将创建一个 Spring web 应用程序，它将产品实体存储在 Azure Cosmos 数据库中，并对其执行基本的 CRUD 操作。首先，我们需要按照[文档](https://web.archive.org/web/20220625230703/https://docs.microsoft.com/en-us/azure/cosmos-db/create-cosmosdb-resources-portal)中的说明，在 Azure 门户中配置一个帐户和数据库。

**如果我们不想在 Azure 门户上创建账号，Azure 也提供了 Azure Cosmos 模拟器。**尽管这并不包含 Azure Cosmos 服务的所有功能，并且存在一些差异，但我们可以将其用于本地开发和测试。

我们可以通过两种方式在本地环境中使用模拟器:要么在我们的机器上下载 Azure Cosmos 模拟器，要么在 Docker for Windows 上运行模拟器。

**我们将选择在 Docker** for Windows 上运行它。让我们通过运行以下命令来提取 Docker 映像:

```java
docker pull microsoft/azure-cosmosdb-emulator
```

然后，我们可以运行 Docker 映像，并通过运行以下命令启动容器:

```java
set containerName=azure-cosmosdb-emulator
set hostDirectory=%LOCALAPPDATA%\azure-cosmosdb-emulator.hostd
md %hostDirectory% 2>nul
docker run --name %containerName% --memory 2GB --mount "type=bind,source=%hostDirectory%,destination=C:\CosmosDB.Emulator\bind-mount" -P --interactive --tty microsoft/azure-cosmosdb-emulator
```

一旦我们在 Azure 门户或 Docker 中配置了 Azure Cosmos DB 帐户和数据库，我们可以继续在我们的 Spring Boot 应用程序中配置它。

## 4.在春天使用 Azure Cosmos DB

### 4.1.用 Spring 配置 Spring 数据 Azure Cosmos DB

我们首先在我们的 `pom.xml`中添加 [spring-data-cosmosdb](https://web.archive.org/web/20220625230703/https://mvnrepository.com/artifact/com.microsoft.azure/spring-data-cosmosdb/2.3.0) 依赖项:

```java
<dependency> 
    <groupId>com.microsoft.azure</groupId> 
    <artifactId>spring-data-cosmosdb</artifactId> 
    <version>2.3.0</version> 
</dependency> 
```

**要从我们的 Spring 应用程序访问 Azure Cosmos DB，我们需要数据库的 URI，即[访问键](https://web.archive.org/web/20220625230703/https://docs.microsoft.com/en-us/azure/cosmos-db/secure-access-to-data)和数据库名称。**然后我们在`application.properties`中添加连接属性:

```java
azure.cosmosdb.uri=cosmodb-uri
azure.cosmosdb.key=cosmodb-primary-key
azure.cosmosdb.secondaryKey=cosmodb-secondary-key
azure.cosmosdb.database=cosmodb-name 
```

我们可以从 Azure 门户中找到上述属性的值。URI、主键和辅键将在 Azure 门户的 Azure Cosmos DB 的 keys 部分提供。

要从我们的应用程序连接到 Azure Cosmos DB，我们需要创建一个客户端。为此，**我们需要在我们的配置类中扩展`AbstractCosmosConfiguration`类并添加`@EnableCosmosRepositories `注释。**

该注释将扫描指定包中扩展 Spring 数据存储库接口的接口。

我们还需要**配置一个`CosmosDBConfig`** 类型的 bean:

```java
@Configuration
@EnableCosmosRepositories(basePackages = "com.baeldung.spring.data.cosmosdb.repository")
public class AzureCosmosDbConfiguration extends AbstractCosmosConfiguration {

    @Value("${azure.cosmosdb.uri}")
    private String uri;

    @Value("${azure.cosmosdb.key}")
    private String key;

    @Value("${azure.cosmosdb.database}")
    private String dbName;

    private CosmosKeyCredential cosmosKeyCredential;

    @Bean
    public CosmosDBConfig getConfig() {
        this.cosmosKeyCredential = new CosmosKeyCredential(key);
        CosmosDBConfig cosmosdbConfig = CosmosDBConfig.builder(uri, this.cosmosKeyCredential, dbName)
            .build();
        return cosmosdbConfig;
    }
}
```

### 4.2.为 Azure Cosmos DB 创建实体

为了与 Azure Cosmos DB 交互，我们使用了实体。因此，让我们创建一个将存储在 Azure Cosmos DB 中的实体。为了使我们的`Product`类、**成为一个实体，我们将使用`@Document`注释:**

```java
@Document(collection = "products")
public class Product {

    @Id
    private String productid;

    private String productName;

    private double price;

    @PartitionKey
    private String productCategory;
}
```

在这个例子中，**我们使用了带有值`products`的`collection`属性，来表示这将是我们在数据库**中的容器的名称。如果我们不为`collection`参数提供任何值，那么类名将被用作数据库中的容器名。

我们还为我们的文档定义了一个 id。我们可以在类中创建一个名为`id`的字段，也可以用`@Id` 注释来注释一个字段。这里我们使用了`productid`字段作为文档 id。

**我们可以通过使用分区键对容器中的数据进行逻辑分区，方法是在我们的类中用`@PartitionKey.`** 注释一个字段，我们使用了`productCategory`字段作为分区键。

默认情况下，索引策略是由 Azure 定义的，但是我们也可以通过在实体类上使用`@DocumentIndexingPolicy `注释来定制它。

我们还可以通过创建一个名为`_etag`的字段并用`@Version.`对其进行注释来为我们的实体容器启用乐观锁定

### 4.3.定义存储库

现在**让我们创建一个扩展`CosmosRepository`** 的`ProductRepository`接口。使用这个接口，我们可以在 Azure Cosmos DB 上执行 CRUD 操作:

```java
@Repository
public interface ProductRepository extends CosmosRepository<Product, String> {
    List findByProductName(String productName);

}
```

正如我们所见，这是以类似于其他 Spring 数据模块的方式定义的。

### 4.4.测试连接

现在我们可以创建一个 Junit 测试，使用我们的`ProductRepository`在 Azure Cosmos DB 中保存一个`Product`实体:

```java
@SpringBootTest
public class AzureCosmosDbApplicationManualTest {

    @Autowired
    ProductRepository productRepository;

    @Test
    public void givenProductIsCreated_whenCallFindById_thenProductIsFound() {
        Product product = new Product();
        product.setProductid("1001");
        product.setProductCategory("Shirt");
        product.setPrice(110.0);
        product.setProductName("Blue Shirt");

        productRepository.save(product);
        Product retrievedProduct = productRepository.findById("1001", new PartitionKey("Shirt"))
            .orElse(null);
        Assert.notNull(retrievedProduct, "Retrieved Product is Null");
    }

}
```

通过运行这个 Junit 测试，我们可以从我们的 Spring 应用程序测试我们与 Azure Cosmos DB 的连接。

## 5.结论

在本教程中，我们学习了 Azure Cosmos DB。此外，我们学习了如何从 Spring Boot 应用程序访问 Azure Cosmos DB，如何通过扩展`CosmosRepository`来创建实体和配置存储库以与之交互。

上面例子的代码可以在 GitHub 的[上找到。](https://web.archive.org/web/20220625230703/https://github.com/eugenp/tutorials/tree/master/persistence-modules/spring-data-cosmosdb)