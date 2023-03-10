# REST API 中的 HTTP PUT 与 POST

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/rest-http-put-vs-post>

## 1.概观

在本教程中，我们将快速浏览两个重要的 HTTP 方法——PUT 和 POST——它们在 REST 架构中经常使用。众所周知，**开发人员在设计 RESTful web 服务时，有时会在这两种方法之间做出选择。因此，我们将在 Spring Boot 用一个 RESTful 应用程序的简单实现来解决这个问题。**

## 2.PUT vs POST 困境

在典型的 [REST 架构](/web/20220630011322/https://www.baeldung.com/cs/rest-architecture)中，客户端以 HTTP 方法的形式向服务器发送请求，以创建、检索、修改或销毁资源。虽然 PUT 和 POST 都可以用来创建资源，但是就它们的预期应用程序而言，它们之间有很大的不同。

根据 [RFC 2616](https://web.archive.org/web/20220630011322/https://tools.ietf.org/html/rfc2616) 标准，应该使用 POST 方法来请求服务器接受所包含的实体，作为请求 URI 所标识的现有资源的下属。这意味着**POST 方法调用将在一个资源集合下创建一个子资源**。

另一方面，应该使用 PUT 方法请求服务器在提供的请求 URI 下存储封装的实体。如果请求 URI 指向服务器上的现有资源，则提供的实体将被视为现有资源的修改版本。因此，**PUT 方法调用将创建一个新的资源或者更新一个现有的资源**。

这两种方法的另一个重要区别是， **PUT 是一个幂等方法，而 POST 不是**。例如，多次调用 PUT 方法将创建或更新同一个资源。相反，多个 POST 请求将导致多次创建同一个资源。

## 3.示例应用程序

为了演示 PUT 和 POST 之间的区别，我们将使用 [Spring Boot](/web/20220630011322/https://www.baeldung.com/spring-boot) 创建一个简单的 RESTful web 应用程序。该应用程序将存储人名和地址。

### 3.1.Maven 依赖性

首先，我们需要在我们的`pom.xml`文件中包含 Spring Web、Spring Data JPA 和内存中的 H2 数据库的依赖项:

```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
<dependency>
    <groupId>com.h2database</groupId>
    <artifactId>h2</artifactId>
    <scope>runtime</scope>
</dependency>
```

### 3.2.领域实体和存储库接口

让我们首先创建域对象。对于地址簿，让我们定义一个名为`Address`的`Entity`类，我们将使用它存储个人的地址信息。为了简单起见，对于我们的`Address`实体，我们将使用三个字段——`name`、`city`和`postalCode`:

```java
@Entity
public class Address {

    private @Id @GeneratedValue Long id;
    private String name;
    private String city;
    private String postalCode;

    // constructors, getters, and setters
}
```

下一步是从数据库中访问数据。为简单起见，我们将利用 [Spring Data JPA 的](/web/20220630011322/https://www.baeldung.com/the-persistence-layer-with-spring-data-jpa) `JpaRepository. `这将允许我们在不编写任何额外代码的情况下对数据执行 CRUD 功能:

```java
public interface AddressRepository extends JpaRepository<Address, Long> {
}
```

### 3.3.休息控制器

最后，我们需要为我们的应用程序定义 API 端点。我们将创建一个`RestController`，它将消费来自客户端的 HTTP 请求并发回适当的响应。

这里，**将定义一个@ `PostMapping`来创建新地址**并将它们存储在数据库**中，以及一个`@PutMapping`来根据请求 URI 更新地址簿**的内容。如果找不到 URI，它将创建一个新地址并将其存储在数据库中:

```java
@RestController
public class AddressController {

    private final AddressRepository repository;

    AddressController(AddressRepository repository) {
        this.repository = repository;
    }

    @PostMapping("/addresses")
    Address createNewAddress(@RequestBody Address newAddress) {
        return repository.save(newAddress);
    }

    @PutMapping("/addresses/{id}")
    Address replaceEmployee(@RequestBody Address newAddress, @PathVariable Long id) {

        return repository.findById(id)
            .map(address -> {
                address.setCity(newAddress.getCity());
                address.setPin(newAddress.getPostalCode());
                return repository.save(address);
            })
            .orElseGet(() -> {
                return repository.save(newAddress);
            });
    }
    //additional methods omitted
}
```

### 3.4.卷曲请求

现在，我们可以通过使用 cURL 向服务器发送示例 HTTP 请求来测试我们开发的应用程序。

为了创建新的地址，我们将以 JSON 格式封装数据，并通过 POST 请求发送它:

```java
curl -X POST --header 'Content-Type: application/json' \
    -d '{ "name": "John Doe", "city": "Berlin", "postalCode": "10585" }' \ 
    http://localhost:8080/addresses
```

现在，让我们更新我们创建的地址的内容。我们将使用 URL 中该地址的`id`发送一个 PUT 请求。在本例中，我们将更新刚刚创建的地址的`city`和`postalCode`部分——我们假设它是用`id` =1 保存的:

```java
curl -X PUT --header 'Content-Type: application/json' \
  -d '{ "name": "John Doe", "city": "Frankfurt", "postalCode": "60306" }' \ 
  http://localhost:8080/addresses/1
```

## 4.结论

在本教程中，我们了解了 HTTP 方法 PUT 和 POST 之间的概念差异。此外，我们还了解了如何使用开发 RESTful 应用程序的 Spring Boot 框架来实现这些方法。

总之，我们应该使用 POST 方法创建新资源，使用 PUT 方法更新现有资源。和往常一样，本教程的代码可以在 GitHub 上找到[。](https://web.archive.org/web/20220630011322/https://github.com/eugenp/tutorials/tree/master/spring-web-modules/spring-rest-http-2)