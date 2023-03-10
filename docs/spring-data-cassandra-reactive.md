# 反应式 Cassandra 的春季数据

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-data-cassandra-reactive>

## 1。简介

在本教程中，我们将学习如何使用 Spring Data Cassandra 的反应式数据访问特性。

特别是，这是 Spring Data Cassandra 文章系列的第三篇文章。在这个例子中，我们将使用 REST API 公开一个 Cassandra 数据库。

我们可以在该系列的第[篇第](/web/20220525142226/https://www.baeldung.com/spring-data-cassandra-tutorial)篇和第[篇第二](/web/20220525142226/https://www.baeldung.com/spring-data-cassandratemplate-cqltemplate)篇中读到更多关于 Spring Data Cassandra 的内容。

## 延伸阅读:

## [使用 Cassandra、Astra 和 Stargate 构建仪表板](/web/20220525142226/https://www.baeldung.com/cassandra-astra-stargate-dashboard)

了解如何使用 DataStax Astra 构建仪表板，这是一种由 Apache Cassandra 和 Stargate APIs 支持的数据库即服务。[阅读更多](/web/20220525142226/https://www.baeldung.com/cassandra-astra-stargate-dashboard) →

## [用 Cassandra、Astra、REST&graph QL——记录状态更新](/web/20220525142226/https://www.baeldung.com/cassandra-astra-rest-dashboard-updates)

用 Cassandra 存储时序数据的例子。[阅读更多信息](/web/20220525142226/https://www.baeldung.com/cassandra-astra-rest-dashboard-updates) →

## [使用 Cassandra、Astra 和 CQL 构建一个仪表盘——绘制事件数据](/web/20220525142226/https://www.baeldung.com/cassandra-astra-rest-dashboard-map)

了解如何根据 Astra 数据库中存储的数据在交互式地图上显示事件。[阅读更多](/web/20220525142226/https://www.baeldung.com/cassandra-astra-rest-dashboard-map) →

## 2。Maven 依赖关系

事实上，Spring Data Cassandra 支持 Project Reactor 和 RxJava reactive 类型。为了演示，我们将在本教程中使用项目反应器的反应型`Flux`和`Mono `。

首先，让我们添加教程所需的依赖项:

```java
<dependency>
    <groupId>org.springframework.data</groupId>
    <artifactId>spring-data-cassandra</artifactId>
    <version>2.1.2.RELEASE</version>
</dependency>
<dependency>
    <groupId>io.projectreactor</groupId>
    <artifactId>reactor-core</artifactId>
</dependency>
```

最新版本的`spring-data-cassandra `可以在[这里](https://web.archive.org/web/20220525142226/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22org.springframework.data%22%20AND%20a%3A%22spring-data-cassandra%22)找到。

现在，我们将通过 REST API 从数据库中公开`SELECT`操作。因此，让我们也为`RestController`添加依赖关系:

```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

## 3。实施我们的应用

因为我们将持久化数据，所以让我们首先定义我们的实体对象`:`

```java
@Table
public class Employee {
    @PrimaryKey
    private int id;
    private String name;
    private String address;
    private String email;
    private int age;
}
```

接下来，是时候创建一个从`ReactiveCassandraRepository.` 扩展而来的`EmployeeRepository `了，需要注意的是**这个接口启用了对反应型**的支持；

```java
public interface EmployeeRepository extends ReactiveCassandraRepository<Employee, Integer> {
    @AllowFiltering
    Flux<Employee> findByAgeGreaterThan(int age);
}
```

### 3.1。用于 CRUD 操作的 Rest 控制器

为了便于说明，我们将使用一个简单的 Rest 控制器来展示一些基本的`SELECT`操作:

```java
@RestController
@RequestMapping("employee")
public class EmployeeController {

    @Autowired
    EmployeeService employeeService;

    @PostConstruct
    public void saveEmployees() {
        List<Employee> employees = new ArrayList<>();
        employees.add(new Employee(123, "John Doe", "Delaware", "[[email protected]](/web/20220525142226/https://www.baeldung.com/cdn-cgi/l/email-protection)", 31));
        employees.add(new Employee(324, "Adam Smith", "North Carolina", "[[email protected]](/web/20220525142226/https://www.baeldung.com/cdn-cgi/l/email-protection)", 43));
        employees.add(new Employee(355, "Kevin Dunner", "Virginia", "[[email protected]](/web/20220525142226/https://www.baeldung.com/cdn-cgi/l/email-protection)", 24));
        employees.add(new Employee(643, "Mike Lauren", "New York", "[[email protected]](/web/20220525142226/https://www.baeldung.com/cdn-cgi/l/email-protection)", 41));
        employeeService.initializeEmployees(employees);
    }

    @GetMapping("/list")
    public Flux<Employee> getAllEmployees() {
        Flux<Employee> employees = employeeService.getAllEmployees();
        return employees;
    }

    @GetMapping("/{id}")
    public Mono<Employee> getEmployeeById(@PathVariable int id) {
        return employeeService.getEmployeeById(id);
    }

    @GetMapping("/filterByAge/{age}")
    public Flux<Employee> getEmployeesFilterByAge(@PathVariable int age) {
        return employeeService.getEmployeesFilterByAge(age);
    }
}
```

最后再补充一个简单的`EmployeeService`:

```java
@Service
public class EmployeeService {

    @Autowired
    EmployeeRepository employeeRepository;

    public void initializeEmployees(List<Employee> employees) {
        Flux<Employee> savedEmployees = employeeRepository.saveAll(employees);
        savedEmployees.subscribe();
    }

    public Flux<Employee> getAllEmployees() {
        Flux<Employee> employees =  employeeRepository.findAll();
        return employees;
    }

    public Flux<Employee> getEmployeesFilterByAge(int age) {
        return employeeRepository.findByAgeGreaterThan(age);
    }

    public Mono<Employee> getEmployeeById(int id) {
        return employeeRepository.findById(id);
    }
}
```

### 3.2。数据库配置

然后，让我们在`application.properties`中指定用于连接 Cassandra 的密钥空间和端口:

```java
spring.data.cassandra.keyspace-name=practice
spring.data.cassandra.port=9042
spring.data.cassandra.local-datacenter=datacenter1
```

注意:`datacenter1`是默认的数据中心名称。

## 4。测试端点

最后，是时候测试我们的 API 端点了。

### 4.1.人工测试

首先，让我们从数据库中获取雇员记录:

```java
curl localhost:8080/employee/list
```

结果，我们得到了所有的员工:

```java
[
    {
        "id": 324,
        "name": "Adam Smith",
        "address": "North Carolina",
        "email": "[[email protected]](/web/20220525142226/https://www.baeldung.com/cdn-cgi/l/email-protection)",
        "age": 43
    },
    {
        "id": 123,
        "name": "John Doe",
        "address": "Delaware",
        "email": "[[email protected]](/web/20220525142226/https://www.baeldung.com/cdn-cgi/l/email-protection)",
        "age": 31
    },
    {
        "id": 355,
        "name": "Kevin Dunner",
        "address": "Virginia",
        "email": "[[email protected]](/web/20220525142226/https://www.baeldung.com/cdn-cgi/l/email-protection)",
        "age": 24
    },
    {
        "id": 643,
        "name": "Mike Lauren",
        "address": "New York",
        "email": "[[email protected]](/web/20220525142226/https://www.baeldung.com/cdn-cgi/l/email-protection)",
       "age": 41
    }
]
```

接下来，让我们试着通过 id 找到一个特定的员工:

```java
curl localhost:8080/employee/643
```

结果，迈克·劳伦先生回来了:

```java
{
    "id": 643,
    "name": "Mike Lauren",
    "address": "New York",
    "email": "[[email protected]](/web/20220525142226/https://www.baeldung.com/cdn-cgi/l/email-protection)",
    "age": 41
}
```

最后，让我们看看我们的年龄过滤器是否有效:

```java
curl localhost:8080/employee/filterByAge/35
```

不出所料，我们得到了所有年龄大于 35 岁的员工:

```java
[
    {
        "id": 324,
        "name": "Adam Smith",
        "address": "North Carolina",
        "email": "[[email protected]](/web/20220525142226/https://www.baeldung.com/cdn-cgi/l/email-protection)",
        "age": 43
    },
    {
        "id": 643,
        "name": "Mike Lauren",
        "address": "New York",
        "email": "[[email protected]](/web/20220525142226/https://www.baeldung.com/cdn-cgi/l/email-protection)",
        "age": 41
    }
]
```

### 4.2.集成测试

此外，让我们通过编写测试用例来测试相同的功能:

```java
@RunWith(SpringRunner.class)
@SpringBootTest
public class ReactiveEmployeeRepositoryIntegrationTest {

    @Autowired
    EmployeeRepository repository;

    @Before
    public void setUp() {
        Flux<Employee> deleteAndInsert = repository.deleteAll()
          .thenMany(repository.saveAll(Flux.just(
            new Employee(111, "John Doe", "Delaware", "[[email protected]](/web/20220525142226/https://www.baeldung.com/cdn-cgi/l/email-protection)", 31),
            new Employee(222, "Adam Smith", "North Carolina", "[[email protected]](/web/20220525142226/https://www.baeldung.com/cdn-cgi/l/email-protection)", 43),
            new Employee(333, "Kevin Dunner", "Virginia", "[[email protected]](/web/20220525142226/https://www.baeldung.com/cdn-cgi/l/email-protection)", 24),
            new Employee(444, "Mike Lauren", "New York", "[[email protected]](/web/20220525142226/https://www.baeldung.com/cdn-cgi/l/email-protection)", 41))));

        StepVerifier
          .create(deleteAndInsert)
          .expectNextCount(4)
          .verifyComplete();
    }

    @Test
    public void givenRecordsAreInserted_whenDbIsQueried_thenShouldIncludeNewRecords() {
        Mono<Long> saveAndCount = repository.count()
          .doOnNext(System.out::println)
          .thenMany(repository
            .saveAll(Flux.just(
            new Employee(325, "Kim Jones", "Florida", "[[email protected]](/web/20220525142226/https://www.baeldung.com/cdn-cgi/l/email-protection)", 42),
            new Employee(654, "Tom Moody", "New Hampshire", "[[email protected]](/web/20220525142226/https://www.baeldung.com/cdn-cgi/l/email-protection)", 44))))
          .last()
          .flatMap(v -> repository.count())
          .doOnNext(System.out::println);

        StepVerifier
          .create(saveAndCount)
          .expectNext(6L)
          .verifyComplete();
    }

    @Test
    public void givenAgeForFilter_whenDbIsQueried_thenShouldReturnFilteredRecords() {
        StepVerifier
          .create(repository.findByAgeGreaterThan(35))
          .expectNextCount(2)
          .verifyComplete();
    }
}
```

## 5。结论

总之，我们通过 Spring Data Cassandra 学习了如何使用反应类型来构建非阻塞应用程序。

像往常一样，在 GitHub 上查看本教程[的源代码。](https://web.archive.org/web/20220525142226/https://github.com/eugenp/tutorials/tree/master/persistence-modules/spring-data-cassandra-reactive)