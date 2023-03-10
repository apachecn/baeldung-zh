# Spring RestTemplate 异常:“变量不足，无法扩展”

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-not-enough-variables-available>

## 1.概观

在这个简短的教程中，我们将仔细看看 Spring 的 [`RestTemplate`](/web/20220525141537/https://www.baeldung.com/rest-template) 异常`IllegalArgumentException`:没有足够的变量可供扩展。

首先，我们将详细讨论这个异常背后的主要原因。然后，我们将展示如何产生它，最后，如何解决它。

## 2.原因

简而言之，当我们试图在 GET 请求中发送 JSON 数据时，异常通常会发生**。**

简单地说，`RestTemplate` 提供了 [`getForObject`](/web/20220525141537/https://www.baeldung.com/rest-template#2-retrieving-pojo-instead-of-json) 方法，通过在指定的 URL 上发出 get 请求来获得一个表示。

异常的主要原因是 **`RestTemplate` 认为封装在花括号中的 JSON 数据是 URI 变量**的占位符。

由于我们没有为预期的 URI 变量提供任何值，`getForObject` 方法抛出异常。

例如，尝试将`{“name”:”HP EliteBook”}`作为值发送:

```java
String url = "http://products.api.com/get?key=a123456789z&criterion;={\"name\":\"HP EliteBook\"}";
Product product = restTemplate.getForObject(url, Product.class);
```

只会导致`RestTemplate`抛出异常:

```java
java.lang.IllegalArgumentException: Not enough variable values available to expand 'name'
```

## 3.示例应用程序

现在，让我们看一个如何使用`RestTemplate`产生这个`IllegalArgumentException`的例子。

为了简单起见，我们将为产品管理创建一个基本的 [REST API](/web/20220525141537/https://www.baeldung.com/rest-with-spring-series) ，使用一个 GET 端点。

首先，让我们创建我们的模型类`Product`:

```java
public class Product {

    private int id;
    private String name;
    private double price;

    // default constructor + all args constructor + getters + setters 
}
```

接下来，我们将定义一个 spring 控制器来封装 REST API 的逻辑:

```java
@RestController
@RequestMapping("/api")
public class ProductApi {

    private List<Product> productList = new ArrayList<>(Arrays.asList(
      new Product(1, "Acer Aspire 5", 437), 
      new Product(2, "ASUS VivoBook", 650), 
      new Product(3, "Lenovo Legion", 990)
    ));

    @GetMapping("/get")
    public Product get(@RequestParam String criterion) throws JsonMappingException, JsonProcessingException {
        ObjectMapper objectMapper = new ObjectMapper();
        Criterion crt = objectMapper.readValue(criterion, Criterion.class);
        if (crt.getProp().equals("name")) {
            return findByName(crt.getValue());
        }

        // Search by other properties (id,price)

        return null;
    }

    private Product findByName(String name) {
        for (Product product : this.productList) {
            if (product.getName().equals(name)) {
                return product;
            }
        }
        return null;
    }

    // Other methods
}
```

## 4.示例应用说明

处理程序方法`get()` 的基本思想是根据特定的标准`.`检索产品对象

标准可以表示为一个 JSON 字符串，带有两个键:`prop`和`value`。

`prop` 键引用一个产品属性，所以它可以是一个 id、一个名称或一个价格。

如上所示，标准作为字符串参数传递给处理程序方法。我们使用 [`ObjectMapper`](/web/20220525141537/https://www.baeldung.com/jackson-object-mapper-tutorial) 类将我们的 [JSON 字符串转换为`Criterion`的对象](/web/20220525141537/https://www.baeldung.com/spring-mvc-send-json-parameters#send-json-parameter-in-get)。

这是我们的`Criterion`类的样子:

```java
public class Criterion {

    private String prop;
    private String value;

    // default constructor + getters + setters
}
```

最后，让我们尝试向映射到处理程序方法`get()`的 URL 发送一个 GET 请求。

```java
@RunWith(SpringRunner.class)
@SpringBootTest(classes = { RestTemplate.class, RestTemplateExceptionApplication.class })
public class RestTemplateExceptionLiveTest {

    @Autowired
    RestTemplate restTemplate;

    @Test(expected = IllegalArgumentException.class)
    public void givenGetUrl_whenJsonIsPassed_thenThrowException() {
        String url = "http://localhost:8080/spring-rest/api/get?criterion={\"prop\":\"name\",\"value\":\"ASUS VivoBook\"}";
        Product product = restTemplate.getForObject(url, Product.class);
    }
}
```

事实上，单元测试抛出了`IllegalArgumentException`，因为我们试图将`{“prop”:”name”,”value”:”ASUS VivoBook”}`作为 URL 的一部分传递。

## 5.解决方案

根据经验，我们应该总是使用 POST 请求来发送 JSON 数据。

然而，尽管不推荐，使用 GET 的一个可能的解决方案是**定义一个包含我们的标准的`String`对象，并在 URL** 中提供一个真实的 URI 变量。

```java
@Test
public void givenGetUrl_whenJsonIsPassed_thenGetProduct() {
    String criterion = "{\"prop\":\"name\",\"value\":\"ASUS VivoBook\"}";
    String url = "http://localhost:8080/spring-rest/api/get?criterion={criterion}";
    Product product = restTemplate.getForObject(url, Product.class, criterion);

    assertEquals(product.getPrice(), 650, 0);
}
```

让我们看看另一个使用`[UriComponentsBuilder](/web/20220525141537/https://www.baeldung.com/spring-uricomponentsbuilder)`类的解决方案:

```java
@Test
public void givenGetUrl_whenJsonIsPassed_thenReturnProduct() {
    String criterion = "{\"prop\":\"name\",\"value\":\"Acer Aspire 5\"}";
    String url = "http://localhost:8080/spring-rest/api/get";

    UriComponentsBuilder builder = UriComponentsBuilder.fromUriString(url).queryParam("criterion", criterion);
    Product product = restTemplate.getForObject(builder.build().toUri(), Product.class);

    assertEquals(product.getId(), 1, 0);
}
```

正如我们所见，在将查询参数`criterion` 传递给`getForObject`方法`.`之前，我们使用了`UriComponentsBuilder` 类来构建我们的 URI

## 6.结论

在这篇简短的文章中，我们讨论了是什么原因导致`RestTemplate`抛出`IllegalArgumentException: “`没有足够的变量来扩展”。

在这个过程中，我们通过一个实例展示了如何产生异常并解决它。

与往常一样，GitHub 上的[提供了示例的完整源代码。](https://web.archive.org/web/20220525141537/https://github.com/eugenp/tutorials/tree/master/spring-web-modules/spring-resttemplate-2)