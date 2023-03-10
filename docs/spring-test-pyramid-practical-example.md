# 测试金字塔在基于弹簧的微服务中的实际应用

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-test-pyramid-practical-example>

## 1.概观

在本教程中，我们将了解流行的软件测试模型，称为测试金字塔。

我们将看到它在微服务领域的相关性。在这个过程中，我们将开发一个示例应用程序和相关测试来符合这个模型。此外，我们将尝试理解使用模型的好处和界限。

## 2.让我们后退一步

在我们开始理解任何像测试金字塔这样的特定模型之前，理解我们为什么需要一个模型是必要的。

测试软件的需求是与生俱来的，也许和软件开发的历史一样悠久。软件测试已经走过了从手工到自动化的漫长道路。然而，目标是一样的——交付符合规范的软件。

### 2.1.测试类型

实践中有几种不同类型的测试，侧重于特定的目标。可悲的是，词汇甚至对这些测试的理解都有很大的差异。

让我们回顾一些流行的和可能明确的观点:

*   **单元测试**:单元测试是**针对小代码单元的测试，最好是孤立的**。这里的目标是验证最小的可测试代码的行为，而不用担心代码库的其余部分。这自然意味着任何依赖都需要用 mock 或 stub 或类似的构造来替换。
*   集成测试(Integration Tests):虽然单元测试关注的是一段代码的内部，但事实是大量的复杂性存在于代码之外。代码单元需要协同工作，并且经常与数据库、消息代理或 web 服务等外部服务协同工作。集成测试是**在与外部依赖项集成时针对应用程序行为的测试**。
*   **UI 测试**:我们开发的一个软件，往往是通过一个界面来消费的，消费者可以与之交互。通常，一个应用程序有一个 web 界面。然而，API 接口正变得越来越流行。UI 测试**的目标是这些界面的行为，这些行为通常在本质上是高度交互的**。现在，这些测试可以以端到端的方式进行，或者用户界面也可以隔离测试。

### 2.2.手动测试与自动化测试

自从测试开始以来，软件测试一直是手工完成的，即使在今天，它也广泛地应用于实践中。但是，手动测试有限制也就不难理解了。为了让测试有用，它们必须全面且经常运行。

这在敏捷开发方法和云原生微服务架构中更加重要。然而，测试自动化的需求很早就被意识到了。

如果我们回想一下我们之前讨论过的不同类型的测试，随着我们从单元测试转移到集成和 UI 测试，它们的复杂性和范围都在增加。出于同样的原因，**单元测试的自动化更容易，也带来了大部分的好处**。随着我们的深入，自动化测试变得越来越困难，而好处却越来越少。

除了某些方面，到目前为止，自动化测试大多数软件行为是可能的。然而，这必须与自动化所需的努力相比，合理地权衡收益。

## 3.什么是测试金字塔？

既然我们已经收集了足够多的关于测试类型和工具的上下文，那么是时候理解测试金字塔到底是什么了。我们已经看到，我们应该编写不同类型的测试。

然而，我们应该如何决定我们应该为每种类型编写多少测试呢？需要注意的好处或陷阱是什么？这些是测试自动化模型(如测试金字塔)解决的一些问题。

Mike Cohn 在他的书《[用敏捷取得成功](https://web.archive.org/web/20221206082223/https://www.pearson.com/us/higher-education/program/Cohn-Succeeding-with-Agile-Software-Development-Using-Scrum/PGM201415.html)》中提出了一个被称为测试金字塔的概念。这个**展示了我们应该在不同粒度级别**编写的测试数量的可视化表示。

这个想法是，它应该在最细粒度的级别上是最高的，并且应该随着我们扩大测试范围而开始降低。这是典型的金字塔形状，因此得名:

[![pyramid](img/7855d190cbe5eb3651766948b50cc940.png)](/web/20221206082223/https://www.baeldung.com/wp-content/uploads/2019/10/Screenshot-2019-10-31-at-22.27.41.png)

虽然这个概念非常简单和优雅，但有效地采用它通常是一个挑战。理解这一点很重要，我们不能拘泥于模型的形状和它提到的测试类型。关键要点应该是:

*   我们必须编写不同粒度级别的测试
*   我们必须编写更少的测试，因为我们对它们的范围越来越粗糙

## 4.测试自动化工具

在所有主流编程语言中，有几种工具可以用来编写不同类型的测试。我们将讨论 Java 世界中一些流行的选择。

### 4.1.单元测试

*   测试框架:Java 中最流行的选择是 [JUnit](/web/20221206082223/https://www.baeldung.com/junit) ，它的下一代版本被称为 [JUnit5](/web/20221206082223/https://www.baeldung.com/junit-5) 。这个领域其他受欢迎的选择包括 [TestNG](/web/20221206082223/https://www.baeldung.com/testng) ，与 JUnit5 相比，它提供了一些与众不同的特性。然而，对于大多数应用程序来说，这两者都是合适的选择。
*   嘲讽:正如我们前面看到的，在执行单元测试的时候，我们肯定想要扣除大部分的依赖项，如果不是全部的话。为此，我们需要一种机制来用一个像 mock 或 stub 这样的测试 double 替换依赖项。Mockito 是一个优秀的框架，为 Java 中的真实对象提供 mocks。

### 4.2.集成测试

*   测试框架:集成测试的范围比单元测试更广，但是入口点通常是更高抽象层次上的相同代码。因此，适用于单元测试的测试框架也适用于集成测试。
*   嘲讽:集成测试的目标是用真实的集成测试应用程序的行为。然而，我们可能不想为了测试而使用实际的数据库或消息代理。许多数据库和类似的服务提供了一个[可嵌入版本](/web/20221206082223/https://www.baeldung.com/java-in-memory-databases)来编写集成测试。

### 4.3.UI 测试

*   测试框架:UI 测试的复杂程度取决于处理软件 UI 元素的客户。例如，网页的行为可能因设备、浏览器甚至操作系统而异。Selenium 是用 web 应用程序模拟浏览器行为的流行选择。然而，对于 REST APIs，像[放心](/web/20221206082223/https://www.baeldung.com/rest-assured-tutorial)这样的框架是更好的选择。
*   嘲讽:用户界面变得越来越互动，客户端使用 JavaScript 框架呈现，如 [Angular](https://web.archive.org/web/20221206082223/https://angular.io/) 和 [React](https://web.archive.org/web/20221206082223/https://reactjs.org/) 。使用像 [Jasmine](https://web.archive.org/web/20221206082223/https://jasmine.github.io/) 和 [Mocha](https://web.archive.org/web/20221206082223/https://mochajs.org/) 这样的测试框架来单独测试这样的 UI 元素更合理。显然，我们应该结合端到端测试来做这件事。

## 5.在实践中采用原则

让我们开发一个小应用程序来演示我们到目前为止所讨论的原则。我们将开发一个小型的微服务，并了解如何编写符合测试金字塔的测试。

[微服务架构](/web/20221206082223/https://www.baeldung.com/spring-microservices-guide)帮助**构建一个应用，作为围绕领域边界绘制的松散耦合服务的集合**。 [Spring Boot](/web/20221206082223/https://www.baeldung.com/spring-boot) 提供了一个出色的平台，几乎可以在任何时候通过用户界面和数据库等依赖项来引导微服务。

我们将利用这些来展示测试金字塔的实际应用。

### 5.1.应用架构

我们将开发一个基本的应用程序，允许我们存储和查询我们看过的电影:

[![Persitent Datastore](img/618b3f5d2c0169083d75d326c48613fa.png)](/web/20221206082223/https://www.baeldung.com/wp-content/uploads/2019/10/Screenshot-2019-10-31-at-22.28.37.png)

正如我们所看到的，它有一个简单的 REST 控制器，公开了三个端点:

```java
@RestController
public class MovieController {

    @Autowired
    private MovieService movieService;

    @GetMapping("/movies")
    public List<Movie> retrieveAllMovies() {
        return movieService.retrieveAllMovies();
    }

    @GetMapping("/movies/{id}")
    public Movie retrieveMovies(@PathVariable Long id) {
        return movieService.retrieveMovies(id);
    }

    @PostMapping("/movies")
    public Long createMovie(@RequestBody Movie movie) {
        return movieService.createMovie(movie);
    }
}
```

除了处理数据编组和解组之外，控制器仅仅路由到适当的服务:

```java
@Service
public class MovieService {

    @Autowired
    private MovieRepository movieRepository;

    public List<Movie> retrieveAllMovies() {
        return movieRepository.findAll();
    }

    public Movie retrieveMovies(@PathVariable Long id) {
        Movie movie = movieRepository.findById(id)
          .get();
        Movie response = new Movie();
        response.setTitle(movie.getTitle()
          .toLowerCase());
        return response;
    }

    public Long createMovie(@RequestBody Movie movie) {
        return movieRepository.save(movie)
          .getId();
    }
}
```

此外，我们有一个映射到持久层的 JPA 存储库:

```java
@Repository
public interface MovieRepository extends JpaRepository<Movie, Long> {
}
```

最后，我们保存和传递电影数据的简单域实体:

```java
@Entity
public class Movie {
    @Id
    private Long id;
    private String title;
    private String year;
    private String rating;

    // Standard setters and getters
}
```

有了这个简单的应用程序，我们现在就可以探索不同粒度和数量的测试了。

### 5.2.单元测试

首先，我们将了解如何为我们的应用程序编写一个简单的单元测试。从这个应用中可以明显看出，**大多数逻辑倾向于在服务层**中积累。这要求我们更广泛、更频繁地测试它——非常适合单元测试:

```java
public class MovieServiceUnitTests {

    @InjectMocks
    private MovieService movieService;

    @Mock
    private MovieRepository movieRepository;

    @Before
    public void setUp() throws Exception {
        MockitoAnnotations.initMocks(this);
    }

    @Test
    public void givenMovieServiceWhenQueriedWithAnIdThenGetExpectedMovie() {
        Movie movie = new Movie(100L, "Hello World!");
        Mockito.when(movieRepository.findById(100L))
          .thenReturn(Optional.ofNullable(movie));

        Movie result = movieService.retrieveMovies(100L);

        Assert.assertEquals(movie.getTitle().toLowerCase(), result.getTitle());
    }
}
```

这里，我们使用 JUnit 作为测试框架，使用 Mockito 模拟依赖关系。我们的服务，因为一些奇怪的需求，被期望以小写形式返回电影标题，这就是我们打算在这里测试的。有几个这样的行为，我们应该用这样的单元测试广泛地覆盖。

### 5.3.集成测试

在我们的单元测试中，我们模拟了存储库，它是我们对持久层的依赖。虽然我们已经彻底测试了服务层的行为，但当它连接到数据库时，我们仍然可能会有问题。这就是集成测试发挥作用的地方:

```java
@RunWith(SpringRunner.class)
@SpringBootTest
public class MovieControllerIntegrationTests {

    @Autowired
    private MovieController movieController;

    @Test
    public void givenMovieControllerWhenQueriedWithAnIdThenGetExpectedMovie() {
        Movie movie = new Movie(100L, "Hello World!");
        movieController.createMovie(movie);

        Movie result = movieController.retrieveMovies(100L);

        Assert.assertEquals(movie.getTitle().toLowerCase(), result.getTitle());
    }
}
```

请注意这里的一些有趣的差异。现在，我们没有嘲笑任何依赖。然而，**我们可能仍然需要根据情况模拟一些依赖关系**。此外，我们正在用`SpringRunner`运行这些测试。

这意味着我们将有一个 Spring 应用程序上下文和实时数据库来运行这个测试。难怪，这样会跑的慢一点！因此，我们在这里选择更少的场景来测试。

### 5.4.用户界面测试

最后，我们的应用程序有 REST 端点可供使用，它们可能有自己的细微差别需要测试。由于这是我们应用程序的用户界面，我们将在 UI 测试中重点介绍它。现在让我们使用放心测试应用程序:

```java
@RunWith(SpringRunner.class)
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
public class MovieApplicationE2eTests {

    @Autowired
    private MovieController movieController;

    @LocalServerPort
    private int port;

    @Test
    public void givenMovieApplicationWhenQueriedWithAnIdThenGetExpectedMovie() {
        Movie movie = new Movie(100L, "Hello World!");
        movieController.createMovie(movie);

        when().get(String.format("http://localhost:%s/movies/100", port))
          .then()
          .statusCode(is(200))
          .body(containsString("Hello World!".toLowerCase()));
    }
}
```

正如我们所看到的，**这些测试是用一个正在运行的应用程序来运行的，并通过可用的端点**来访问它。我们重点测试与 HTTP 相关的典型场景，比如响应代码。由于显而易见的原因，这些将是运行最慢的测试。

因此，我们必须非常仔细地选择要在这里测试的场景。我们应该只关注那些我们在之前的更精细的测试中无法涵盖的复杂性。

## 6.微服务的测试金字塔

现在我们已经看到了如何用不同的粒度编写测试，并适当地构建它们。然而，关键目标是通过更细粒度和更快速的测试来捕获大多数应用程序的复杂性。

虽然在**单片应用中解决这一问题为我们提供了理想的金字塔结构，但这对于其他架构来说可能并不必要**。

我们知道，微服务架构取一个应用，给我们一套松耦合的应用。这样，它将应用程序固有的一些复杂性外部化了。

现在，这些复杂性体现在服务之间的通信中。通过单元测试来捕获它们并不总是可能的，我们必须编写更多的集成测试。

虽然这可能意味着我们偏离了经典的金字塔模型，但这并不意味着我们也偏离了原则。请记住，**我们仍然通过尽可能精细的测试来捕捉大部分复杂性**。只要我们清楚这一点，一个可能不符合完美金字塔的模型仍然是有价值的。

这里需要理解的重要一点是，模型只有在交付价值时才是有用的。通常，该值取决于上下文，在这种情况下，上下文就是我们为应用程序选择的架构。因此，虽然使用模型作为指导方针是有帮助的，但是我们应该**关注底层原则**，并最终选择在我们的架构环境中有意义的东西。

## 7.与 CI 的集成

当我们将自动化测试集成到持续的集成管道中时，自动化测试的力量和好处在很大程度上得以实现。Jenkins 是以声明方式定义构建和[部署管道](/web/20221206082223/https://www.baeldung.com/jenkins-pipelines)的流行选择。

我们**可以集成我们在 Jenkins pipeline** 中自动化的任何测试。但是，我们必须理解，这增加了管道执行的时间。持续集成的主要目标之一是快速反馈。如果我们开始添加测试使它变慢，这可能会发生冲突。

关键的收获应该是**将快速的测试(如单元测试)添加到预计运行更频繁的管道中**。例如，我们可能不会从在每次提交时触发的管道中添加 UI 测试中受益。但是，这只是一个指导原则，最后，它取决于我们正在处理的应用程序的类型和复杂性。

## 8.结论

在本文中，我们介绍了软件测试的基础知识。我们理解不同的测试类型，以及使用一个可用的工具来自动化它们的重要性。

此外，我们理解了测试金字塔的含义。我们使用 Spring Boot 构建的微服务实现了这一点。

最后，我们讨论了测试金字塔的相关性，尤其是在微服务这样的架构环境中。