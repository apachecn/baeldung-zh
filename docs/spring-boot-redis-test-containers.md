# Spring Boot——用 Testcontainers 测试 Redis

> 原文:# t0]https://web . archive . org/web/202209930061024/https://www . BAE message . com/spring-boot-redis-test-containers

## 1.概观

Testcontainers 是一个 Java 库，用于创建临时 Docker 容器以进行单元测试。当我们想避免使用实际的服务器进行测试时，这很有用。

在本教程中，我们将学习如何在测试使用 Redis 的 Spring Boot 应用程序时使用 Testcontainers。

## 2.项目设置

使用任何测试容器的第一个先决条件是在运行测试的机器上安装 Docker。

一旦安装了 Docker，我们就可以开始设置我们的 Spring Boot 应用程序了。

在这个应用程序中，我们将设置一个 Redis 散列、一个存储库和一个使用存储库与 Redis 交互的服务。

### 2.1.属国

让我们从向项目添加所需的依赖项开始。

首先，我们将添加 [Spring Boot 起动机测试](https://web.archive.org/web/20220810173458/https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter-test)和 [Spring Boot 起动机数据 Redis](https://web.archive.org/web/20220810173458/https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter-data-redis) 相关性:

```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-test</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
```

接下来，让我们添加 [Testcontainers](https://web.archive.org/web/20220810173458/https://mvnrepository.com/artifact/org.testcontainers/testcontainers) 依赖项:

```java
<dependency> 
    <groupId>org.testcontainers</groupId> 
    <artifactId>testcontainers</artifactId> 
    <version>1.17.2</version> 
    <scope>test</scope> 
</dependency>
```

### 2.2.自动配置

由于我们不需要任何[高级配置](/web/20220810173458/https://www.baeldung.com/spring-data-redis-tutorial#the-redis-configuration)，我们可以使用自动配置来建立到 Redis 服务器的连接。

为此，我们需要将 Redis 连接细节添加到`application.properties`文件中:

```java
spring.redis.host=127.0.0.1
spring.redis.port=6379
```

## 3.应用程序设置

让我们从主要应用程序的代码开始。我们将构建一个向 Redis 数据库读写产品的小应用程序。

### 3.1.实体

让我们从`Product`类开始:

```java
@RedisHash("product")
public class Product implements Serializable {
    private String id;
    private String name;
    private double price;

    // Constructor,  getters and setters
}
```

`@RedisHash`注释用于告诉 Spring Data Redis，这个类应该存储在 Redis 散列中。对于不包含嵌套对象的实体来说，保存为哈希是很好的选择。

### 3.2.贮藏室ˌ仓库

接下来，我们可以为我们的`Product`散列:定义一个存储库

```java
@Repository
public interface ProductRepository extends CrudRepository<Product, String> {
}
```

CRUD 存储库接口已经实现了我们保存、更新、删除和查找产品所需的方法。所以我们不需要自己定义任何方法。

### 3.3.服务

最后，让我们创建一个使用`ProductRepository`执行读写操作的服务:

```java
@Service
public class ProductService {

    private final ProductRepository productRepository;

    public ProductService(ProductRepository productRepository) {
        this.productRepository = productRepository;
    }

    public Product getProduct(String id) {
        return productRepository.findById(id).orElse(null);
    }

    // other methods
}
```

然后，控制器或服务可以使用该服务对产品执行 CRUD 操作。

在实际应用中，这些方法可能包含更复杂的逻辑，但是出于本教程的目的，我们将只关注 Redis 交互。

## 4.测试

我们现在将为我们的`ProductService`编写测试来测试 CRUD 操作。

### 4.1.测试服务

让我们为`ProductService:`编写一个集成测试

```java
@Test
void givenProductCreated_whenGettingProductById_thenProductExistsAndHasSameProperties() {
    Product product = new Product("1", "Test Product", 10.0);
    productService.createProduct(product);
    Product productFromDb = productService.getProduct("1");
    assertEquals("1", productFromDb.getId());
    assertEquals("Test Product", productFromDb.getName());
    assertEquals(10.0, productFromDb.getPrice());
}
```

**这里假设 Redis 数据库正在属性中指定的 URL 上运行。如果我们没有运行 Redis 实例，或者我们的服务器无法连接到它，测试将会出错。**

### 4.2.用 Testcontainers 启动 Redis 容器

让我们通过在测试运行时运行 Redis 测试容器来解决这个问题。然后，我们将从代码本身更改连接细节。

让我们看看创建和运行测试容器的代码:

```java
static {
    GenericContainer<?> redis = 
      new GenericContainer<>(DockerImageName.parse("redis:5.0.3-alpine")).withExposedPorts(6379);
    redis.start();
}
```

让我们理解这段代码的不同部分:

*   我们已经从图片`redis:5.0.3-alpine`中创建了一个新的容器
*   默认情况下，Redis 实例将在端口`6379`上运行。为了暴露这个端口，我们可以使用`withExposedPorts()`方法。**它将暴露这个端口，并将其映射到主机上的一个随机端口**
*   `start()`方法将启动容器并等待它准备好
*   **我们已经将这段代码添加到了一个`static`代码块中，这样它就可以在依赖项被注入之前运行，并且测试也可以运行**

### 4.3.更改连接详细信息

此时，我们有一个 Redis 容器在运行，但是我们没有更改应用程序使用的连接细节。为此，我们需要做的就是使用系统属性覆盖`application.properties`文件中的连接细节:

```java
static {
    GenericContainer<?> redis = 
      new GenericContainer<>(DockerImageName.parse("redis:5.0.3-alpine")).withExposedPorts(6379);
    redis.start();
    System.setProperty("spring.redis.host", redis.getHost());
    System.setProperty("spring.redis.port", redis.getMappedPort(6379).toString());
}
```

我们已经将**属性设置为容器的 IP 地址。**

我们可以通过**获取端口`6379`的映射端口来设置`spring.redis.port`属性。**

现在，当测试运行时，它们将连接到运行在容器上的 Redis 数据库。

## 5.结论

在本文中，我们学习了如何使用 Redis Testcontainer 来运行测试。我们还研究了 Spring Data Redis 的某些方面，以了解如何使用它。

和往常一样，例子的源代码可以在 GitHub 上找到[。](https://web.archive.org/web/20220810173458/https://github.com/eugenp/tutorials/tree/master/spring-boot-modules/spring-boot-testing-2)