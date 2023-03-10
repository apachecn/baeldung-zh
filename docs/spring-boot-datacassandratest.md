# 使用 Spring Boot 和@DataCassandraTest 测试 NoSQL 查询

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-boot-datacassandratest>

## 1。概述

通常，我们使用 Spring 的自动配置系统如*@ Spring boot test*来测试 Spring Boot 应用。但是 **这导致了大量自动配置组件的进口。**

然而，只加载所需的部分来测试应用程序的一部分总是有帮助的。 **为此，Spring Boot 为切片检验提供了许多注解。** 最重要的是，这些 Spring 注释中的每一个都加载了特定层所需的非常有限的一组自动配置的组件。

在本教程中，我们将重点测试 Spring Boot 应用程序的 Cassandra 数据库片，以便 **了解 Spring 提供的*****@ DataCassandraTest*****注释。**

另外，我们将看看一个小型的基于 Cassandra 的 Spring Boot 应用程序。

而且，如果你在生产中运行 Cassandra，你肯定可以省去运行和维护自己的服务器的复杂性，而使用 [Astra 数据库](/web/20220524033958/https://www.baeldung.com/datastax-post)，这是**一个基于 Apache Cassandra 的云数据库。**

## 2。Maven 依赖关系

为了在我们的 Cassandra Spring Boot 应用程序中使用 *@DataCassandraTest* 注释，我们必须添加[*spring-boot-starter-test*](https://web.archive.org/web/20220524033958/https://search.maven.org/artifact/org.springframework.boot/spring-boot-starter-test)依赖项:

```java
<dependency> 
    <groupId>org.springframework.boot</groupId> 
    <artifactId>spring-boot-starter-test</artifactId>
    <version>2.5.3</version>
    <scope>test</scope>
</dependency>
```

Spring 的[*Spring-boot-test-auto configure*](https://web.archive.org/web/20220524033958/https://search.maven.org/search?q=a:spring-boot-test-autoconfigure)是[*Spring-boot-starter-test*](https://web.archive.org/web/20220524033958/https://search.maven.org/artifact/org.springframework.boot/spring-boot-starter-test)库的一部分，它包含了许多用于测试应用程序不同部分的自动配置组件。

**一般这种测试标注有** ***@XXXTest*** **的模式。**

*@ DataCassandraTest*注释导入以下 Spring 数据自动配置组件:

*   `CacheAutoConfiguration`
*   `CassandraAutoConfiguration`
*   `CassandraDataAutoConfiguration`
*   `CassandraReactiveDataAutoConfiguration`
*   `CassandraReactiveRepositoriesAutoConfiguration`
*   `CassandraRepositoriesAutoConfiguration`

## 3。卡珊德拉·Spring Boot 应用示例

为了说明这些概念，我们有一个简单的 Spring Boot web 应用程序，它的主要领域是车辆库存。

**为了简单起见，这个应用程序提供了基本的** ***CRUD*** **库存数据的操作。**

### 3.1。卡珊德拉·玛文依赖

Spring 提供了[*Spring-boot-starter-data-cassandra*](https://web.archive.org/web/20220524033958/https://search.maven.org/artifact/org.springframework.boot/spring-boot-starter-data-cassandra)模块用于 Cassandra 数据:

```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-cassandra</artifactId>
    <version>2.5.3</version>
</dependency>
```

我们还需要依赖 Datastax Cassandra 的[*Java-driver-core*](https://web.archive.org/web/20220524033958/https://search.maven.org/artifact/com.datastax.oss/java-driver-core)来启用集群连接和请求执行:

```java
<dependency> 
    <groupId>com.datastax.oss</groupId> 
    <artifactId>java-driver-core</artifactId> 
    <version>4.13.0</version> 
</dependency>
```

### 3.2。卡珊德拉配置

这里我们的*Cassandra config*类扩展了 Spring 的*abstract Cassandra configuration*，是 Spring 数据 Cassandra 配置的基类。

**这个 spring 类用于用*****cql session*****配置 Cassandra 客户端应用程序，以连接到 Cassandra 集群。**

此外，我们可以配置密钥空间名称和集群主机:

```java
@Configuration
public class CassandraConfig extends AbstractCassandraConfiguration {

    @Override
    protected String getKeyspaceName() {
        return "inventory";
    }

    @Override
    public String getContactPoints() {
        return "localhost";
    }

    @Override
    protected String getLocalDataCenter() {
         return "datacenter1";
    }
}
```

## 4。数据模型

下面的 [Cassandra 查询语言(*CQL*)](https://web.archive.org/web/20220524033958/https://docs.datastax.com/en/cql-oss/3.x/cql/cql_using/useCreateKeyspace.html)按名称 *创建 Cassandra keyspace 库存* :

```java
CREATE KEYSPACE inventory
WITH replication = {
    'class' : 'NetworkTopologyStrategy',
    'datacenter1' : 3
};
```

和这个[](https://web.archive.org/web/20220524033958/https://docs.datastax.com/en/cql-oss/3.x/cql/cql_using/useSimplePrimaryKeyConcept.html#useSimplePrimaryKeyConcept)创建一个 Cassandra 表按名称 *车辆* 在 *库存* 键位:

```java
use inventory;

CREATE TABLE vehicles (
   vin text PRIMARY KEY,
   year int,
   make varchar,
   model varchar
);
```

## 5。***@ DataCassandraTest*****用法**

让我们看看如何在单元测试中使用*@ DataCassandraTest*来测试应用程序的数据层。

*****@ DataCassandraTest*****注释导入需要的 Cassandra 自动配置模块，包括扫描*****@表*** **和*****@库*** **组件。** 这样一来，就使得*@ Autowire**储存库* 类成为可能。**

 **但是， **并不扫描和导入常规的*****@组件*** **和*****@配置属性*****bean。**

另外，在 JUnit 4 中，这个注释要和*@ run with(spring runner . class)*结合使用。

### 5.1。集成测试类

这里是*InventoryServiceIntegrationTest*类与*@ DataCassandraTest*注释和知识库*@ Autowired:*

```java
@RunWith(SpringRunner.class)
@DataCassandraTest
@Import(CassandraConfig.class)
public class InventoryServiceIntegrationTest {

    @Autowired
    private InventoryRepository repository;

    @Test
    public void givenVehiclesInDBInitially_whenRetrieved_thenReturnAllVehiclesFromDB() {
        List<Vehicle> vehicles = repository.findAllVehicles();

        assertThat(vehicles).isNotNull();
        assertThat(vehicles).isNotEmpty();
    }
}
```

我们还在上面添加了一个简单的测试方法。

为了更容易地运行这个测试，我们将使用一个 DockerCompose 测试容器，它建立了一个三节点 Cassandra 集群:

```java
public class InventoryServiceLiveTest {

    // ...

    public static DockerComposeContainer container =
            new DockerComposeContainer(new File("src/test/resources/compose-test.yml"));

    @BeforeAll
    static void beforeAll() {
        container.start();
    }

    @AfterAll
    static void afterAll() {
        container.stop();
    }
}
```

你可以在这里 找到 GitHub 项目 [中的 compose-test.yml 文件。](https://web.archive.org/web/20220524033958/https://github.com/Baeldung/datastax-cassandra/blob/main/spring-data-cassandra-test/src/test/resources/compose-test.yml)

### 5.2。存储库类

示例资源库类*inventory repository*定义了几个自定义 *JPA* 方法:

```java
@Repository
public interface InventoryRepository extends CrudRepository<Vehicle, String> {

    @Query("select * from vehicles")
    List<Vehicle> findAllVehicles();

    Optional<Vehicle> findByVin(@Param("vin") String vin);

    void deleteByVin(String vin);
}
```

我们还将定义“本地仲裁”的一致性级别，这意味着强一致性，通过将该属性添加到*application . yml*文件:

```java
spring:
  data:
    cassandra:
      request:
        consistency: local-quorum
```

## 6。其他***@ data xxxtest*****注解**

以下是测试任何应用程序的数据库层的一些其他类似注释:

*   *@DataJpaTest* 导入 *JPA* 储存库、*@实体* 类等。
*   *@ DataJdbcTest*导入 Spring 数据仓库、JdbcTemplate 等。
*   *@ Data mongotest*导入 Spring 数据 MongoDB 仓库，Mongo 模板和*@ Document*类
*   *@ dataneo4j test*导入 Spring 数据 Neo4j 仓库， *@Node* 类
*   *@ dataredist*导入 Spring 数据 Redis 仓库， *@RedisHash*

请访问 [Spring 文档](https://web.archive.org/web/20220524033958/https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#features.testing.spring-boot-applications.autoconfigured-tests) 获取更多测试注释和详细信息。

## 7。结论

在本文中，我们学习了*@ DataCassandraTest*注释如何加载一些 Spring Data Cassandra 自动配置组件。因此，它避免了加载许多不需要的 Spring 上下文模块。

一如既往，完整的源代码可以在 GitHub 上 [获得。](https://web.archive.org/web/20220524033958/https://github.com/Baeldung/datastax-cassandra/tree/main/spring-data-cassandra-test)**