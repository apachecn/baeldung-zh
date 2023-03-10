# Spring 数据 Web 支持

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-data-web-support>

## 1。概述

Spring MVC 和 [Spring Data](/web/20221208143917/https://www.baeldung.com/spring-data) 各自在简化应用程序开发方面做得很好。但是，如果我们把它们放在一起呢？

在本教程中，我们将看看 [Spring Data 的 web 支持](https://web.archive.org/web/20221208143917/https://docs.spring.io/spring-data/jpa/docs/current/reference/html/#core.web)和**它的`resolvers`如何减少样板文件并使我们的控制器更具表现力。**

在这个过程中，我们将会看到 [Querydsl](/web/20221208143917/https://www.baeldung.com/intro-to-querydsl) 以及它与 Spring 数据的集成。

## 2.一点背景知识

Spring Data 的 web 支持**是在标准 Spring MVC 平台之上实现的一组与 web 相关的特性，旨在向控制器层**添加额外的功能。

Spring Data web support 的功能是围绕几个`resolver`类构建的。解析器简化了与 [Spring 数据](/web/20221208143917/https://www.baeldung.com/spring-data)库互操作的控制器方法的实现，并且用额外的特性丰富了它们。

这些特性包括**从存储库层获取域对象**，**无需显式调用**存储库实现，以及**构造控制器响应**，这些响应可以作为支持分页和排序的数据段发送给客户端。

此外，对采用一个或多个请求参数的控制器方法的请求可以在内部解析为 [Querydsl](/web/20221208143917/https://www.baeldung.com/intro-to-querydsl) 查询。

## 3。一个演示 Spring Boot 项目

为了理解我们如何使用 Spring Data web support 来改进控制器的功能，让我们创建一个基本的 Spring Boot 项目。

我们的演示项目的 Maven 依赖项是相当标准的，有一些例外，我们将在后面讨论:

```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
<dependency>
    <groupId>com.h2database</groupId>
    <artifactId>h2</artifactId>
    <scope>runtime</scope>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-test</artifactId>
    <scope>test</scope>
</dependency>
```

在本例中，我们包含了`[spring-boot-starter-web](https://web.archive.org/web/20221208143917/https://search.maven.org/search?q=g:org.springframework.boot%20AND%20a:spring-boot-starter-web)`，因为我们将使用它来创建 RESTful 控制器，`[spring-boot-starter-jpa](https://web.archive.org/web/20221208143917/https://search.maven.org/search?q=g:org.springframework.boot%20AND%20a:spring-boot-starter-data-jpa)`用于实现持久层，`[spring-boot-starter-test](https://web.archive.org/web/20221208143917/https://search.maven.org/search?q=g:org.springframework.boot%20AND%20a:spring-boot-starter-test)`用于测试控制器 API。

因为我们将使用 [H2](/web/20221208143917/https://www.baeldung.com/java-in-memory-databases) 作为底层数据库，所以我们也包括了 [`com.h2database`](https://web.archive.org/web/20221208143917/https://search.maven.org/search?q=g:com.h2database%20AND%20a:h2) 。

让我们记住， **`spring-boot-starter-web`默认启用 Spring Data web 支持。**因此，我们不需要创建任何额外的`@Configuration`类来让它在我们的应用程序中工作。

相反，对于非 Spring Boot 项目，我们需要定义一个`@Configuration`类并用`@EnableWebMvc`和`@EnableSpringDataWebSupport`注释对其进行注释。

### 3.1.域类

现在，让我们将一个简单的`User` JPA 实体类添加到项目中，这样我们就可以使用一个工作域模型了:

```java
@Entity
@Table(name = "users")
public class User {

    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    private long id;
    private final String name;

    // standard constructor / getters / toString

}
```

### 3.2.存储库层

为了保持代码简单，我们的演示 Spring Boot 应用程序的功能将缩小到只从 H2 内存数据库中获取一些`User`实体。

Spring Boot 使得创建现成的、提供最少 CRUD 功能的存储库实现变得容易。因此，让我们定义一个与`User` JPA 实体一起工作的简单存储库接口:

```java
@Repository
public interface UserRepository extends PagingAndSortingRepository<User, Long> {}
```

除了扩展了 [`PagingAndSortingRepository`](/web/20221208143917/https://www.baeldung.com/spring-data-repositories) 之外，`UserRepository`接口的定义本身并不复杂。

**这向 Spring MVC 发出信号，使其能够对数据库记录进行自动分页和排序**。

### 3.3.控制器层

现在，我们至少需要实现一个基本的 RESTful 控制器，作为客户端和存储库层之间的中间层。

因此，让我们创建一个控制器类，它在其构造函数中接受一个`UserRepository`实例，并添加一个通过`id`查找`User`实体的方法:

```java
@RestController
public class UserController {

    @GetMapping("/users/{id}")
    public User findUserById(@PathVariable("id") User user) {
        return user;
    }
} 
```

### 3.4.运行应用程序

最后，让我们定义应用程序的主类，并用几个`User`实体填充 H2 数据库:

```java
@SpringBootApplication
public class Application {

    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }

    @Bean
    CommandLineRunner initialize(UserRepository userRepository) {
        return args -> {
            Stream.of("John", "Robert", "Nataly", "Helen", "Mary").forEach(name -> {
                User user = new User(name);
                userRepository.save(user);
            });
            userRepository.findAll().forEach(System.out::println);
        };
    }
}
```

现在，让我们运行应用程序。正如所料，我们看到启动时控制台打印出持久化的`User`实体列表:

```java
User{id=1, name=John}
User{id=2, name=Robert}
User{id=3, name=Nataly}
User{id=4, name=Helen}
User{id=5, name=Mary}
```

## 4。`DomainClassConverter`班

目前，`UserController`类只实现了`findUserById()`方法。

乍一看，该方法实现看起来相当简单。但它实际上在幕后封装了许多 Spring Data web 支持功能。

由于该方法将一个`User`实例作为参数，我们可能会认为需要在请求中显式传递域对象。但是，我们没有。

Spring MVC 使用`[DomainClassConverter](https://web.archive.org/web/20221208143917/https://docs.spring.io/spring-data/commons/docs/current/api/org/springframework/data/repository/support/DomainClassConverter.html)`类**将`id`路径变量转换为域类的`id`类型，并使用它从存储库层**获取匹配的域对象。不需要进一步查找。

例如，对`[http://localhost:8080/users/1](https://web.archive.org/web/20221208143917/http://localhost:8080/user/1)` 端点的 GET HTTP 请求将返回以下结果:

```java
{
  "id":1,
  "name":"John"
}
```

因此，我们可以创建一个集成测试并检查`findUserById()`方法的行为:

```java
@Test
public void whenGetRequestToUsersEndPointWithIdPathVariable_thenCorrectResponse() throws Exception {
    mockMvc.perform(MockMvcRequestBuilders.get("/users/{id}", "1")
      .contentType(MediaType.APPLICATION_JSON_UTF8))
      .andExpect(MockMvcResultMatchers.status().isOk())
      .andExpect(MockMvcResultMatchers.jsonPath("$.id").value("1"));
    }
} 
```

或者，我们可以使用 REST API 测试工具，比如 [Postman](/web/20221208143917/https://www.baeldung.com/postman-testing-collections) 来测试方法。

关于`DomainClassConverter`的好处是我们不需要在控制器方法中显式调用存储库实现。

**通过简单地指定`id`路径变量，以及一个可解析的域类实例，我们已经自动触发了域对象的查找**。

## 5。`PageableHandlerMethodArgumentResolver`班

Spring MVC 支持在控制器和存储库中使用 [`Pageable`](https://web.archive.org/web/20221208143917/https://docs.spring.io/spring-data/commons/docs/current/api/org/springframework/data/domain/Pageable.html) 类型。

简单地说，`Pageable`实例是保存分页信息的对象。因此，当我们向控制器方法传递一个`Pageable`参数时，Spring MVC 使用`[PageableHandlerMethodArgumentResolver](https://web.archive.org/web/20221208143917/https://docs.spring.io/spring-data/commons/docs/current/api/org/springframework/data/web/PageableHandlerMethodArgumentResolver.html)`类**将`Pageable`实例解析成一个`[PageRequest](https://web.archive.org/web/20221208143917/https://docs.spring.io/spring-data/commons/docs/current/api/org/springframework/data/domain/PageRequest.html)`对象**，这是一个简单的`Pageable`实现。

### 5.1。使用`Pageable`作为控制器方法参数

为了理解`PageableHandlerMethodArgumentResolver`类是如何工作的，让我们给`UserController`类添加一个新方法:

```java
@GetMapping("/users")
public Page<User> findAllUsers(Pageable pageable) {
    return userRepository.findAll(pageable);
} 
```

与`findUserById()`方法相反，这里我们需要调用存储库实现来获取数据库中持久化的所有`User` JPA 实体。

因为该方法采用了一个`Pageable`实例，所以它返回整个实体集的一个子集，存储在一个`Page<User>`对象中。

一个 [`Page`](https://web.archive.org/web/20221208143917/https://docs.spring.io/spring-data/commons/docs/current/api/org/springframework/data/domain/Page.html) 对象**是一个对象列表的子列表，它公开了我们可以用来检索分页结果**信息的几种方法，包括结果页面的总数，以及我们正在检索的页面的数量。

默认情况下，Spring MVC 使用`PageableHandlerMethodArgumentResolver`类来构造一个`PageRequest`对象，带有以下请求参数:

*   `page`:我们要检索的页面的索引——该参数为零索引，默认值为`0`
*   `size`:我们要检索的页数——默认值为`20`
*   `sort`:一个或多个我们可以用来对结果进行排序的属性，使用以下格式:`property1,property2(,asc|desc) –` 例如，`?sort=name&sort;=email,asc`

例如，对`[http://localhost:8080/user](https://web.archive.org/web/20221208143917/http://localhost:8080/users)[s](https://web.archive.org/web/20221208143917/http://localhost:8080/users)`端点的 GET 请求将返回以下输出:

```java
{
  "content":[
    {
      "id":1,
      "name":"John"
    },
    {
      "id":2,
      "name":"Robert"
    },
    {
      "id":3,
      "name":"Nataly"
    },
    {
      "id":4,
      "name":"Helen"
    },
    {
      "id":5,
      "name":"Mary"
    }],
  "pageable":{
    "sort":{
      "sorted":false,
      "unsorted":true,
      "empty":true
    },
    "pageSize":5,
    "pageNumber":0,
    "offset":0,
    "unpaged":false,
    "paged":true
  },
  "last":true,
  "totalElements":5,
  "totalPages":1,
  "numberOfElements":5,
  "first":true,
  "size":5,
  "number":0,
  "sort":{
    "sorted":false,
    "unsorted":true,
    "empty":true
  },
  "empty":false
} 
```

正如我们所看到的，响应包括了`first`、`pageSize`、`totalElements`和`totalPages` JSON 元素。这非常有用，因为前端可以使用这些元素轻松创建分页机制。

此外，我们可以使用集成测试来检查`findAllUsers()`方法:

```java
@Test
public void whenGetRequestToUsersEndPoint_thenCorrectResponse() throws Exception {
    mockMvc.perform(MockMvcRequestBuilders.get("/users")
      .contentType(MediaType.APPLICATION_JSON_UTF8))
      .andExpect(MockMvcResultMatchers.status().isOk())
      .andExpect(MockMvcResultMatchers.jsonPath("$['pageable']['paged']").value("true"));
}
```

### 5.2。定制寻呼参数

在许多情况下，我们需要定制分页参数。最简单的方法是使用 [`@PageableDefault`](https://web.archive.org/web/20221208143917/https://docs.spring.io/spring-data/commons/docs/current/api/org/springframework/data/web/PageableDefault.html) 标注:

```java
@GetMapping("/users")
public Page<User> findAllUsers(@PageableDefault(value = 2, page = 0) Pageable pageable) {
    return userRepository.findAll(pageable);
}
```

或者，我们可以使用`PageRequest`的`[of()](https://web.archive.org/web/20221208143917/https://docs.spring.io/spring-data/commons/docs/current/api/org/springframework/data/domain/PageRequest.html#of-int-int-org.springframework.data.domain.Sort-)`静态工厂方法来创建一个定制的`PageRequest`对象，并将其传递给存储库方法:

```java
@GetMapping("/users")
public Page<User> findAllUsers() {
    Pageable pageable = PageRequest.of(0, 5);
    return userRepository.findAll(pageable);
}
```

第一个参数是从零开始的页面索引，而第二个参数是我们想要检索的页面的大小。

在上面的例子中，我们创建了一个由`User`个实体组成的`PageRequest`对象，从第一个页面(`0`)开始，这个页面有`5`个条目。

此外，我们可以使用`page`和`size`请求参数构建一个`PageRequest`对象:

```java
@GetMapping("/users")
public Page<User> findAllUsers(@RequestParam("page") int page, 
  @RequestParam("size") int size, Pageable pageable) {
    return userRepository.findAll(pageable);
}
```

使用这个实现，对`[http://localhost:8080/users?page=0&size;=2](https://web.archive.org/web/20221208143917/http://localhost:8080/users?page=0&size=2)`端点的 GET 请求将返回`User`对象的第一页，结果页的大小将是 2:

```java
{
  "content": [
    {
      "id": 1,
      "name": "John"
    },
    {
      "id": 2,
      "name": "Robert"
    }
  ],

  // continues with pageable metadata

}
```

## 6。`SortHandlerMethodArgumentResolver`班

分页是有效管理大量数据库记录的实际方法。但是，就其本身而言，如果我们不能以某种特定的方式对记录进行排序，那么它就毫无用处。

为此，Spring MVC 提供了 [`SortHandlerMethodArgumentResolver`](https://web.archive.org/web/20221208143917/https://docs.spring.io/spring-data/commons/docs/current/api/org/springframework/data/web/SortHandlerMethodArgumentResolver.html) 类。解析器**根据请求参数或`[@SortDefault](https://web.archive.org/web/20221208143917/https://docs.spring.io/spring-data/commons/docs/current/api/org/springframework/data/web/SortDefault.html)`注释**自动创建`[Sort](https://web.archive.org/web/20221208143917/https://docs.spring.io/spring-data/commons/docs/current/api/org/springframework/data/domain/Sort.html)`实例。

### 6.1。使用`sort`控制器方法参数

为了清楚地了解`SortHandlerMethodArgumentResolver`类是如何工作的，让我们将`findAllUsersSortedByName()`方法添加到控制器类中:

```java
@GetMapping("/sortedusers")
public Page<User> findAllUsersSortedByName(@RequestParam("sort") String sort, Pageable pageable) {
    return userRepository.findAll(pageable);
}
```

在这种情况下，`SortHandlerMethodArgumentResolver`类将通过使用`sort`请求参数创建一个`Sort`对象。

因此，对`[http://localhost:8080/sortedusers?sort=name](https://web.archive.org/web/20221208143917/http://the http//localhost:8080/sortedusers)`端点的 GET 请求将返回一个 JSON 数组，其中包含按照`name`属性排序的`User`对象列表:

```java
{
  "content": [
    {
      "id": 4,
      "name": "Helen"
    },
    {
      "id": 1,
      "name": "John"
    },
    {
      "id": 5,
      "name": "Mary"
    },
    {
      "id": 3,
      "name": "Nataly"
    },
    {
      "id": 2,
      "name": "Robert"
    }
  ],

  // continues with pageable metadata

}
```

### 6.2。使用`Sort.by()`静态工厂方法

或者，我们可以通过使用 [`Sort.by()`](https://web.archive.org/web/20221208143917/https://docs.spring.io/spring-data/commons/docs/current/api/org/springframework/data/domain/Sort.html#by-java.lang.String...-) 静态工厂方法创建一个`Sort`对象，该方法接受一个非空的、非空的`array`的`String`属性进行排序。

在这种情况下，我们将只按`name`属性对记录进行排序:

```java
@GetMapping("/sortedusers")
public Page<User> findAllUsersSortedByName() {
    Pageable pageable = PageRequest.of(0, 5, Sort.by("name"));
    return userRepository.findAll(pageable);
}
```

当然，我们可以使用多个属性，只要它们是在 domain 类中声明的。

### 6.3。使用`@SortDefault`标注

同样，我们可以使用`@SortDefault`注释并得到相同的结果:

```java
@GetMapping("/sortedusers")
public Page<User> findAllUsersSortedByName(@SortDefault(sort = "name", 
  direction = Sort.Direction.ASC) Pageable pageable) {
    return userRepository.findAll(pageable);
}
```

最后，让我们创建一个集成测试来检查方法的行为:

```java
@Test
public void whenGetRequestToSorteredUsersEndPoint_thenCorrectResponse() throws Exception {
    mockMvc.perform(MockMvcRequestBuilders.get("/sortedusers")
      .contentType(MediaType.APPLICATION_JSON_UTF8))
      .andExpect(MockMvcResultMatchers.status().isOk())
      .andExpect(MockMvcResultMatchers.jsonPath("$['sort']['sorted']").value("true"));
} 
```

## 7。Querydsl 网络支持

正如我们在介绍中提到的，Spring Data web support 允许我们在控制器方法中使用请求参数来构建 [Querydsl](/web/20221208143917/https://www.baeldung.com/intro-to-querydsl) 的`[Predicate](https://web.archive.org/web/20221208143917/http://www.querydsl.com/static/querydsl/4.1.3/apidocs/com/querydsl/core/types/Predicate.html)`类型，并构建 Querydsl 查询。

为了简单起见，我们只看一下 Spring MVC 是如何将一个请求参数转换成一个 Querydsl [`BooleanExpression`](https://web.archive.org/web/20221208143917/http://www.querydsl.com/static/querydsl/4.1.3/apidocs/com/querydsl/core/types/dsl/BooleanExpression.html) ，再依次传递给一个 [`QuerydslPredicateExecutor`](https://web.archive.org/web/20221208143917/https://docs.spring.io/spring-data/commons/docs/current/api/org/springframework/data/querydsl/QuerydslPredicateExecutor.html) 。

为此，首先我们需要将`[querydsl-apt](https://web.archive.org/web/20221208143917/https://search.maven.org/search?q=g:com.querydsl%20AND%20a:querydsl-apt)`和`[querydsl-jpa](https://web.archive.org/web/20221208143917/https://search.maven.org/search?q=g:com.querydsl%20AND%20a:querydsl-jpa)` Maven 依赖项添加到`pom.xml`文件中:

```java
<dependency>
    <groupId>com.querydsl</groupId>
    <artifactId>querydsl-apt</artifactId>
</dependency>
<dependency>
    <groupId>com.querydsl</groupId>
    <artifactId>querydsl-jpa</artifactId>
</dependency>
```

接下来，我们需要重构我们的`UserRepository`接口，它也必须扩展`QuerydslPredicateExecutor`接口:

```java
@Repository
public interface UserRepository extends PagingAndSortingRepository<User, Long>,
  QuerydslPredicateExecutor<User> {
}
```

最后，让我们将下面的方法添加到`UserController`类中:

```java
@GetMapping("/filteredusers")
public Iterable<User> getUsersByQuerydslPredicate(@QuerydslPredicate(root = User.class) 
  Predicate predicate) {
    return userRepository.findAll(predicate);
}
```

尽管该方法的实现看起来相当简单，但它实际上在表面之下公开了许多功能。

假设我们想从数据库中获取所有匹配给定名称的`User`实体。我们可以通过调用该方法并在 URL 中指定一个`name`请求参数来实现这个**:**

`[http://localhost:8080/filteredusers?name=John](https://web.archive.org/web/20221208143917/http://localhost:8080/filteredusers?name=John)`

正如所料，该请求将返回以下结果:

```java
[
  {
    "id": 1,
    "name": "John"
  }
] 
```

正如我们之前所做的，我们可以使用集成测试来检查`getUsersByQuerydslPredicate()`方法:

```java
@Test
public void whenGetRequestToFilteredUsersEndPoint_thenCorrectResponse() throws Exception {
    mockMvc.perform(MockMvcRequestBuilders.get("/filteredusers")
      .param("name", "John")
      .contentType(MediaType.APPLICATION_JSON_UTF8))
      .andExpect(MockMvcResultMatchers.status().isOk())
      .andExpect(MockMvcResultMatchers.jsonPath("$[0].name").value("John"));
}
```

这只是 Querydsl web 支持如何工作的一个基本示例。但它实际上并没有显示出它的全部力量。 ****

现在，假设我们想要获取一个与给定的`id.` 匹配的`User`实体，在这种情况下，**我们只需要在 URL** 中传递一个`id`请求参数:

`[http://localhost:8080/filteredusers?id=2](https://web.archive.org/web/20221208143917/http://localhost:8080/filteredusers?id=1)`

在这种情况下，我们会得到这样的结果:

```java
[
  {
    "id": 2,
    "name": "Robert"
  }
]
```

显而易见，Querydsl web 支持是一个非常强大的特性，我们可以用它来获取匹配给定条件的数据库记录。

在所有情况下，整个过程归结为**只是用不同的请求参数**调用单个控制器方法。

## 8。结论

在本教程中，**我们深入了解了 Spring web support 的关键组件，并学习了如何在一个演示 Spring Boot 项目中使用它。**

像往常一样，本教程中展示的所有例子都可以在 [GitHub](https://web.archive.org/web/20221208143917/https://github.com/eugenp/tutorials/tree/master/persistence-modules/spring-data-rest-2) 上找到。