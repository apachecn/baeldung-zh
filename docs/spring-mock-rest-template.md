# 在 Spring 中模仿 RestTemplate

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-mock-rest-template>

## 1.介绍

我们经常发现自己的应用程序执行某种 web 请求。当测试这种行为时，我们有一些关于 Spring 应用的选项。

在这个快速教程中，我们将会看到几种模仿只通过 rest templateT5 执行的调用的方法。

我们将从测试一个流行的模仿库 Mockito 开始。然后我们将使用 Spring Test，它为我们提供了创建模拟服务器来定义服务器交互的机制。

## 2.使用`Mockito`

我们可以用 Mockito 来嘲笑所有的人。使用这种方法，测试我们的服务将会像任何其他涉及模仿的[测试一样简单。](/web/20221128042755/https://www.baeldung.com/mockito-annotations)

假设我们有一个简单的`EmployeeService`类，它通过 HTTP:

```java
@Service
public class EmployeeService {

    @Autowired
    private RestTemplate restTemplate;

    public Employee getEmployee(String id) {
	ResponseEntity resp = 
          restTemplate.getForEntity("http://localhost:8080/employee/" + id, Employee.class);

	return resp.getStatusCode() == HttpStatus.OK ? resp.getBody() : null;
    }
} 
```

现在让我们为前面的代码实现我们的测试:

```java
@ExtendWith(MockitoExtension.class)
public class EmployeeServiceTest {

    @Mock
    private RestTemplate restTemplate;

    @InjectMocks
    private EmployeeService empService = new EmployeeService();

    @Test
    public void givenMockingIsDoneByMockito_whenGetIsCalled_shouldReturnMockedObject() {
        Employee emp = new Employee(“E001”, "Eric Simmons");
        Mockito
          .when(restTemplate.getForEntity(
            “http://localhost:8080/employee/E001”, Employee.class))
          .thenReturn(new ResponseEntity(emp, HttpStatus.OK));

        Employee employee = empService.getEmployee(id);
        Assertions.assertEquals(emp, employee);
    }
}
```

在上面的 JUnit 测试类中，我们首先要求 Mockito 使用`@Mock`注释创建一个虚拟的`RestTemplate`实例。

然后我们用`@InjectMocks`注释了 *EmployeeService* 实例，将虚拟实例注入其中。

最后，在测试方法中，我们使用 [Mockito 的 when/then 支持](/web/20221128042755/https://www.baeldung.com/mockito-behavior)来定义我们的 mock 的行为。

## 3.使用弹簧测试

Spring 测试模块包括一个名为 *MockRestServiceServer 的模拟服务器。* **使用这种方法，我们配置服务器在通过我们的`RestTemplate`实例发送特定请求时返回特定的对象。**此外，我们可以*在服务器实例上验证()*是否满足了所有的期望。

`MockRestServiceServer`实际上是通过使用`MockClientHttpRequestFactory`拦截 HTTP API 调用来工作的。基于我们的配置，它创建了一个预期请求和相应响应的列表。当`RestTemplate`实例调用 API 时，它在期望列表中查找请求，并返回相应的响应。

因此，它消除了在任何其他端口运行 HTTP 服务器来发送模拟响应的需要。

让我们使用 *MockRestServiceServer* 为同一个 *getEmployee()* 示例创建一个简单的测试:

```java
@ExtendWith(SpringExtension.class)
@SpringBootTest(classes = SpringTestConfig.class)
public class EmployeeServiceMockRestServiceServerUnitTest {

    @Autowired
    private EmployeeService empService;
    @Autowired
    private RestTemplate restTemplate;

    private MockRestServiceServer mockServer;
    private ObjectMapper mapper = new ObjectMapper();

    @BeforeEach
    public void init() {
        mockServer = MockRestServiceServer.createServer(restTemplate);
    }

    @Test                                                                                          
    public void givenMockingIsDoneByMockRestServiceServer_whenGetIsCalled_thenReturnsMockedObject()() {   
        Employee emp = new Employee("E001", "Eric Simmons");
        mockServer.expect(ExpectedCount.once(), 
          requestTo(new URI("http://localhost:8080/employee/E001")))
          .andExpect(method(HttpMethod.GET))
          .andRespond(withStatus(HttpStatus.OK)
          .contentType(MediaType.APPLICATION_JSON)
          .body(mapper.writeValueAsString(emp))
        );                                   

        Employee employee = empService.getEmployee(id);
        mockServer.verify();
        Assertions.assertEquals(emp, employee);                                                        
    }
} 
```

在前面的代码片段中，我们使用了来自于`MockRestRequestMatchers`和`MockRestResponseCreators`的静态方法，以一种清晰易读的方式定义了对 REST 调用的期望和响应:

```java
import static org.springframework.test.web.client.match.MockRestRequestMatchers.*;      
import static org.springframework.test.web.client.response.MockRestResponseCreators.*;
```

我们应该记住，测试类中的 *RestTemplate* 应该是在`EmployeeService`类中使用的同一个实例。为了确保这一点，我们在 spring 配置中定义了一个 RestTemplate bean，并在测试和实现中自动连接了实例:

```java
@Bean
public RestTemplate restTemplate() {
    return new RestTemplate();
}
```

当我们编写集成测试并且只需要模拟外部 HTTP 调用时，使用`MockRestServiceServer`非常有用。

## 4.结论

在这篇简短的文章中，我们讨论了在编写单元测试时通过 HTTP 模拟外部 REST API 调用的一些有效选项。

上述文章的源代码可以在 GitHub 上的[处获得。](https://web.archive.org/web/20221128042755/https://github.com/eugenp/tutorials/tree/master/spring-web-modules/spring-resttemplate)*