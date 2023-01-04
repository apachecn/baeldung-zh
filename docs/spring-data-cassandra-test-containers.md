# 使用带有 Spring 数据的测试容器

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-data-cassandra-test-containers>

## 1.概观

Apache Cassandra 是一个开源的分布式 NoSQL 数据库。它旨在**以快速读写性能处理大量数据，并且没有单点故障**。

在本教程中，我们将测试一个使用 Cassandra 数据库的 Spring Boot 应用程序。我们将解释如何使用来自 [Testcontainers](/web/20220707143816/https://www.baeldung.com/spring-boot-testcontainers-integration-test) 库中的 Cassandra 容器来设置集成测试。此外，我们将利用 Spring 数据仓库抽象来处理 Cassandra 的数据层。

最后，我们将展示如何在多个集成测试中重用共享的 Cassandra 容器实例。

## 2.测试容器

Testcontainers 是一个 Java 库，**提供了 Docker 容器**的轻量级、一次性实例。因此，我们通常在 Spring 中使用它对使用数据库的应用程序进行集成测试。Testcontainers 使我们能够在真实的数据库实例上进行测试，而不需要我们在本地机器上安装和管理数据库。

### 2.1.Maven 依赖性

Cassandra 容器在 [Cassandra Testcontainers 模块](https://web.archive.org/web/20220707143816/https://www.testcontainers.org/modules/databases/cassandra/)中可用。这使得可以使用容器化的 Cassandra 实例。

与 [`cassandra-unit`](/web/20220707143816/https://www.baeldung.com/spring-data-cassandra-tutorial) 库`,`不同，Testcontainers 库**与 JUnit 5** 完全兼容。让我们首先列出所需的 Maven 依赖项:

```java
<dependency>
    <groupId>org.testcontainers</groupId>
    <artifactId>testcontainers</artifactId>
    <version>1.15.3</version>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>org.testcontainers</groupId>
    <artifactId>cassandra</artifactId>
    <version>1.15.3</version>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>org.testcontainers</groupId>
    <artifactId>junit-jupiter</artifactId>
    <version>1.15.3</version>
    <scope>test</scope>
<dependency>
```

### 2.2.卡珊德拉集装箱

容器化的数据库实例通常用于集成测试。以及确保我们的数据访问层代码与特定的数据库版本完全兼容。

首先，我们需要用`@SpringBootTest`和`@Testcontainers`注释我们的测试类:

```java
@SpringBootTest
@Testcontainers
class CassandraSimpleIntegrationTest {}
```

然后，我们可以**定义一个 Cassandra 容器，并公开它的** **特定端口**:

```java
@Container
public static final CassandraContainer cassandra 
  = (CassandraContainer) new CassandraContainer("cassandra:3.11.2").withExposedPorts(9042); 
```

这里我们公开了容器端口`9042.`,但是我们应该注意到，Testcontainers 会将它链接到一个随机的主机端口，我们稍后可以得到这个端口。

使用上面的方法，Testcontainers 库自动为我们**启动一个 dockerized Cassandra 容器实例，与测试类**的生命周期保持一致:

```java
@Test
void givenCassandraContainer_whenSpringContextIsBootstrapped_thenContainerIsRunningWithNoExceptions() {
    assertThat(cassandra.isRunning()).isTrue();
}
```

现在我们有了一个运行的 Cassandra 容器。但是，Spring 应用程序还不知道它。

### 2.3.覆盖测试属性

为了让 Spring 数据能够**与 Cassandra 容器**建立连接，我们需要提供一些连接属性。我们将通过`java.lang.System`类定义系统属性来覆盖默认的 Cassandra 连接属性:

```java
@BeforeAll
static void setupCassandraConnectionProperties() {
    System.setProperty("spring.data.cassandra.keyspace-name", KEYSPACE_NAME);
    System.setProperty("spring.data.cassandra.contact-points", cassandra.getContainerIpAddress());
    System.setProperty("spring.data.cassandra.port", String.valueOf(cassandra.getMappedPort(9042)));
}
```

现在我们配置了 Spring 数据来连接我们的 Cassandra 容器。然而，我们仍然需要创建一个密钥空间。

### 2.4.创建密钥空间

作为在 Cassandra 中创建任何表之前的最后一步**，我们需要创建一个键空间:**

```java
private static void createKeyspace(Cluster cluster) {
    try (Session session = cluster.connect()) {
        session.execute("CREATE KEYSPACE IF NOT EXISTS " + KEYSPACE_NAME +
          " WITH replication = \n" +
          "{'class':'SimpleStrategy','replication_factor':'1'};");
    }
}
```

Cassandra 中的键空间非常类似于 RDBMS 中的数据库。它定义了数据如何在 Cassandra 集群中的节点上复制。

## 3.卡珊德拉的春季数据

Apache Cassandra **的 Spring Data 将核心的 Spring 概念应用到使用 Cassandra** 的应用程序开发中。它为丰富的对象映射提供了存储库、查询构建器和简单的注释。因此，它为使用不同数据库的 Spring 开发人员提供了一个熟悉的接口。

### 3.1.数据访问对象

让我们从准备一个简单的 DAO 类开始，稍后我们将在集成测试中使用它:

```java
@Table
public class Car {

    @PrimaryKey
    private UUID id;
    private String make;
    private String model;
    private int year;

    public Car(UUID id, String make, String model, int year) {
        this.id = id;
        this.make = make;
        this.model = model;
        this.year = year;
    }

    //getters, setters, equals and hashcode
}
```

这里的关键是**用`org.springframework.data.cassandra.core.mapping`包中的`@Table`注释**来注释这个类。事实上，这个注释支持自动域对象映射。

### 3.2.卡珊德拉知识库

Spring Data 使得为我们的 DAO 创建存储库变得非常简单。首先，我们需要在我们的 Spring Boot 主类中启用 Cassandra 存储库:

```java
@SpringBootApplication
@EnableCassandraRepositories(basePackages = "org.baeldung.springcassandra.repository")
public class SpringCassandraApplication {}
```

然后，我们只需要创建一个扩展`CassandraRepository`的接口:

```java
@Repository
public interface CarRepository extends CassandraRepository<Car, UUID> {}
```

在开始集成测试之前，我们需要定义两个额外的属性:

```java
spring.data.cassandra.local-datacenter=datacenter1
spring.data.cassandra.schema-action=create_if_not_exists
```

第一个属性定义默认的本地数据中心名称。第二个将确保 Spring Data 自动为我们创建所需的数据库表。我们应该注意到**这个设置不应该用在生产系统**中。

因为我们使用 Testcontainers，所以一旦测试完成，我们不需要担心删除表。每次我们运行测试时，都会为我们启动一个新的容器。

## 4.集成测试

既然已经设置了 Cassandra 容器、一个简单的 DAO 类和一个 Spring 数据存储库，我们就可以开始编写集成测试了。

### 4.1.保存记录测试

让我们从测试向 Cassandra 数据库插入新记录开始:

```java
@Test
void givenValidCarRecord_whenSavingIt_thenRecordIsSaved() {
    UUID carId = UUIDs.timeBased();
    Car newCar = new Car(carId, "Nissan", "Qashqai", 2018);

    carRepository.save(newCar);

    List<Car> savedCars = carRepository.findAllById(List.of(carId));
    assertThat(savedCars.get(0)).isEqualTo(newCar);
}
```

### 4.2.更新记录测试

然后，我们可以编写一个类似的测试来更新现有的数据库记录:

```java
@Test
void givenExistingCarRecord_whenUpdatingIt_thenRecordIsUpdated() {
    UUID carId = UUIDs.timeBased();
    Car existingCar = carRepository.save(new Car(carId, "Nissan", "Qashqai", 2018));

    existingCar.setModel("X-Trail");
    carRepository.save(existingCar);

    List<Car> savedCars = carRepository.findAllById(List.of(carId));
    assertThat(savedCars.get(0).getModel()).isEqualTo("X-Trail");
}
```

### 4.3.删除记录测试

最后，让我们编写一个删除现有数据库记录的测试:

```java
@Test
void givenExistingCarRecord_whenDeletingIt_thenRecordIsDeleted() {
    UUID carId = UUIDs.timeBased();
    Car existingCar = carRepository.save(new Car(carId, "Nissan", "Qashqai", 2018));

    carRepository.delete(existingCar);

    List<Car> savedCars = carRepository.findAllById(List.of(carId));
    assertThat(savedCars.isEmpty()).isTrue();
}
```

## 5.共享容器实例

大多数时候，当使用集成测试时，我们希望在多个测试中重用同一个数据库实例。我们可以通过使用多个嵌套的测试类来共享同一个容器实例:

```java
@Testcontainers
@SpringBootTest
class CassandraNestedIntegrationTest {

    private static final String KEYSPACE_NAME = "test";

    @Container
    private static final CassandraContainer cassandra 
      = (CassandraContainer) new CassandraContainer("cassandra:3.11.2").withExposedPorts(9042);

    // Set connection properties and create keyspace

    @Nested
    class ApplicationContextIntegrationTest {
        @Test
        void givenCassandraContainer_whenSpringContextIsBootstrapped_thenContainerIsRunningWithNoExceptions() {
            assertThat(cassandra.isRunning()).isTrue();
        }
    }

    @Nested
    class CarRepositoryIntegrationTest {

        @Autowired
        private CarRepository carRepository;

        @Test
        void givenValidCarRecord_whenSavingIt_thenRecordIsSaved() {
            UUID carId = UUIDs.timeBased();
            Car newCar = new Car(carId, "Nissan", "Qashqai", 2018);

            carRepository.save(newCar);

            List<Car> savedCars = carRepository.findAllById(List.of(carId));
            assertThat(savedCars.get(0)).isEqualTo(newCar);
        }

        // Tests for update and delete
    }
}
```

由于 Docker 容器需要时间来启动，**多个嵌套测试类之间的共享容器实例将确保更快的执行**。然而，我们应该注意，这个共享实例不会在测试之间被自动清除。

## 6.结论

在本文中，我们探索了如何使用 Cassandra 容器来测试使用 Cassandra 数据库的 Spring Boot 应用程序。

在示例中，我们介绍了如何设置 dockerized Cassandra 容器实例、覆盖测试属性、创建一个 keyspace、一个 DAO 类和一个 Cassandra 存储库接口。

我们看到了如何编写利用 Cassandra 容器的集成测试。因此，我们的示例测试不需要模仿。最后，我们看到了如何跨多个嵌套测试类重用同一个容器实例。

和往常一样，源代码可以在 GitHub 上的[处获得。](https://web.archive.org/web/20220707143816/https://github.com/eugenp/tutorials/tree/master/persistence-modules/spring-data-cassandra-2)