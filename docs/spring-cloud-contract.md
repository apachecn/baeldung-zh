# Spring Cloud 合同简介

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-cloud-contract>

## 1。简介

**[春云契约](https://web.archive.org/web/20220926190605/https://cloud.spring.io/spring-cloud-contract/)简单来说就是帮助我们编写[消费者驱动契约(CDC)](https://web.archive.org/web/20220926190605/https://martinfowler.com/articles/consumerDrivenContracts.html) 的项目。**

这确保了分布式系统中`Producer`和`Consumer`之间的契约——对于基于 HTTP 和基于消息的交互都是如此。

在这篇简短的文章中，我们将探索通过 HTTP 交互为 Spring Cloud 契约编写生产者和消费者端测试用例。

## 2。生产者-服务器端

我们将编写一个生产者端 CDC，以`EvenOddController` 的形式——它只是告诉我们`number`参数是偶数还是奇数:

```java
@RestController
public class EvenOddController {

    @GetMapping("/validate/prime-number")
    public String isNumberPrime(@RequestParam("number") Integer number) {
        return Integer.parseInt(number) % 2 == 0 ? "Even" : "Odd";
    }
}
```

### 2.1。Maven 依赖关系

对于我们的生产者来说，我们需要 [`spring-cloud-starter-contract-verifier`](https://web.archive.org/web/20220926190605/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.springframework.cloud%22%20AND%20a%3A%22spring-cloud-starter-contract-verifier%22) 的依赖关系:

```java
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-contract-verifier</artifactId>
    <version>2.1.1.RELEASE</version>
    <scope>test</scope>
</dependency>
```

我们需要用我们的基本测试类的名称来配置 [`spring-cloud-contract-maven-plugin`](https://web.archive.org/web/20220926190605/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.springframework.cloud%22%20AND%20a%3A%22spring-cloud-contract-maven-plugin%22) ，我们将在下一节描述它:

```java
<plugin>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-contract-maven-plugin</artifactId>
    <version>2.1.1.RELEASE</version>
    <extensions>true</extensions>
    <configuration>
        <baseClassForTests>
            com.baeldung.spring.cloud.springcloudcontractproducer.BaseTestClass
        </baseClassForTests>
    </configuration>
</plugin>
```

### 2.2。生产者端设置

我们需要在加载 Spring 上下文的测试包中添加一个基类:

```java
@RunWith(SpringRunner.class)
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.MOCK)
@DirtiesContext
@AutoConfigureMessageVerifier
public class BaseTestClass {

    @Autowired
    private EvenOddController evenOddController;

    @Before
    public void setup() {
        StandaloneMockMvcBuilder standaloneMockMvcBuilder 
          = MockMvcBuilders.standaloneSetup(evenOddController);
        RestAssuredMockMvc.standaloneSetup(standaloneMockMvcBuilder);
    }
}
```

**在`/src/test/resources/contracts/` 包中，我们将添加测试存根**，比如文件`shouldReturnEvenWhenRequestParamIsEven.groovy`中的这个:

```java
import org.springframework.cloud.contract.spec.Contract
Contract.make {
    description "should return even when number input is even"
    request{
        method GET()
        url("/validate/prime-number") {
            queryParameters {
                parameter("number", "2")
            }
        }
    }
    response {
        body("Even")
        status 200
    }
} 
```

当我们运行构建时，**插件自动生成一个名为`ContractVerifierTest`的测试类，它扩展了我们的`BaseTestClass`** ，并将其放入`/target/generated-test-sources/contracts/`。

测试方法的名称来源于前缀“`validate_”`”和 Groovy 测试存根的名称。对于上面的 Groovy 文件，生成的方法名将是`“validate_shouldReturnEvenWhenRequestParamIsEven”`。

让我们来看看这个自动生成的测试类:

```java
public class ContractVerifierTest extends BaseTestClass {

@Test
public void validate_shouldReturnEvenWhenRequestParamIsEven() throws Exception {
    // given:
    MockMvcRequestSpecification request = given();

    // when:
    ResponseOptions response = given().spec(request)
      .queryParam("number","2")
      .get("/validate/prime-number");

    // then:
    assertThat(response.statusCode()).isEqualTo(200);

    // and:
    String responseBody = response.getBody().asString();
    assertThat(responseBody).isEqualTo("Even");
} 
```

构建还将在我们的本地 Maven 存储库中添加存根 jar，以便我们的消费者可以使用它。

存根将出现在`stubs/mapping/`下的输出文件夹中。

## 3。消费者-客户端

**我们 CDC 的消费端会通过 HTTP 交互来消费生产者端**生成的存根来维护契约，所以**生产者端的任何改变都会破坏契约**。

我们将添加`BasicMathController,` ,它将发出一个 HTTP 请求，从生成的存根中获取响应:

```java
@RestController
public class BasicMathController {

    @Autowired
    private RestTemplate restTemplate;

    @GetMapping("/calculate")
    public String checkOddAndEven(@RequestParam("number") Integer number) {
        HttpHeaders httpHeaders = new HttpHeaders();
        httpHeaders.add("Content-Type", "application/json");

        ResponseEntity<String> responseEntity = restTemplate.exchange(
          "http://localhost:8090/validate/prime-number?number=" + number,
          HttpMethod.GET,
          new HttpEntity<>(httpHeaders),
          String.class);

        return responseEntity.getBody();
    }
}
```

### 3.1。美芬依赖

对于我们的消费者，我们需要添加 [`spring-cloud-contract-wiremock`](https://web.archive.org/web/20220926190605/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.springframework.cloud%22%20AND%20a%3A%22spring-cloud-contract-wiremock%22) 和 [`spring-cloud-contract-stub-runner`](https://web.archive.org/web/20220926190605/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.springframework.cloud%22%20AND%20a%3A%22spring-cloud-contract-stub-runner%22) 依赖项:

```java
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-contract-wiremock</artifactId>
    <version>2.1.1.RELEASE</version>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-contract-stub-runner</artifactId>
    <version>2.1.1.RELEASE</version>
    <scope>test</scope>
</dependency> 
```

### 3.2。消费者端设置

现在是配置存根控件的时候了，它将通知我们的用户本地 Maven 存储库中的可用存根:

```java
@RunWith(SpringRunner.class)
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.MOCK)
@AutoConfigureMockMvc
@AutoConfigureJsonTesters
@AutoConfigureStubRunner(
  stubsMode = StubRunnerProperties.StubsMode.LOCAL,
  ids = "com.baeldung.spring.cloud:spring-cloud-contract-producer:+:stubs:8090")
public class BasicMathControllerIntegrationTest {

    @Autowired
    private MockMvc mockMvc;

    @Test
    public void given_WhenPassEvenNumberInQueryParam_ThenReturnEven()
      throws Exception {

        mockMvc.perform(MockMvcRequestBuilders.get("/calculate?number=2")
          .contentType(MediaType.APPLICATION_JSON))
          .andExpect(status().isOk())
          .andExpect(content().string("Even"));
    }
}
```

注意，`@AutoConfigureStubRunner`注释的`ids`属性指定:

*   `com.baeldung.spring.cloud` —我们神器的`groupId`
*   `spring-cloud-contract-producer` —生产者存根罐的`artifactId`
*   `8090` —运行生成的存根控件的端口

## 4。当合同违约时

如果我们在生产者方面做出任何直接影响合同的改变，而没有更新消费者方面，这可能导致合同失败。

例如，假设我们要在我们的生产者端将`EvenOddController`请求 URI 改为`/validate/change/prime-number`。

如果我们没有通知我们的消费者这个变化，消费者仍然会发送它的请求到`/validate/prime-number` URI，并且消费者端测试用例将抛出`org.springframework.web.client.HttpClientErrorException: 404 Not Found`。

## 5。总结

我们已经看到了 Spring Cloud Contract 如何帮助我们维护服务消费者和生产者之间的合同，这样我们就可以推出新代码，而不用担心违反合同。

和往常一样，本教程的完整实现可以在 GitHub 上找到[。](https://web.archive.org/web/20220926190605/https://github.com/eugenp/tutorials/tree/master/spring-cloud-modules/spring-cloud-contract)