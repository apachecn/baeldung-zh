# 使用参数化的 SpringJUnit4ClassRunner

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/springjunit4classrunner-parameterized>

## 1.概观

在本教程中，我们将看到如何用一个`Parameterized` JUnit 测试运行器来参数化一个在 JUnit4 中实现的 [Spring 集成测试](/web/20221127023837/https://www.baeldung.com/spring-tests)。

## 2.`SpringJUnit4ClassRunner`

`SpringJUnit4ClassRunner `是 JUnit4 的`ClassRunner `的实现，即**将 Spring 的`TestContextManager `嵌入到 JUnit 测试**中。

`TestContextManager `是 Spring `TestContext `框架的入口点，因此管理对 Spring `ApplicationContext `的访问和 JUnit 测试类中的依赖注入。因此，`SpringJUnit4ClassRunner`使开发人员能够为 Spring 组件(如控制器和存储库)实现集成测试。

例如，我们可以为我们的`RestController`实现一个集成测试:

```java
@RunWith(SpringJUnit4ClassRunner.class)
@WebAppConfiguration
@ContextConfiguration(classes = WebConfig.class)
public class RoleControllerIntegrationTest {

    @Autowired
    private WebApplicationContext wac;

    private MockMvc mockMvc;

    private static final String CONTENT_TYPE = "application/text;charset=ISO-8859-1";

    @Before
    public void setup() throws Exception {
        this.mockMvc = MockMvcBuilders.webAppContextSetup(this.wac).build();
    }

    @Test
    public void givenEmployeeNameJohnWhenInvokeRoleThenReturnAdmin() throws Exception {
        this.mockMvc.perform(MockMvcRequestBuilders
          .get("/role/John"))
          .andDo(print())
          .andExpect(MockMvcResultMatchers.status().isOk())
          .andExpect(MockMvcResultMatchers.content().contentType(CONTENT_TYPE))
          .andExpect(MockMvcResultMatchers.content().string("ADMIN"));
    }
}
```

从测试中可以看出，我们的`Controller `接受一个用户名作为路径参数，并相应地返回用户角色。

现在，**为了用不同的用户名/角色组合测试这个 REST 服务，我们必须实现一个新的测试:**

```java
@Test
public void givenEmployeeNameDoeWhenInvokeRoleThenReturnEmployee() throws Exception {
    this.mockMvc.perform(MockMvcRequestBuilders
      .get("/role/Doe"))
      .andDo(print())
      .andExpect(MockMvcResultMatchers.status().isOk())
      .andExpect(MockMvcResultMatchers.content().contentType(CONTENT_TYPE))
      .andExpect(MockMvcResultMatchers.content().string("EMPLOYEE"));
}
```

对于可能有大量输入组合的服务，这个**可能会很快失控。**

为了在我们的测试类中避免这种重复，让我们看看如何使用`Parameterized `来实现接受多个输入的 JUnit 测试。

## 3.使用`Parameterized`

### 3.1.定义参数

`Parameterized`是一个定制的 JUnit 测试运行程序，它允许我们编写一个测试用例，并根据多个输入参数运行它:

```java
@RunWith(Parameterized.class)
@WebAppConfiguration
@ContextConfiguration(classes = WebConfig.class)
public class RoleControllerParameterizedIntegrationTest {

    @Parameter(value = 0)
    public String name;

    @Parameter(value = 1)
    public String role;

    @Parameters
    public static Collection<Object[]> data() {
        Collection<Object[]> params = new ArrayList();
        params.add(new Object[]{"John", "ADMIN"});
        params.add(new Object[]{"Doe", "EMPLOYEE"});

        return params;
    }

    //...
}
```

如上所示，我们使用了`@Parameters`注释来准备要注入 JUnit 测试的输入参数。我们还提供了这些值在`@Parameter `字段`name `和`role.`中的映射

但是现在，我们有另一个问题要解决— **JUnit 不允许在一个 JUnit 测试类中有多个 runners】。这意味着**我们不能利用`SpringJUnit4ClassRunner`将** `**TestContextManager** `嵌入到我们的测试类中。我们必须找到另一种嵌入`TestContextManager`的方法。**

幸运的是，Spring 提供了几个选项来实现这一点。我们将在下面的部分中讨论这些。

### 3.2.手动初始化`TestContextManager`

第一个选项非常简单，因为 Spring 允许我们手动初始化`TestContextManager `:

```java
@RunWith(Parameterized.class)
@WebAppConfiguration
@ContextConfiguration(classes = WebConfig.class)
public class RoleControllerParameterizedIntegrationTest {

    @Autowired
    private WebApplicationContext wac;

    private MockMvc mockMvc;

    private TestContextManager testContextManager;

    @Before
    public void setup() throws Exception {
        this.testContextManager = new TestContextManager(getClass());
        this.testContextManager.prepareTestInstance(this);

        this.mockMvc = MockMvcBuilders.webAppContextSetup(this.wac).build();
    }

    //...
}
```

值得注意的是，在这个例子中，我们使用了`Parameterized ` runner 而不是`SpringJUnit4ClassRunner.` 。接下来，我们在`setup()` 方法中初始化了`TestContextManager `。

现在，我们可以实现我们的参数化 JUnit 测试:

```java
@Test
public void givenEmployeeNameWhenInvokeRoleThenReturnRole() throws Exception {
    this.mockMvc.perform(MockMvcRequestBuilders
      .get("/role/" + name))
      .andDo(print())
      .andExpect(MockMvcResultMatchers.status().isOk())
      .andExpect(MockMvcResultMatchers.content().contentType(CONTENT_TYPE))
      .andExpect(MockMvcResultMatchers.content().string(role));
}
```

**JUnit 将执行这个测试用例两次**——对我们使用`@Parameters`注释定义的每组输入执行一次。

### 3.3.`SpringClassRule` 和`SpringMethodRule`

一般情况下，**不建议手动初始化`TestContextManager`**。相反，Spring 推荐使用`SpringClassRule`和`SpringMethodRule.`

`SpringClassRule`实现 JUnit 的`TestRule —` 编写测试用例的另一种方法。`TestRule`可用于取代之前用`@Before, ` `@BeforeClass, @After,`和`@AfterClass `方法完成的设置和清理操作。

`SpringClassRule`在 JUnit 测试类中嵌入`TestContextManager `的类级功能。它初始化`TestContextManager `并调用 Spring `TestContext.` 的设置和清理，因此，它提供了对`ApplicationContext`的依赖注入和访问。

除了`SpringClassRule`，我们还必须使用`SpringMethodRule`。它为`TestContextManager.`提供了实例级和方法级的功能

`SpringMethodRule `负责测试方法的准备。它还检查被标记为跳过的测试用例，并阻止它们运行。

让我们看看如何在我们的测试类中使用这种方法:

```java
@RunWith(Parameterized.class)
@WebAppConfiguration
@ContextConfiguration(classes = WebConfig.class)
public class RoleControllerParameterizedClassRuleIntegrationTest {
    @ClassRule
    public static final SpringClassRule scr = new SpringClassRule();

    @Rule
    public final SpringMethodRule smr = new SpringMethodRule();

    @Before
    public void setup() throws Exception {
        this.mockMvc = MockMvcBuilders.webAppContextSetup(this.wac).build();
    }

    //...
}
```

## 4.结论

在本文中，我们讨论了使用`Parameterized`测试运行器而不是`SpringJUnit4ClassRunner`来实现 Spring 集成测试的两种方法。我们看到了如何手动初始化`TestContextManager `，我们还看到了一个使用`SpringClassRule` 和`SpringMethodRule`的例子，这是 Spring 推荐的方法。

尽管我们在本文中只讨论了`Parameterized ` runner，**我们实际上可以使用这些方法中的任何一种与任何 JUnit runner** 一起编写 Spring 集成测试。

像往常一样，所有的示例代码都可以在 [GitHub](https://web.archive.org/web/20221127023837/https://github.com/eugenp/tutorials/tree/master/testing-modules/spring-testing) 上找到。