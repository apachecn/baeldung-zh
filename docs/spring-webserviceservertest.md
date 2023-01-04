# 用@WebServiceServerTest 进行 Spring Web 服务集成测试

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-webserviceservertest>

## 1.介绍

在本文中，我们将看到如何为使用 Spring Boot 构建的 SOAP web 服务编写集成测试。

我们已经知道如何为应用程序类编写单元测试，并且我们已经在 Spring Boot 的[测试教程中介绍了一般的测试概念。所以在这里，我们将关注使用`@WebServiceServerTest` 的**集成测试，仅仅是 web 服务层。**](/web/20221128035531/https://www.baeldung.com/spring-boot-testing)

## 2.测试 Spring Web 服务

在 Spring Web 服务中，端点是服务器端服务实现的关键概念。专门化的`@Endpoint`注释将被注释的类标记为 web 服务端点。重要的是，这些**端点负责接收 XML 请求消息，调用所需的业务逻辑，并将结果作为响应消息**返回。

### 2.1.Spring Web 服务测试支持

为了测试这样的端点，我们可以通过传入所需的参数或模拟来轻松地创建单元测试。然而，主要的缺点是这并没有真正测试通过网络发送的 XML 消息的内容。另一种方法是**创建集成测试来验证消息**的 XML 内容。

Spring Web Services 2.0 引入了对这种端点的集成测试的支持。**提供这种支持的核心类是 [`MockWebServiceClient`](https://web.archive.org/web/20221128035531/https://docs.spring.io/spring-ws/docs/current/api/org/springframework/ws/test/server/MockWebServiceClient.html)** 。它提供了一个流畅的 API，将 XML 消息发送到 Spring 应用程序上下文中配置的适当端点。此外，我们可以设置响应预期，验证响应 XML，并为我们的端点执行完整的集成测试。

然而，这需要启动整个应用程序上下文，这会降低测试的执行速度。这通常是不可取的，尤其是当我们希望为特定的 web 服务端点创建快速且独立的测试时。

### 2.2.Spring Boot `@WebServiceServerTest`

Spring Boot 2.6 用`@WebServiceServerTest`注解扩展了 web 服务测试支持。

我们可以将它用于只关注 web 服务层的**测试，而不是加载整个应用程序上下文**。换句话说，我们可以创建一个只包含所需的`@Endpoint`bean 的测试片，并且我们可以使用`@MockBean`模拟任何依赖关系。

这与 Spring Boot 已经提供的得心应手的[测试切片标注](/web/20221128035531/https://www.baeldung.com/spring-tests#5-using-test-slices)非常相似，如`@WebMvcTest`、`@DataJpaTest,`等各种。

## 3.设置示例项目

### 3.1.属国

由于我们已经详细介绍了一个 [Spring Boot web 服务项目](/web/20221128035531/https://www.baeldung.com/spring-boot-soap-web-service)，这里我们将只包括我们项目所需的额外的测试范围 [spring-ws-test](https://web.archive.org/web/20221128035531/https://search.maven.org/search?q=g:org.springframework.ws%20a:spring-ws-test) 依赖:

```java
<dependency>
    <groupId>org.springframework.ws</groupId>
    <artifactId>spring-ws-test</artifactId>
    <version>3.1.3</version>
    <scope>test</scope>
</dependency>
```

### 3.2.示例 Web 服务

接下来，让我们创建一个简单的服务，为指定的产品 id 返回一些产品数据:

```java
@Endpoint
public class ProductEndpoint {

    @Autowired
    private ProductRepository productRepository;

    @ResponsePayload
    public GetProductResponse getProduct(@RequestPayload GetProductRequest request) {
        GetProductResponse response = new GetProductResponse();
        response.setProduct(productRepository.findProduct(request.getId()));
        return response;
    }
}
```

在这里，我们用`@Endpoint,`对`ProductEndpoint`组件进行了注释，注册它来处理适当的 XML 请求。

`getProduct`方法接收请求对象，并在返回响应之前从存储库中获取产品数据。存储库的细节在这里并不重要。在我们的例子中，我们可以使用一个简单的内存实现来保持应用程序的简单，并专注于我们的测试策略。

## 4.端点测试

最后，我们可以创建一个测试片，并在 web 服务层验证 XML 消息的正确处理:

```java
@WebServiceServerTest
class ProductEndpointIntegrationTest {

    @Autowired
    private MockWebServiceClient client;

    @MockBean
    private ProductRepository productRepository;

    @Test
    void givenXmlRequest_whenServiceInvoked_thenValidResponse() throws IOException {
        Product product = createProduct();
        when(productRepository.findProduct("1")).thenReturn(product);

        StringSource request = new StringSource(
          "<bd:getProductRequest xmlns:bd='http://baeldung.com/spring-boot-web-service'>" + 
            "<bd:id>1</bd:id>" + 
          "</bd:getProductRequest>"
        );

        StringSource expectedResponse = new StringSource(
          "<bd:getProductResponse xmlns:bd='http://baeldung.com/spring-boot-web-service'>" + 
            "<bd:product>" + 
              "<bd:id>1</bd:id>" + 
              "<bd:name>Product 1</bd:name>" + 
            "</bd:product>" + 
          "</bd:getProductResponse>"
        );

        client.sendRequest(withPayload(request))
          .andExpect(noFault())
          .andExpect(validPayload(new ClassPathResource("webservice/products.xsd")))
          .andExpect(payload(expectedResponse))
          .andExpect(xpath("/bd:getProductResponse/bd:product[1]/bd:name", NAMESPACE_MAPPING)
            .evaluatesTo("Product 1"));
    }
}
```

这里，我们只在应用程序中为集成测试配置了用`@Endpoint`标注的 beans。换句话说，**这个测试片创建了一个简化的应用程序上下文**。这有助于我们构建有针对性的快速集成测试，而没有与重复加载整个应用程序上下文相关的性能损失。

重要的是，**这个注释还配置了一个`MockWebServiceClient`以及其他相关的自动配置**。因此，我们可以将这个客户端连接到我们的测试中，并使用它来发送`getProductRequest` XML 请求，随后是各种流畅的期望。

预期验证响应 XML 根据给定的 XSD 模式进行验证，并且它与预期的 XML 响应相匹配。我们还可以使用 XPath 表达式来评估和比较来自响应 XML 的各种值。

### 4.1.端点协作者

在我们的例子中，我们使用了 [`@MockBean`来模仿](/web/20221128035531/https://www.baeldung.com/spring-boot-testing#mocking-with-mockbean)我们的`ProductEndpoint`中所需的存储库。如果没有这个模拟，应用程序上下文将无法启动，因为完全自动配置被禁用。换句话说，**测试框架在测试执行**之前不配置任何`@Component`、`@Service,`或`@Repository`bean。

然而，如果我们确实需要实际的合作者而不是模仿者，那么我们可以使用`@Import`来声明这些。Spring 将寻找这些类，然后根据需要将它们连接到端点。

### 4.2.加载整个上下文

如前所述，`@WebServiceServerTest`不会加载整个应用程序上下文。如果我们确实需要为测试加载整个应用程序上下文，那么我们应该考虑将`@SpringBootTest`与`@AutoConfigureMockWebServiceClient.` 结合使用，然后我们可以以类似的方式使用这个客户端来发送请求并验证响应，如前所示。

## 5.结论

在本文中，我们研究了 Spring Boot 引入的`@WebServiceServerTest`注释。

最初，我们讨论了 web 服务应用程序中的 Spring Boot 测试支持。接下来，我们看到了如何使用这个注释为 web 服务层创建一个测试片，这有助于构建快速且集中的集成测试。

和往常一样，完整的源代码可以在 GitHub 的[上找到。](https://web.archive.org/web/20221128035531/https://github.com/eugenp/tutorials/tree/master/spring-boot-modules/spring-boot-testing-2)