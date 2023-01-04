# 优化 Spring 集成测试

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-tests>

## 1.介绍

在本文中，我们将全面讨论使用 Spring 的集成测试以及如何优化它们。

首先，我们将简要讨论集成测试的重要性及其在现代软件中的位置，重点关注 Spring 生态系统。

稍后，我们将讨论多个场景，重点是 web 应用程序。

**接下来，我们将讨论一些提高测试速度的策略**，通过学习不同的方法，这些方法可能会影响我们设计测试的方式和设计应用程序本身的方式。

在开始之前，重要的是要记住这是一篇基于经验的观点文章。有些东西可能适合你，有些可能不适合。

最后，本文使用 Kotlin 作为代码示例，以使它们尽可能简洁，但是概念并不特定于这种语言，代码片段应该对 Java 和 Kotlin 开发人员一样有意义。

## 2.集成测试

集成测试是自动化测试套件的基础部分。尽管如果我们遵循[健康测试金字塔](https://web.archive.org/web/20221212193357/https://martinfowler.com/articles/practical-test-pyramid.html)，它们不应该像单元测试一样多。依赖像 Spring 这样的框架会让我们需要大量的集成测试来降低系统的某些行为的风险。

**我们越是通过使用 Spring 模块(数据、安全、社交……)来简化代码，就越需要集成测试。**当我们将基础设施的一点一滴转移到`@Configuration`类中时，这一点变得尤其真实。

我们不应该“测试框架”，但是我们当然应该验证框架是否满足我们的需求。

集成测试有助于我们建立信心，但这是有代价的:

*   这是一个较慢的执行速度，这意味着较慢的构建
*   此外，集成测试意味着更广泛的测试范围，这在大多数情况下并不理想

考虑到这一点，我们将尝试找到一些解决方案来缓解上述问题。

## 3.测试 Web 应用程序

Spring 提供了几个选项来测试 web 应用程序，大多数 Spring 开发人员都很熟悉它们，它们是:

*   [`MockMvc`](/web/20221212193357/https://www.baeldung.com/integration-testing-in-spring) :模仿 servlet API，对非反应式 web 应用程序有用
*   [`TestRestTemplate`](/web/20221212193357/https://www.baeldung.com/spring-boot-testresttemplate) :可以用来指向我们的应用程序，对于不需要模仿 servlets 的非反应式 web 应用程序非常有用
*   WebTestClient 是一个反应式 web 应用的测试工具，既可以模拟请求/响应，也可以访问真实的服务器

因为我们已经有了涉及这些主题的文章，所以我们不会花时间来讨论它们。

如果你想深入了解，请随意查看。

## 4.优化执行时间

集成测试很棒。他们给了我们很大的信心。此外，如果实施得当，他们可以用一种非常清晰的方式描述我们的应用程序的意图，减少嘲笑和设置噪音。

然而，随着我们的应用程序的成熟和开发的堆积，构建时间不可避免地会增加。随着构建时间的增加，每次都运行所有测试可能变得不切实际。

此后，影响我们的反馈循环，走上最佳开发实践的道路。

此外，集成测试本身就很昂贵。启动某种持久化、发送请求(即使它们从未离开过`localhost`)或者做一些 IO 只是需要时间。

关注我们的构建时间非常重要，包括测试执行。在春天，我们可以运用一些技巧来保持较低的温度。

在接下来的部分中，我们将讨论几个要点来帮助我们优化构建时间，以及一些可能影响构建速度的陷阱:

*   明智地使用配置文件——配置文件如何影响性能
*   重新思考嘲笑是如何影响表现的
*   重构`@MockBean `—提高性能的替代方案
*   仔细思考@ `DirtiesContext – `一个有用但危险的注释以及如何不使用它
*   使用测试切片——这是一个很酷的工具，可以帮助或帮助我们
*   使用类继承——一种安全组织测试的方法
*   状态管理——避免古怪测试的良好实践
*   重构到单元测试中——获得可靠和快速构建的最佳方式

我们开始吧！

### 4.1.明智地使用配置文件

简介是一个非常好的工具。也就是说，简单的标签可以启用或禁用我们应用程序的某些区域。我们甚至可以[用它们实现特征标志](/web/20221212193357/https://www.baeldung.com/spring-feature-flags)！

随着我们的概要文件越来越丰富，在我们的集成测试中时不时地进行交换是很有诱惑力的。有一些方便的工具可以做到这一点，比如`@ActiveProfiles`。然而，**每次我们用一个新的概要文件进行测试，一个新的`ApplicationContext`就会被创建。**

创建应用程序上下文对于一个没有任何内容的普通的 spring boot 应用程序来说可能很快。添加一个 ORM 和几个模块，它会迅速飙升到 7 秒以上。

添加一堆概要文件，将它们分散到几个测试中，我们将很快得到一个 60 秒以上的构建(假设我们将测试作为构建的一部分运行——我们应该这样做)。

一旦我们面对一个足够复杂的应用程序，解决这个问题是令人生畏的。然而，如果我们事先仔细计划，保持合理的构建时间就变得微不足道了。

当涉及到集成测试中的概要文件时，我们可以记住一些技巧:

*   创建一个聚合配置文件，即`test`，包括所有需要的配置文件–在任何地方都坚持我们的测试配置文件
*   在设计我们的概要文件时要考虑到可测试性。如果我们最终不得不转换配置文件，也许有更好的方法
*   在一个集中的地方陈述我们的测试概要——我们稍后将讨论这个
*   避免测试所有配置文件组合。或者，我们可以在每个环境中使用一个 e2e 测试套件，用特定的配置文件集测试应用程序

### 4.2.`@MockBean`的问题

是一个非常强大的工具。

当我们需要一些弹簧魔术，但又想模仿某个特定的组件时，`@MockBean`就派上了用场。但这是有代价的。

**每次`@MockBean`出现在一个类中，`ApplicationContext`缓存被标记为脏，因此运行器将在测试类完成后清理缓存。**这又给我们的构建增加了额外的几秒钟。

这是一个有争议的问题，但尝试使用实际的应用程序而不是嘲笑这个特定的场景可能会有所帮助。当然，这里没有灵丹妙药。当我们不允许自己模仿依赖时，界限就变得模糊了。

我们可能会想:当我们想要测试的只是我们的 REST 层时，为什么我们还要坚持呢？这是一个公平的观点，总会有妥协。

然而，记住一些原则，这实际上可能会变成一个优势，导致测试和我们的应用程序的更好设计，并减少测试时间。

### 4.3.重构`@MockBean`

**在这一节中，我们将尝试使用`@MockBean`重构一个“慢”测试，使其重用缓存的`ApplicationContext`。**

让我们假设我们想要测试一个创建用户的帖子。如果我们在模仿——使用`@MockBean`,我们可以简单地验证我们的服务是由一个很好地序列化的用户调用的。

如果我们正确地测试了我们的服务，这种方法应该足够了:

```
class UsersControllerIntegrationTest : AbstractSpringIntegrationTest() {

    @Autowired
    lateinit var mvc: MockMvc

    @MockBean
    lateinit var userService: UserService

    @Test
    fun links() {
        mvc.perform(post("/users")
          .contentType(MediaType.APPLICATION_JSON)
          .content("""{ "name":"jose" }"""))
          .andExpect(status().isCreated)

        verify(userService).save("jose")
    }
}

interface UserService {
    fun save(name: String)
}
```

但是我们想避免`@MockBean`。所以我们将最终持久化实体(假设这是服务所做的)。

这里最天真的方法是测试副作用:发布后，我的用户在我的数据库中，在我们的例子中，这将使用 JDBC。

然而，这违反了测试界限:

```
@Test
fun links() {
    mvc.perform(post("/users")
      .contentType(MediaType.APPLICATION_JSON)
      .content("""{ "name":"jose" }"""))
      .andExpect(status().isCreated)

    assertThat(
      JdbcTestUtils.countRowsInTable(jdbcTemplate, "users"))
      .isOne()
}
```

在这个特定的例子中，我们违反了测试边界，因为我们将我们的应用程序视为一个 HTTP 黑盒来发送用户，但后来我们断言使用实现细节，也就是说，我们的用户已经被持久化在某个 DB 中。

如果我们通过 HTTP 运行我们的应用程序，我们也可以通过 HTTP 断言结果吗？

```
@Test
fun links() {
    mvc.perform(post("/users")
      .contentType(MediaType.APPLICATION_JSON)
      .content("""{ "name":"jose" }"""))
      .andExpect(status().isCreated)

    mvc.perform(get("/users/jose"))
      .andExpect(status().isOk)
}
```

如果我们采用最后一种方法，会有一些优势:

*   我们的测试会开始得更快(可以说，执行起来可能会花一点时间，但应该会有回报)
*   此外，我们的测试没有意识到与 HTTP 边界无关的副作用，即 DBs
*   最后，我们的测试清楚地表达了系统的意图:如果你发布，你将能够获得用户

当然，由于各种原因，这并不总是可能的:

*   我们可能没有“副作用”端点:这里的一个选项是考虑创建“测试端点”
*   复杂度太高，无法触及整个应用程序:这里的一个选择是考虑切片(我们稍后将讨论它们)

### 4.4.仔细思考`@DirtiesContext`

有时，我们可能需要在测试中修改`ApplicationContext`。对于这个场景，`@DirtiesContext`正好提供了这个功能。

出于上面提到的同样原因，在执行时间方面，`@DirtiesContext `是一种极其昂贵的资源，因此，我们应该小心。

**`@DirtiesContext `的一些误用包括应用缓存重置或内存数据库重置。**在集成测试中有更好的方法来处理这些场景，我们将在后面的章节中介绍一些。

### 4.5.使用测试切片

**测试切片是 1.4 中引入的 Spring Boot 功能。这个想法相当简单，Spring 将为你的应用程序的特定部分创建一个简化的应用程序上下文。**

此外，该框架将负责最低限度的配置。

在 Spring Boot，现成的切片数量相当可观，我们也可以创建自己的切片:

*   `@JsonTest: `注册 JSON 相关组件
*   `@DataJpaTest`:注册 JPA beans，包括可用的 ORM
*   对原始 JDBC 测试有用，处理数据源和内存中的数据库，没有 ORM 多余的东西
*   尝试提供一个内存中的 mongo 测试设置
*   没有应用程序其余部分的模拟 MVC 测试片段
*   …(我们可以检查[源](https://web.archive.org/web/20221212193357/https://github.com/spring-projects/spring-boot/tree/master/spring-boot-project/spring-boot-test-autoconfigure/src/main/java/org/springframework/boot/test/autoconfigure)来找到它们)

如果明智地使用这个特性，它可以帮助我们构建狭窄的测试，而不会在性能方面有太大的损失，特别是对于中小型应用程序。

然而，如果我们的应用程序不断增长，它也会堆积起来，因为它为每个片创建了一个(小的)应用程序上下文。

### 4.6.使用类继承

使用一个单一的`AbstractSpringIntegrationTest`类作为我们所有集成测试的父类是保持快速构建的一种简单、强大和实用的方式。

如果我们提供一个可靠的设置，我们的团队会简单地扩展它，因为我们知道一切都“正常工作”。这样我们就可以更少地担心管理状态或配置框架，并专注于手头的问题。

我们可以在那里设置所有的测试要求:

*   春季赛跑者——或者最好是规则，以防我们以后需要其他赛跑者
*   个人资料–理想情况下是我们的汇总`test `个人资料
*   初始配置——设置应用程序的状态

让我们来看一个简单的基类，它考虑了前面的几点:

```
@SpringBootTest
@ActiveProfiles("test")
abstract class AbstractSpringIntegrationTest {

    @Rule
    @JvmField
    val springMethodRule = SpringMethodRule()

    companion object {
        @ClassRule
        @JvmField
        val SPRING_CLASS_RULE = SpringClassRule()
    }
}
```

### 4.7.状态管理

重要的是要记住单元测试[中的‘单元’来自于](https://web.archive.org/web/20221212193357/https://content.pivotal.io/blog/what-is-a-unit-test-the-answer-might-surprise-you)。简单地说，这意味着我们可以在任何时候运行一个测试(或者一个子集)来获得一致的结果。

因此，在每个测试开始之前，状态应该是干净的和已知的。

换句话说，无论是单独执行还是与其他测试一起执行，测试的结果都应该是一致的。

这个想法同样适用于集成测试。在开始新的测试之前，我们需要确保我们的应用程序有一个已知的(和可重复的)状态。我们为了加速而重用的组件越多(应用上下文、数据库、队列、文件……)，状态污染的机会就越大。

假设我们全部使用了类继承，现在，我们有了一个管理状态的中心位置。

让我们增强我们的抽象类，以确保我们的应用程序在运行测试之前处于已知状态。

在我们的例子中，我们将假设有几个存储库(来自各种数据源)和一个`Wiremock`服务器:

```
@SpringBootTest
@ActiveProfiles("test")
@AutoConfigureWireMock(port = 8666)
@AutoConfigureMockMvc
abstract class AbstractSpringIntegrationTest {

    //... spring rules are configured here, skipped for clarity

    @Autowired
    protected lateinit var wireMockServer: WireMockServer

    @Autowired
    lateinit var jdbcTemplate: JdbcTemplate

    @Autowired
    lateinit var repos: Set<MongoRepository<*, *>>

    @Autowired
    lateinit var cacheManager: CacheManager

    @Before
    fun resetState() {
        cleanAllDatabases()
        cleanAllCaches()
        resetWiremockStatus()
    }

    fun cleanAllDatabases() {
        JdbcTestUtils.deleteFromTables(jdbcTemplate, "table1", "table2")
        jdbcTemplate.update("ALTER TABLE table1 ALTER COLUMN id RESTART WITH 1")
        repos.forEach { it.deleteAll() }
    }

    fun cleanAllCaches() {
        cacheManager.cacheNames
          .map { cacheManager.getCache(it) }
          .filterNotNull()
          .forEach { it.clear() }
    }

    fun resetWiremockStatus() {
        wireMockServer.resetAll()
        // set default requests if any
    }
}
```

### 4.8.重构到单元测试中

这大概是最重要的一点。我们会发现自己一遍又一遍地进行一些集成测试，这些测试实际上是在执行我们应用程序的一些高级策略。

每当我们发现一些集成测试在测试一堆核心业务逻辑的案例时，是时候重新思考我们的方法并把它们分解成单元测试了。

成功实现这一点的一种可能模式是:

*   识别测试多个核心业务逻辑场景的集成测试
*   复制套件，并将副本重构到单元测试中——在这个阶段，我们可能还需要分解产品代码以使其可测试
*   让所有测试都变绿
*   留下一个在集成套件中足够显著的快乐路径样本——我们可能需要重构或加入并重塑一些样本
*   移除剩余的集成测试

Michael Feathers 介绍了许多实现这一点的技术，以及更多有效处理遗留代码的技术。

## 5.摘要

在本文中，我们介绍了集成测试，重点是 Spring。

首先，我们讨论了集成测试的重要性，以及为什么它们与 Spring 应用程序特别相关。

之后，我们总结了一些工具，这些工具对于 Web 应用程序中某些类型的集成测试可能会派上用场。

最后，我们讨论了一系列降低测试执行时间的潜在问题，以及改进它的技巧。