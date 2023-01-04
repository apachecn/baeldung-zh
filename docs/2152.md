# 具有各种数据库配置的网飞·阿歇斯

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/netflix-archaius-database-configurations>

## 1.概观

网飞 Archaius 提供了连接许多数据源的库和功能。

在本教程中，我们将学习如何获得配置 **:**

*   **使用 JDBC API 连接数据库**
*   **来自 DynamoDB 实例中存储的配置**
*   **通过将 Zookeeper 配置为动态分布式配置**

关于网飞考古的介绍，请看这篇文章。

## 2.利用网飞·阿歇斯和 JDBC 的关系

**正如我们在介绍性教程中解释的，每当我们希望 Archaius 处理配置时，我们都需要创建一个 Apache 的`AbstractConfiguration` bean。**

Spring Cloud Bridge 将自动捕获 bean，并将其添加到 Archaius 的复合配置堆栈中。

### 2.1.属国

使用 JDBC 连接到数据库所需的所有功能都包含在核心库中，因此除了我们在介绍性教程中提到的那些之外，我们不需要任何额外的依赖:

```
<dependencies>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-netflix-archaius</artifactId>
    </dependency>
</dependencies>

<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-netflix</artifactId>
            <version>2.0.1.RELEASE</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
```

我们可以检查 Maven Central 来验证我们使用的是最新版本的 [starter 库](https://web.archive.org/web/20220626193424/https://search.maven.org/search?q=spring-cloud-starter-netflix-archaius)。

### 2.2.如何创建配置 Bean

**在这种情况下，我们需要使用`JDBCConfigurationSource`实例创建`AbstractConfiguration` bean。**

为了说明如何从 JDBC 数据库中获取值，我们必须指定:

*   一个`javax.sql.Datasource`物体
*   一个 SQL 查询字符串，它将检索至少两个包含配置键及其相应值的列
*   分别指示属性键和值的两列

让我们继续创建这个 bean:

```
@Autowired
DataSource dataSource;

@Bean
public AbstractConfiguration addApplicationPropertiesSource() {
    PolledConfigurationSource source =
      new JDBCConfigurationSource(dataSource,
        "select distinct key, value from properties",
        "key",
        "value");
    return new DynamicConfiguration(source, new FixedDelayPollingScheduler());
}
```

### 2.3.尝试一下

为了保持简单并且仍然有一个可操作的例子，我们将用一些初始数据建立一个 H2 内存数据库实例。

为此，我们将首先添加必要的依赖项:

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
    <version>2.0.5.RELEASE</version>
</dependency>
<dependency>
    <groupId>com.h2database</groupId>
    <artifactId>h2</artifactId>
    <version>1.4.197</version>
    <scope>runtime</scope>
</dependency>
```

注意:我们可以在 Maven Central 中查看最新版本的`[h2](https://web.archive.org/web/20220626193424/https://search.maven.org/search?q=g:com.h2database%20a:h2)`和`[spring-boot-starter-data-jpa](https://web.archive.org/web/20220626193424/https://search.maven.org/search?q=a:spring-boot-starter-data-jpa%20g:org.springframework.boot)`库。

接下来，我们将声明包含我们属性的 JPA 实体:

```
@Entity
public class Properties {
    @Id
    private String key;
    private String value;
}
```

我们将在资源中包含一个 `data.sql`文件，用一些初始值填充内存数据库:

```
insert into properties
values('baeldung.archaius.properties.one', 'one FROM:jdbc_source');
```

最后，为了检查任意给定点的属性值，我们可以创建一个端点来检索由 Archaius 管理的值:

```
@RestController
public class ConfigPropertiesController {

    private DynamicStringProperty propertyOneWithDynamic = DynamicPropertyFactory
      .getInstance()
      .getStringProperty("baeldung.archaius.properties.one", "not found!");

    @GetMapping("/properties-from-dynamic")
    public Map<String, String> getPropertiesFromDynamic() {
        Map<String, String> properties = new HashMap<>();
        properties.put(propertyOneWithDynamic.getName(), propertyOneWithDynamic.get());
        return properties;
    }
}
```

如果数据在任何时候发生变化，Archaius 都会在运行时检测到，并开始检索新值。

当然，这个端点也可以用在接下来的例子中。

## 3.如何使用 DynamoDB 实例创建配置源

正如我们在上一节中所做的，我们将创建一个全功能的项目来正确分析 Archaius 如何使用 DynamoDB 实例作为配置源来管理属性。

### 3.1.属国

让我们将以下库添加到我们的`pom.xml`文件中:

```
<dependency>
    <groupId>com.amazonaws</groupId>
    <artifactId>aws-java-sdk-dynamodb</artifactId>
    <version>1.11.414</version>
</dependency>
<dependency>
    <groupId>com.github.derjust</groupId>
    <artifactId>spring-data-dynamodb</artifactId>
    <version>5.0.3</version>
</dependency>
<dependency>
    <groupId>com.netflix.archaius</groupId>
    <artifactId>archaius-aws</artifactId>
    <version>0.7.6</version>
</dependency>
```

我们可以在 Maven Central 上查看最新的依赖版本，但是对于`archaius-aws`版本，我们建议坚持使用由[Spring Cloud 网飞库](https://web.archive.org/web/20220626193424/https://github.com/spring-cloud/spring-cloud-netflix/blob/master/spring-cloud-netflix-dependencies/pom.xml)支持的版本。

[`aws-java-sdk-dynamodb`](https://web.archive.org/web/20220626193424/https://search.maven.org/search?q=a:aws-java-sdk-dynamodb) 依赖项将允许我们设置 DynamoDB 客户端来连接数据库。

使用 [`spring-data-dynamodb`](https://web.archive.org/web/20220626193424/https://search.maven.org/search?q=a:spring-data-dynamodb%20g:com.github.derjust) 库，我们将建立 DynamoDB 库。

最后，我们将使用 [`archaius-aws`](https://web.archive.org/web/20220626193424/https://search.maven.org/search?q=a:archaius-aws) 库来创建`AbstractConfiguration`。

### 3.2.使用 DynamoDB 作为配置源

这一次，将使用一个`DynamoDbConfigurationSource`对象:来创建

```
@Autowired
AmazonDynamoDB amazonDynamoDb;

@Bean
public AbstractConfiguration addApplicationPropertiesSource() {
    PolledConfigurationSource source = new DynamoDbConfigurationSource(amazonDynamoDb);
    return new DynamicConfiguration(
      source, new FixedDelayPollingScheduler());
}
```

**默认情况下，Archaius 在 Dynamo 数据库中搜索一个名为“archaiusProperties”的表，其中包含一个“key”和一个“value”属性，用作源。**

如果我们想覆盖这些值，我们必须声明以下系统属性:

*   `com.netflix.config.dynamo.tableName`
*   `com.netflix.config.dynamo.keyAttributeName`
*   `com.netflix.config.dynamo.valueAttributeName`

### 3.3.创建一个功能齐全的示例

正如我们在本 DynamoDB 指南中所做的，我们将从安装一个本地 DynamoDB 实例开始，以便轻松测试功能。

我们还将按照指南的指示创建我们之前“自动连接”的`AmazonDynamoDB`实例。

为了用一些初始数据填充数据库，我们将首先创建一个`DynamoDBTable`实体来映射数据:

```
@DynamoDBTable(tableName = "archaiusProperties")
public class ArchaiusProperties {

    @DynamoDBHashKey
    @DynamoDBAttribute
    private String key;

    @DynamoDBAttribute
    private String value;

    // ...getters and setters...
}
```

接下来，我们将为这个实体创建一个`CrudRepository`:

```
public interface ArchaiusPropertiesRepository extends CrudRepository<ArchaiusProperties, String> {}
```

最后，我们将使用存储库和`AmazonDynamoDB` 实例来创建表，然后插入数据:

```
@Autowired
private ArchaiusPropertiesRepository repository;

@Autowired
AmazonDynamoDB amazonDynamoDb;

private void initDatabase() {
    DynamoDBMapper mapper = new DynamoDBMapper(amazonDynamoDb);
    CreateTableRequest tableRequest = mapper
      .generateCreateTableRequest(ArchaiusProperties.class);
    tableRequest.setProvisionedThroughput(new ProvisionedThroughput(1L, 1L));
    TableUtils.createTableIfNotExists(amazonDynamoDb, tableRequest);

    ArchaiusProperties property = new ArchaiusProperties("baeldung.archaius.properties.one", "one FROM:dynamoDB");
    repository.save(property);
}
```

我们可以在创建`DynamoDbConfigurationSource`之前调用这个方法。

我们现在已经准备好运行应用程序了。

## 4.如何设置动态 Zookeeper 分布式配置

**正如我们之前在介绍 Zookeeper 的文章中看到的那样，这个工具的好处之一是可以将它用作分布式配置存储。**

如果我们将它与 Archaius 结合起来，我们最终会得到一个灵活的、可扩展的配置管理解决方案。

### 4.1.属国

让我们按照[官方 Spring Cloud 的说明](https://web.archive.org/web/20220626193424/https://cloud.spring.io/spring-cloud-zookeeper/single/spring-cloud-zookeeper.html#spring-cloud-zookeeper-install)来设置更稳定版本的 Apache 的 Zookeeper。

唯一的区别是，我们只需要 Zookeeper 提供的部分功能，因此我们可以使用`spring-cloud-starter-zookeeper-config`依赖项，而不是官方指南中使用的那个:

```
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-zookeeper-config</artifactId>
    <version>2.0.0.RELEASE</version>
    <exclusions>
        <exclusion>
            <groupId>org.apache.zookeeper</groupId>
            <artifactId>zookeeper</artifactId>
        </exclusion>
    </exclusions>
</dependency>
<dependency>
    <groupId>org.apache.zookeeper</groupId>
    <artifactId>zookeeper</artifactId>
    <version>3.4.13</version>
    <exclusions>
        <exclusion>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-log4j12</artifactId>
        </exclusion>
    </exclusions>
</dependency>
```

再次，我们可以在 Maven Central 查看[`spring-cloud-starter-zookeeper-config`](https://web.archive.org/web/20220626193424/https://search.maven.org/search?q=a:spring-cloud-starter-zookeeper-config) [`zookeeper`](https://web.archive.org/web/20220626193424/https://search.maven.org/search?q=g:org.apache.zookeeper%20a:zookeeper)的最新版本依赖。

请确保避免使用`zookeeper`测试版。

### 4.2.春云的自动配置

**正如在官方文档中解释的那样，包括`spring-cloud-starter-zookeeper-config`依赖项足以设置 Zookeeper 属性源。**

默认情况下，只有一个源是自动配置的，在`config/application` Zookeeper 节点下搜索属性。因此，该节点用作不同应用程序之间的共享配置源。

此外，如果我们使用`spring.application.name`属性指定一个应用程序名，另一个源会自动配置，这次是在`config/<app_name>`节点中搜索属性。

**这些父节点下的每个节点名都会表示一个属性键，它们的数据就是属性值。**

幸运的是，由于 Spring Cloud 将这些属性源添加到上下文中，Archaius 会自动管理它们。不需要以编程方式创建 AbstractConfiguration。

### 4.3.准备初始数据

在这种情况下，我们还需要一个本地 Zookeeper 服务器来存储节点配置。我们可以按照这个 Apache 的指南来设置一个运行在端口 2181 上的独立服务器。

为了连接到 Zookeeper 服务并创建一些初始数据，我们将使用 [Apache 的馆长客户端](/web/20220626193424/https://www.baeldung.com/apache-curator):

```
@Component
public class ZookeeperConfigsInitializer {

    @Autowired
    CuratorFramework client;

    @EventListener
    public void appReady(ApplicationReadyEvent event) throws Exception {
        createBaseNodes();
        if (client.checkExists().forPath("/config/application/baeldung.archaius.properties.one") == null) {
            client.create()
              .forPath("/config/application/baeldung.archaius.properties.one",
              "one FROM:zookeeper".getBytes());
        } else {
            client.setData()
              .forPath("/config/application/baeldung.archaius.properties.one",
              "one FROM:zookeeper".getBytes());
        }
    }

    private void createBaseNodes() throws Exception {
        if (client.checkExists().forPath("/config") == null) {
            client.create().forPath("/config");
        }
        if (client.checkExists().forPath("/config/application") == null) {
            client.create().forPath("/config/application");
        }
    }
}
```

我们可以检查日志来查看属性源，以验证网飞·阿歇斯在属性更改后是否刷新了它们。

## 5.结论

在本文中，我们已经了解了如何使用网飞·阿歇斯来设置高级配置源。我们必须考虑到它也支持其他来源，如 Etcd、Typesafe、AWS S3 文件和 JClouds。

和往常一样，我们可以在 Github repo 中查看所有的例子。**