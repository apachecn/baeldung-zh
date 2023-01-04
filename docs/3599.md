# Spring MVC 的 JSON 参数

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-mvc-send-json-parameters>

## 1.概观

在这个简短的教程中，我们将仔细看看如何在 Spring MVC 中使用 JSON 参数。

首先，我们将从 JSON 参数的一些背景知识开始。然后，我们将深入兔子洞，看看如何在 POST 和 GET 请求中发送 JSON 参数。

## 2.Spring MVC 中的 JSON 参数

使用 [JSON](/web/20220714004847/https://www.baeldung.com/java-org-json) 发送或接收数据是 web 开发人员的常见做法。JSON 字符串的层次结构为表示 HTTP 请求参数提供了一种更紧凑、更易读的方式。

默认情况下，Spring MVC 为简单数据类型(如`String`)提供了现成的数据绑定。为此，它使用了一个内置属性编辑器的[列表](https://web.archive.org/web/20220714004847/https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/beans/propertyeditors/package-summary.html)。

然而，在现实项目中，我们可能希望绑定更复杂的数据类型。例如，将一个 JSON 参数映射到一个模型对象可能会很方便。

## 3.在 POST 中发送 JSON 数据

Spring 提供了一种通过 POST 请求发送 JSON 数据的简单方法。[内置的`@RequestBody` 注释](/web/20220714004847/https://www.baeldung.com/spring-request-response-body#@requestbody)可以自动将封装在请求体中的 JSON 数据反序列化为特定的模型对象。

一般来说，我们不必自己解析请求体。我们可以使用[杰克森图书馆](/web/20220714004847/https://www.baeldung.com/jackson)来帮我们完成所有繁重的工作。

现在，让我们看看如何在 Spring MVC 中通过 POST 请求发送 JSON 数据。

首先，我们需要创建一个模型对象来表示传递的 JSON 数据。例如，考虑一下`Product`类:

```
public class Product {

    private int id;
    private String name;
    private double price;

    // default constructor + getters + setters

}
```

其次，让我们定义一个接受 POST 请求的 Spring 处理程序方法:

```
@PostMapping("/create")
@ResponseBody
public Product createProduct(@RequestBody Product product) {
    // custom logic
    return product;
}
```

正如我们所见，**用`@RequestBody`标注`product` 参数就足以绑定从客户端**发送的 JSON 数据。

现在，我们可以使用 [cURL](/web/20220714004847/https://www.baeldung.com/curl-rest) 来测试我们的 POST 请求:

```
curl -i \
-H "Accept: application/json" \
-H "Content-Type: application/json" \
-X POST --data \
  '{"id": 1,"name": "Asus Zenbook","price": 800}' "http://localhost:8080/spring-mvc-basics-4/products/create"
```

## 4.在 GET 中发送 JSON 参数

Spring MVC 提供 [`@RequestParam`](/web/20220714004847/https://www.baeldung.com/spring-request-param) 从 GET 请求中提取查询参数。然而，与`@RequestBody,` 不同的是， `@RequestParam` 注释只支持简单的数据类型，比如`int`和`String`。

因此，要发送 JSON，我们需要将 JSON 参数定义为一个简单的字符串。

这里的大问题是:我们如何将我们的 JSON 参数(它是一个`String`)转换成一个`Product`类的对象？

答案很简单！由 Jackson 库提供的 [`ObjectMapper`类](/web/20220714004847/https://www.baeldung.com/jackson-object-mapper-tutorial)提供了一种将 JSON 字符串转换成 Java 对象的灵活方法。

现在，让我们看看如何在 Spring MVC 中通过 GET 请求发送 JSON 参数。首先，我们需要在控制器中创建另一个处理程序方法来处理 GET 请求:

```
@GetMapping("/get")
@ResponseBody
public Product getProduct(@RequestParam String product) throws JsonMappingException, JsonProcessingException {
    Product prod = objectMapper.readValue(product, Product.class);
    return prod;
}
```

如上所示，`readValue()`方法允许将 JSON 参数`product`直接反序列化到`Product`类的实例中。

注意，我们将 JSON 查询参数定义为一个`String`对象。现在，如果我们想传递一个`Product`对象，就像我们在使用`@RequestBody`时做的那样，该怎么办呢？

为了回答这个问题，Spring 通过[自定义属性编辑器](/web/20220714004847/https://www.baeldung.com/spring-mvc-custom-property-editor)提供了一个简洁灵活的解决方案。

首先，**我们需要创建一个定制的属性编辑器来封装将作为`String`给出的 JSON 参数转换成`Product `对象**的逻辑:

```
public class ProductEditor extends PropertyEditorSupport {

    private ObjectMapper objectMapper;

    public ProductEditor(ObjectMapper objectMapper) {
        this.objectMapper = objectMapper;
    }

    @Override
    public void setAsText(String text) throws IllegalArgumentException {
        if (StringUtils.isEmpty(text)) {
            setValue(null);
        } else {
            Product prod = new Product();
            try {
                prod = objectMapper.readValue(text, Product.class);
            } catch (JsonProcessingException e) {
                throw new IllegalArgumentException(e);
            }
            setValue(prod);
        }
    }

}
```

接下来，让我们将 JSON 参数绑定到一个`Product`类的对象:

```
@GetMapping("/get2")
@ResponseBody
public Product get2Product(@RequestParam Product product) {
    // custom logic
    return product;
}
```

最后，我们需要添加最后一个缺失的部分。让我们**在我们的弹簧控制器**中注册`ProductEditor`:

```
@InitBinder
public void initBinder(WebDataBinder binder) {
    binder.registerCustomEditor(Product.class, new ProductEditor(objectMapper));
}
```

记住**我们需要对 JSON 参数进行 URL 编码，以确保安全传输**。

因此，与其说:

```
GET /spring-mvc-basics-4/products/get2?product={"id": 1,"name": "Asus Zenbook","price": 800}
```

我们需要发送:

```
GET /spring-mvc-basics-4/products/get2?product=%7B%22id%22%3A%201%2C%22name%22%3A%20%22Asus%20Zenbook%22%2C%22price%22%3A%20800%7D
```

## 5.结论

总而言之，我们看到了如何在 Spring MVC 中使用 JSON。一路上，我们展示了如何在 POST 和 GET 请求中发送 JSON 参数。

与往常一样，GitHub 上的[提供了示例的完整源代码。](https://web.archive.org/web/20220714004847/https://github.com/eugenp/tutorials/tree/master/spring-web-modules/spring-mvc-basics-4)