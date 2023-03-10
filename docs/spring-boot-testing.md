# 在 Spring Boot 测试

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-boot-testing>

## 1。概述

在本教程中，我们将看看在 Spring Boot 使用框架支持编写测试。我们将涵盖可以独立运行的单元测试，以及在执行测试之前引导 Spring 上下文的集成测试。

如果你刚到 Spring Boot，看看我们的[Spring Boot 介绍](/web/20221212193351/https://www.baeldung.com/spring-boot-start)。

## 延伸阅读:

## [探索 Spring Boot TestRestTemplate](/web/20221212193351/https://www.baeldung.com/spring-boot-testresttemplate)

Learn how to use the new TestRestTemplate in Spring Boot to test a simple API.[Read more](/web/20221212193351/https://www.baeldung.com/spring-boot-testresttemplate) →

## [Spring Boot @ rest client test 快速指南](/web/20221212193351/https://www.baeldung.com/restclienttest-in-spring-boot)

A quick and practical guide to the @RestClientTest annotation in Spring Boot[Read more](/web/20221212193351/https://www.baeldung.com/restclienttest-in-spring-boot) →

## [将 Mockito Mocks 注入春豆](/web/20221212193351/https://www.baeldung.com/injecting-mocks-in-spring)

This article will show how to use dependency injection to insert Mockito mocks into Spring Beans for unit testing.[Read more](/web/20221212193351/https://www.baeldung.com/injecting-mocks-in-spring) →

## 2。项目设置

本文中我们将要使用的应用程序是一个 API，它提供了对`Employee`资源的一些基本操作。这是一个典型的分层架构 API 调用从`Controller`到`Service`再到`Persistence`层进行处理。

## 3。Maven 依赖关系

让我们首先添加我们的测试依赖项:

```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-test</artifactId>
    <scope>test</scope>
    <version>2.5.0</version>
</dependency>
<dependency>
    <groupId>com.h2database</groupId>
    <artifactId>h2</artifactId>
    <scope>test</scope>
</dependency>
```

[`spring-boot-starter-test`](https://web.archive.org/web/20221212193351/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.springframework.boot%22%20AND%20a%3A%22spring-boot-starter-test%22) 是主要的依赖项，包含了我们测试所需的大部分元素。

H2 数据库是我们的内存数据库。它消除了为测试目的配置和启动实际数据库的需要。

### 3.1。JUnit 4

从 Spring Boot 2.4 开始，JUnit 5 的老式引擎已经从`spring-boot-starter-test`中移除。如果我们仍然想使用 JUnit 4 编写测试，我们需要添加以下 Maven 依赖项:

```java
<dependency>
    <groupId>org.junit.vintage</groupId>
    <artifactId>junit-vintage-engine</artifactId>
    <scope>test</scope>
    <exclusions>
        <exclusion>
            <groupId>org.hamcrest</groupId>
            <artifactId>hamcrest-core</artifactId>
        </exclusion>
    </exclusions>
</dependency>
```

## 4。用`@SpringBootTest` 进行集成测试

顾名思义，集成测试侧重于集成应用程序的不同层。这也意味着没有嘲笑。

理想情况下，我们应该将集成测试从单元测试中分离出来，并且不应该与单元测试一起运行。我们可以通过使用一个不同的概要文件来运行集成测试。这样做的几个原因可能是集成测试很耗时，可能需要一个实际的数据库来执行。

然而在本文中，我们不会关注这个问题，相反，我们将利用内存中的 H2 持久性存储。

集成测试需要启动一个容器来执行测试用例。因此，这需要一些额外的设置—所有这些在 Spring Boot 都很容易:

```java
@RunWith(SpringRunner.class)
@SpringBootTest(
  webEnvironment = SpringBootTest.WebEnvironment.MOCK,
  classes = Application.class)
@AutoConfigureMockMvc
@TestPropertySource(
  locations = "classpath:application-integrationtest.properties")
public class EmployeeRestControllerIntegrationTest {

    @Autowired
    private MockMvc mvc;

    @Autowired
    private EmployeeRepository repository;

    // write test cases here
}
```

**当我们需要引导整个容器时, `@SpringBootTest`注释非常有用。**注释通过创建将在我们的测试中使用的`ApplicationContext`来工作。

我们可以使用`@SpringBootTest`的`webEnvironment`属性来配置我们的运行时环境；我们在这里使用`WebEnvironment.MOCK`,这样容器将在一个模拟的 servlet 环境中运行。

接下来， `@TestPropertySource`注释帮助配置特定于我们的测试的属性文件的位置。注意，用`@TestPropertySource`加载的属性文件将覆盖现有的`application.properties`文件。

`application-integrationtest.properties`包含配置持久存储的详细信息:

```java
spring.datasource.url = jdbc:h2:mem:test
spring.jpa.properties.hibernate.dialect = org.hibernate.dialect.H2Dialect
```

如果我们想对 MySQL 运行集成测试，我们可以在属性文件中更改上面的值。

集成测试的测试用例可能看起来类似于`Controller`层单元测试:

```java
@Test
public void givenEmployees_whenGetEmployees_thenStatus200()
  throws Exception {

    createTestEmployee("bob");

    mvc.perform(get("/api/employees")
      .contentType(MediaType.APPLICATION_JSON))
      .andExpect(status().isOk())
      .andExpect(content()
      .contentTypeCompatibleWith(MediaType.APPLICATION_JSON))
      .andExpect(jsonPath("$[0].name", is("bob")));
}
```

与`Controller`层单元测试的不同之处在于，这里不模拟任何东西，而是执行端到端的场景。

## 5.用`@TestConfiguration`测试配置

正如我们在上一节中看到的，用`@SpringBootTest`注释的测试将引导完整的应用程序上下文，这意味着我们可以`@Autowire`将组件扫描获得的任何 bean 加入到我们的测试中:

```java
@RunWith(SpringRunner.class)
@SpringBootTest
public class EmployeeServiceImplIntegrationTest {

    @Autowired
    private EmployeeService employeeService;

    // class code ...
} 
```

然而，我们可能希望避免引导真实的应用程序上下文，而是使用特殊的测试配置。我们可以通过`@TestConfiguration`注释来实现这一点。有两种使用注释的方法。要么在同一个测试类中的一个静态内部类上，我们希望在这里`@Autowire`bean:

```java
@RunWith(SpringRunner.class)
public class EmployeeServiceImplIntegrationTest {

    @TestConfiguration
    static class EmployeeServiceImplTestContextConfiguration {
        @Bean
        public EmployeeService employeeService() {
            return new EmployeeService() {
                // implement methods
            };
        }
    }

    @Autowired
    private EmployeeService employeeService;
}
```

或者，我们可以创建一个单独的测试配置类:

```java
@TestConfiguration
public class EmployeeServiceImplTestContextConfiguration {

    @Bean
    public EmployeeService employeeService() {
        return new EmployeeService() { 
            // implement methods 
        };
    }
}
```

用`@TestConfiguration`标注的配置类被排除在组件扫描之外，因此我们需要在我们想要`@Autowire`它的每个测试中显式地导入它。我们可以用`@Import`注解来做到这一点:

```java
@RunWith(SpringRunner.class)
@Import(EmployeeServiceImplTestContextConfiguration.class)
public class EmployeeServiceImplIntegrationTest {

    @Autowired
    private EmployeeService employeeService;

    // remaining class code
}
```

## 6。`@MockBean`嘲讽着

我们的`Service`层代码依赖于我们的`Repository:`

```java
@Service
public class EmployeeServiceImpl implements EmployeeService {

    @Autowired
    private EmployeeRepository employeeRepository;

    @Override
    public Employee getEmployeeByName(String name) {
        return employeeRepository.findByName(name);
    }
}
```

然而，为了测试`Service`层，我们不需要知道或者关心持久层是如何实现的。理想情况下，我们应该能够编写和测试我们的`Service`层代码，而无需在我们的完整持久层中布线。

为了实现这一点，我们可以使用 Spring Boot 测试提供的嘲讽支持。

让我们先来看看测试类的框架:

```java
@RunWith(SpringRunner.class)
public class EmployeeServiceImplIntegrationTest {

    @TestConfiguration
    static class EmployeeServiceImplTestContextConfiguration {

        @Bean
        public EmployeeService employeeService() {
            return new EmployeeServiceImpl();
        }
    }

    @Autowired
    private EmployeeService employeeService;

    @MockBean
    private EmployeeRepository employeeRepository;

    // write test cases here
}
```

为了检查`Service`类，我们需要创建一个`Service`类的实例并作为`@Bean`提供，这样我们就可以在我们的测试类中`@Autowire`它。我们可以使用`@TestConfiguration`注释来实现这个配置。

这里另一个有趣的事情是`@MockBean`的使用。它[为`EmployeeRepository`创建一个模拟](/web/20221212193351/https://www.baeldung.com/mockito-mock-methods)，它可以用来绕过对实际`EmployeeRepository`的调用:

```java
@Before
public void setUp() {
    Employee alex = new Employee("alex");

    Mockito.when(employeeRepository.findByName(alex.getName()))
      .thenReturn(alex);
}
```

由于设置已经完成，测试用例将会更简单:

```java
@Test
public void whenValidName_thenEmployeeShouldBeFound() {
    String name = "alex";
    Employee found = employeeService.getEmployeeByName(name);

     assertThat(found.getName())
      .isEqualTo(name);
 }
```

## 7。用`@DataJpaTest` 进行集成测试

我们将使用一个名为`Employee,` 的实体，它有一个`id`和一个`name`属性:

```java
@Entity
@Table(name = "person")
public class Employee {

    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    private Long id;

    @Size(min = 3, max = 20)
    private String name;

    // standard getters and setters, constructors
}
```

这是我们使用 Spring Data JPA 的存储库:

```java
@Repository
public interface EmployeeRepository extends JpaRepository<Employee, Long> {

    public Employee findByName(String name);

}
```

这就是持久层代码。现在让我们开始编写我们的测试类。

首先，让我们创建测试类的框架:

```java
@RunWith(SpringRunner.class)
@DataJpaTest
public class EmployeeRepositoryIntegrationTest {

    @Autowired
    private TestEntityManager entityManager;

    @Autowired
    private EmployeeRepository employeeRepository;

    // write test cases here

}
```

`@RunWith(SpringRunner.class)`在 Spring Boot 测试功能和 JUnit 之间架起了一座桥梁。每当我们在 JUnit 测试中使用任何 Spring Boot 测试特性时，都需要这个注释。

`@DataJpaTest`提供了测试持久层所需的一些标准设置:

*   配置内存数据库 H2
*   设置休眠、弹簧数据和`DataSource`
*   执行`@EntityScan`
*   打开 SQL 日志记录

为了执行数据库操作，我们需要数据库中已经存在的一些记录。要设置这些数据，我们可以使用`TestEntityManager.`

**Spring Boot`TestEntityManager`是标准 JPA `EntityManager`的替代，它提供了编写测试时常用的方法。**

`EmployeeRepository`是我们将要测试的组件。

现在让我们编写我们的第一个测试用例:

```java
@Test
public void whenFindByName_thenReturnEmployee() {
    // given
    Employee alex = new Employee("alex");
    entityManager.persist(alex);
    entityManager.flush();

    // when
    Employee found = employeeRepository.findByName(alex.getName());

    // then
    assertThat(found.getName())
      .isEqualTo(alex.getName());
}
```

在上面的测试中，我们使用`TestEntityManager` 在 DB 中插入一个`Employee`，并通过 find by name API 读取它。

`assertThat(…)`部分来自与 Spring Boot 捆绑在一起的 [Assertj 库](/web/20221212193351/https://www.baeldung.com/introduction-to-assertj)。

## 8。用`@WebMvcTest` 进行单元测试

我们的`Controller`依赖于`Service`层；为了简单起见，我们只包括一个方法:

```java
@RestController
@RequestMapping("/api")
public class EmployeeRestController {

    @Autowired
    private EmployeeService employeeService;

    @GetMapping("/employees")
    public List<Employee> getAllEmployees() {
        return employeeService.getAllEmployees();
    }
}
```

因为我们只关注于`Controller`代码，所以很自然地在我们的单元测试中模仿`Service`层代码:

```java
@RunWith(SpringRunner.class)
@WebMvcTest(EmployeeRestController.class)
public class EmployeeRestControllerIntegrationTest {

    @Autowired
    private MockMvc mvc;

    @MockBean
    private EmployeeService service;

    // write test cases here
}
```

**为了测试`Controllers`，我们可以使用`@WebMvcTest`。它将为我们的单元测试自动配置 Spring MVC 基础设施。**

在大多数情况下，@ `WebMvcTest`将仅限于引导单个控制器。我们还可以将它与`@MockBean`一起使用，为任何所需的依赖项提供模拟实现。

`@WebMvcTest`还可以自动配置`MockMvc`，这提供了一种强大的方法来轻松测试 MVC 控制器，而无需启动一个完整的 HTTP 服务器。

说到这里，让我们来编写我们的测试用例:

```java
@Test
public void givenEmployees_whenGetEmployees_thenReturnJsonArray()
  throws Exception {

    Employee alex = new Employee("alex");

    List<Employee> allEmployees = Arrays.asList(alex);

    given(service.getAllEmployees()).willReturn(allEmployees);

    mvc.perform(get("/api/employees")
      .contentType(MediaType.APPLICATION_JSON))
      .andExpect(status().isOk())
      .andExpect(jsonPath("$", hasSize(1)))
      .andExpect(jsonPath("$[0].name", is(alex.getName())));
}
```

`get(…)`方法调用可以被对应于 HTTP 动词的其他方法替换，如`put()`、`post()`等。请注意，我们也在请求中设置内容类型。

`MockMvc`是灵活的，我们可以使用它创建任何请求。

## 9.自动配置的测试

Spring Boot 的自动配置注释的一个惊人特性是，它有助于加载部分完整的应用程序和代码库的测试特定层。

除了上面提到的注释，下面列出了一些广泛使用的注释:

*   `@WebF` `luxTest`:我们可以使用`@WebFluxTest`注释来测试 Spring WebFlux 控制器。它通常与`@MockBean`一起使用，为所需的依赖关系提供模拟实现。

*   `@JdbcTest` : `W` e 可以使用`@JdbcTest`注释来测试 JPA 应用程序，但是它只适用于那些只需要一个`DataSource.`注释来配置内存嵌入式数据库和一个`JdbcTemplate.`的测试

*   `@JooqTest`:为了测试 jOOQ 相关的测试，我们可以使用`@JooqTest`注释，它配置了一个 DSLContext。

*   `@DataMongoTest`:为了测试 MongoDB 应用程序，`@DataMongoTest`是一个有用的注释。默认情况下，如果驱动程序通过依赖关系可用，它会配置一个内存中的嵌入式 MongoDB，配置一个`MongoTemplate,`扫描用于`@Document`类，并配置 Spring 数据 MongoDB 存储库。

*   使测试 Redis 应用程序变得更加容易。默认情况下，它扫描`@RedisHash`类并配置 Spring Data Redis 存储库。
*   `@DataLdapTest`配置一个内存嵌入式`LDAP`(如果可用)，配置一个`LdapTemplate`，扫描`@Entry`类，默认配置 Spring 数据`LDAP`库。

*   `@RestClientTest`:我们通常使用`@RestClientTest`注释来测试 REST 客户端。它自动配置不同的依赖项，如 Jackson、GSON 和 Jsonb 支持；配置一个`RestTemplateBuilder;`，默认添加对`MockRestServiceServer`的支持。
*   `@JsonTest`:仅用测试 JSON 序列化所需的 beans 来初始化 Spring 应用程序上下文。

您可以在我们的文章[优化 Spring 集成测试](/web/20221212193351/https://www.baeldung.com/spring-tests)中阅读更多关于这些注释以及如何进一步优化集成测试的内容。

## 10。结论

在本文中，我们深入研究了 Spring Boot 的测试支持，并展示了如何高效地编写单元测试。

本文的完整源代码可以在 GitHub 上找到[。源代码包含更多的例子和各种测试用例。](https://web.archive.org/web/20221212193351/https://github.com/eugenp/tutorials/tree/master/spring-boot-modules/spring-boot-testing)

如果你想继续学习关于测试的知识，我们在 JUnit 5 中有关于[集成测试](/web/20221212193351/https://www.baeldung.com/integration-testing-in-spring)、[优化 Spring 集成测试](/web/20221212193351/https://www.baeldung.com/spring-tests)和[单元测试的单独文章。](/web/20221212193351/https://www.baeldung.com/junit-5)