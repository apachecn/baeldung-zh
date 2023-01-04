# 春季@DynamicPropertySource 指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-dynamicpropertysource>

## 1.概观

今天的应用程序不是孤立存在的:我们通常需要连接到各种外部组件，如 PostgreSQL、Apache Kafka、Cassandra、Redis 和其他外部 API。

在本教程中，我们将看到 Spring Framework 5.2.5 是如何通过引入[动态属性](https://web.archive.org/web/20220827110142/https://github.com/spring-projects/spring-framework/issues/24540)来帮助测试这样的应用程序的。

首先，我们将从定义问题开始，看看我们过去是如何用不太理想的方式解决问题的。然后，我们将介绍`[@DynamicPropertySource](https://web.archive.org/web/20220827110142/https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/test/context/DynamicPropertySource.html) `注释，看看它如何为同样的问题提供更好的解决方案。最后，我们还将看看来自测试框架的另一个解决方案，它可能优于纯 Spring 解决方案。

## 2.问题:动态属性

假设我们正在开发一个使用 PostgreSQL 作为数据库的典型应用程序。我们从一个简单的 JPA [实体](/web/20220827110142/https://www.baeldung.com/jpa-entities)开始:

```java
@Entity
@Table(name = "articles")
public class Article {

    @Id
    @GeneratedValue(strategy = IDENTITY)
    private Long id;

    private String title;

    private String content;

    // getters and setters
}
```

为了确保这个实体按预期工作，我们应该为它编写一个测试来验证它的数据库交互。因为这个测试需要与真实的数据库对话，所以我们应该事先设置一个 PostgreSQL 实例。

在测试执行期间，有**种不同的方法来设置这样的基础设施工具**。事实上，这种解决方案主要有三类:

*   在某个地方专门为测试建立一个单独的数据库服务器
*   使用一些轻量级的，测试专用的替代品或赝品，如 H2
*   让测试本身管理数据库的生命周期

因为我们不应该区分我们的测试和生产环境，所以与使用 H2 这样的[测试替身相比，有更好的替代方案。**第三个选项，除了使用真实的数据库之外，为测试提供了更好的隔离**。此外，有了 Docker 和](https://web.archive.org/web/20220827110142/https://monaa.dev/posts/mocks-considered-harmful/) [Testcontainers](/web/20220827110142/https://www.baeldung.com/spring-boot-testcontainers-integration-test) 这样的技术，很容易实现第三种选择。

如果我们使用像 Testcontainers 这样的技术，我们的测试工作流将会是这样的:

1.  在所有测试之前设置一个组件，如 PostgreSQL。通常，这些组件监听随机端口。
2.  进行测试。
3.  拆下组件。

**如果我们的 PostgreSQL 容器每次都要监听一个随机端口，那么我们应该以某种方式动态设置和更改`spring.datasource.url` 配置属性**。基本上，每个测试都应该有它自己版本的配置属性。

当配置是静态的时，我们可以使用 Spring Boot 的[配置管理](/web/20220827110142/https://www.baeldung.com/properties-with-spring)工具轻松管理它们。然而，当我们面对动态配置时，同样的任务可能具有挑战性。

现在我们知道了问题，让我们来看看传统的解决方案。

## 3.传统解决方案

**实现动态属性的第一种方法是使用自定义的 [`ApplicationContextInitializer`](https://web.archive.org/web/20220827110142/https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/context/ApplicationContextInitializer.html)** 。基本上，我们首先建立基础设施，并使用第一步中的信息定制 [`ApplicationContext`](/web/20220827110142/https://www.baeldung.com/spring-application-context) :

```java
@SpringBootTest
@Testcontainers
@ContextConfiguration(initializers = ArticleTraditionalLiveTest.EnvInitializer.class)
class ArticleTraditionalLiveTest {

    @Container
    static PostgreSQLContainer<?> postgres = new PostgreSQLContainer<>("postgres:11")
      .withDatabaseName("prop")
      .withUsername("postgres")
      .withPassword("pass")
      .withExposedPorts(5432);

    static class EnvInitializer implements ApplicationContextInitializer<ConfigurableApplicationContext> {

        @Override
        public void initialize(ConfigurableApplicationContext applicationContext) {
            TestPropertyValues.of(
              String.format("spring.datasource.url=jdbc:postgresql://localhost:%d/prop", postgres.getFirstMappedPort()),
              "spring.datasource.username=postgres",
              "spring.datasource.password=pass"
            ).applyTo(applicationContext);
        }
    }

    // omitted 
}
```

让我们来完成这个有点复杂的设置。JUnit 将首先创建并启动容器。容器准备好之后，Spring 扩展将调用初始化器将动态配置应用到 Spring [`Environment`](https://web.archive.org/web/20220827110142/https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/core/env/Environment.html) 。**显然，这种方法有点冗长和复杂。**

只有在这些步骤之后，我们才能编写我们的测试:

```java
@Autowired
private ArticleRepository articleRepository;

@Test
void givenAnArticle_whenPersisted_thenShouldBeAbleToReadIt() {
    Article article = new Article();
    article.setTitle("A Guide to @DynamicPropertySource in Spring");
    article.setContent("Today's applications...");

    articleRepository.save(article);

    Article persisted = articleRepository.findAll().get(0);
    assertThat(persisted.getId()).isNotNull();
    assertThat(persisted.getTitle()).isEqualTo("A Guide to @DynamicPropertySource in Spring");
    assertThat(persisted.getContent()).isEqualTo("Today's applications...");
}
```

## 4.`@DynamicPropertySource`

**Spring Framework 5.2.5 引入了`@DynamicPropertySource `注释，方便添加带有动态值**的属性。我们所要做的就是创建一个用`@DynamicPropertySource`注释的静态方法，并且只有一个`[DynamicPropertyRegistry](https://web.archive.org/web/20220827110142/https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/test/context/DynamicPropertyRegistry.html) `实例作为输入:

```java
@SpringBootTest
@Testcontainers
public class ArticleLiveTest {

    @Container
    static PostgreSQLContainer<?> postgres = new PostgreSQLContainer<>("postgres:11")
      .withDatabaseName("prop")
      .withUsername("postgres")
      .withPassword("pass")
      .withExposedPorts(5432);

    @DynamicPropertySource
    static void registerPgProperties(DynamicPropertyRegistry registry) {
        registry.add("spring.datasource.url", 
          () -> String.format("jdbc:postgresql://localhost:%d/prop", postgres.getFirstMappedPort()));
        registry.add("spring.datasource.username", () -> "postgres");
        registry.add("spring.datasource.password", () -> "pass");
    }

    // tests are same as before
}
```

如上所示，我们在给定的`DynamicPropertyRegistry `上使用 [`add(String, Supplier<Object>)`](https://web.archive.org/web/20220827110142/https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/test/context/DynamicPropertyRegistry.html#add-java.lang.String-java.util.function.Supplier-) 方法来给弹簧`Environment`添加一些属性。与我们之前看到的初始化器相比，这种方法要干净得多。**请注意，用`@DynamicPropertySource `标注的方法必须声明为`static`，并且必须只接受一个`DynamicPropertyRegistry`类型的参数。**

基本上，`@DynmicPropertySource `注释背后的主要动机是更容易地促进已经可能的事情。尽管它最初被设计为与 Testcontainers 一起工作，但是在我们需要与动态配置一起工作的任何地方都可以使用它。

## 5.另一种选择:测试夹具

到目前为止，在这两种方法中，**夹具设置和测试代码紧密地交织在一起**。有时候，这两个关注点的紧密耦合会使测试代码变得复杂，尤其是当我们需要设置很多东西的时候。想象一下，如果我们在一次测试中使用 PostgreSQL 和 Apache Kafka，基础设施设置会是什么样子。

除此之外，**基础设施设置和应用动态配置将在所有需要它们的测试中重复**。

为了避免这些缺点，**我们可以使用大多数测试框架提供的测试夹具设施**。例如，在 JUnit 5 中，我们可以定义一个[扩展](/web/20220827110142/https://www.baeldung.com/junit-5-extensions)，它在我们的测试类中的所有测试之前启动 PostgreSQL 实例，配置 Spring Boot，并在运行测试之后停止 PostgreSQL 实例:

```java
public class PostgreSQLExtension implements BeforeAllCallback, AfterAllCallback {

    private PostgreSQLContainer<?> postgres;

    @Override
    public void beforeAll(ExtensionContext context) {
        postgres = new PostgreSQLContainer<>("postgres:11")
          .withDatabaseName("prop")
          .withUsername("postgres")
          .withPassword("pass")
          .withExposedPorts(5432);

        postgres.start();
        String jdbcUrl = String.format("jdbc:postgresql://localhost:%d/prop", postgres.getFirstMappedPort());
        System.setProperty("spring.datasource.url", jdbcUrl);
        System.setProperty("spring.datasource.username", "postgres");
        System.setProperty("spring.datasource.password", "pass");
    }

    @Override
    public void afterAll(ExtensionContext context) {
        // do nothing, Testcontainers handles container shutdown
    }
} 
```

这里，我们实现了`[AfterAllCallback](https://web.archive.org/web/20220827110142/https://junit.org/junit5/docs/5.1.1/api/org/junit/jupiter/api/extension/AfterAllCallback.html) `和`[BeforeAllCallback](https://web.archive.org/web/20220827110142/https://junit.org/junit5/docs/5.1.1/api/org/junit/jupiter/api/extension/BeforeAllCallback.html) `来创建一个 JUnit 5 扩展。这样，JUnit 5 将在运行所有测试之前执行`beforeAll() `逻辑，并在运行测试之后执行`afterAll() `方法中的逻辑。使用这种方法，我们的测试代码将会非常简洁:

```java
@SpringBootTest
@ExtendWith(PostgreSQLExtension.class)
@DirtiesContext
public class ArticleTestFixtureLiveTest {
    // just the test code
}
```

这里，我们还向测试类添加了 [`@DirtiesContext`](/web/20220827110142/https://www.baeldung.com/spring-dirtiescontext) 注释。重要的是，这个**重新创建了应用程序上下文，并允许我们的测试类与一个单独的 PostgreSQL 实例交互，运行在一个随机端口**上。因此，这将针对一个单独的数据库实例，在彼此完全隔离的情况下执行我们的测试。

除了可读性更好之外，我们还可以通过添加`@ExtendWith(PostgreSQLExtension.class)`注释轻松地重用相同的功能。不需要像在其他两种方法中那样，将整个 PostgreSQL 设置复制粘贴到我们需要的任何地方。

## 6.结论

在本教程中，我们首先看到了测试一个依赖于数据库的 Spring 组件有多难。然后，我们介绍了这个问题的三种解决方案，每一种都在前一种解决方案的基础上有所改进。

像往常一样，所有的例子都可以在 GitHub 上找到[。](https://web.archive.org/web/20220827110142/https://github.com/eugenp/tutorials/tree/master/testing-modules/spring-testing-2)