# 使用 JSON 的 RestTemplate Post 请求

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-resttemplate-post-json>

## 1.介绍

在这个快速教程中，我们演示了如何使用 Spring 的 [`RestTemplate`](/web/20221105120039/https://www.baeldung.com/rest-template) 来发出发送 JSON 内容的 POST 请求。

## 延伸阅读:

## [探索 Spring Boot TestRestTemplate](/web/20221105120039/https://www.baeldung.com/spring-boot-testresttemplate)

Learn how to use the new TestRestTemplate in Spring Boot to test a simple API.[Read more](/web/20221105120039/https://www.baeldung.com/spring-boot-testresttemplate) →

## [Spring RestTemplate 错误处理](/web/20221105120039/https://www.baeldung.com/spring-rest-template-error-handling)

Learn how to handle errors with Spring's RestTemplate[Read more](/web/20221105120039/https://www.baeldung.com/spring-rest-template-error-handling) →

## 2.树立榜样

让我们首先添加一个简单的`Person`模型类来表示要发布的数据:

```
public class Person {
    private Integer id;
    private String name;

    // standard constructor, getters, setters
}
```

为了使用`Person`对象，我们将添加一个`PersonService`接口，并用两种方法实现:

```
public interface PersonService {

    public Person saveUpdatePerson(Person person);
    public Person findPersonById(Integer id);
}
```

这些方法的实现将简单地返回一个对象。我们在这里使用该层的虚拟实现，这样我们就可以专注于 web 层。

## 3.REST API 设置

让我们为我们的`Person`类定义一个简单的 REST API:

```
@PostMapping(
  value = "/createPerson", consumes = "application/json", produces = "application/json")
public Person createPerson(@RequestBody Person person) {
    return personService.saveUpdatePerson(person);
}

@PostMapping(
  value = "/updatePerson", consumes = "application/json", produces = "application/json")
public Person updatePerson(@RequestBody Person person, HttpServletResponse response) {
    response.setHeader("Location", ServletUriComponentsBuilder.fromCurrentContextPath()
      .path("/findPerson/" + person.getId()).toUriString());

    return personService.saveUpdatePerson(person);
}
```

记住，我们希望以 JSON 格式发布数据。为此，**我们在`@PostMapping`注释中添加了`consumes`属性，并为这两种方法添加了值“application/JSON”**。

类似地，我们将`produces`属性设置为“application/json ”,告诉 Spring 我们想要 json 格式的响应体。

对于这两种方法，我们用 [`@RequestBody`](/web/20221105120039/https://www.baeldung.com/spring-request-response-body) 注释来注释`person`参数。这将告诉 Spring，`person`对象将被绑定到`HTTP`请求的主体。

最后，两种方法都返回一个将被绑定到响应体的`Person`对象。请注意，我们将用 [`@RestController`](/web/20221105120039/https://www.baeldung.com/spring-controller-vs-restcontroller) 来注释我们的 API 类，以便用一个隐藏的 [`@ResponseBody`](/web/20221105120039/https://www.baeldung.com/spring-request-response-body) 注释来注释所有 API 方法。

## 4.使用`RestTemplate`

现在我们可以编写一些单元测试来测试我们的`Person` REST API。在这里，**我们将尝试使用`RestTemplate` : `postForObject`、`postForEntity`和`postForLocation`提供的 POST 方法向`Person` API 发送 POST 请求。**

在我们开始实现单元测试之前，让我们定义一个设置方法来初始化我们将在所有单元测试方法中使用的对象:

```
@BeforeClass
public static void runBeforeAllTestMethods() {
    createPersonUrl = "http://localhost:8082/spring-rest/createPerson";
    updatePersonUrl = "http://localhost:8082/spring-rest/updatePerson";

    restTemplate = new RestTemplate();
    headers = new HttpHeaders();
    headers.setContentType(MediaType.APPLICATION_JSON);
    personJsonObject = new JSONObject();
    personJsonObject.put("id", 1);
    personJsonObject.put("name", "John");
}
```

除了这个设置方法之外，请注意，在我们的单元测试中，我们将引用下面的映射器来将 JSON 字符串转换成一个`JSONNode`对象:

```
private final ObjectMapper objectMapper = new ObjectMapper();
```

如前所述，**我们希望以 JSON 格式发布数据。为了实现这一点，我们将使用`APPLICATION_JSON` 媒体类型向我们的请求添加一个`Content-Type`头。**

Spring 的`HttpHeaders`类提供了不同的方法来访问头部。这里，我们通过调用`setContentType`方法将`Content-Type`头设置为`application/json`。我们将把`headers`对象附加到我们的请求中。

### 4.1.用`postForObject`发布 JSON

`RestTemplate`的`postForObject`方法通过向给定的 URI 模板发送一个对象来创建一个新的资源。它返回自动转换成在`responseType`参数中指定的类型的结果。

假设我们想向我们的`Person` API 发出一个 POST 请求，创建一个新的`Person`对象，并在响应中返回这个新创建的对象。

**首先，我们将基于`personJsonObject`和包含** `**Content-Type**` **的头文件构建`HttpEntity`类型的`request`对象。**这允许`postForObject`方法发送一个 JSON 请求体:

```
@Test
public void givenDataIsJson_whenDataIsPostedByPostForObject_thenResponseBodyIsNotNull()
  throws IOException {
    HttpEntity<String> request = 
      new HttpEntity<String>(personJsonObject.toString(), headers);

    String personResultAsJsonStr = 
      restTemplate.postForObject(createPersonUrl, request, String.class);
    JsonNode root = objectMapper.readTree(personResultAsJsonStr);

    assertNotNull(personResultAsJsonStr);
    assertNotNull(root);
    assertNotNull(root.path("name").asText());
}
```

`postForObject()`方法将响应体作为`String`类型返回。

我们还可以通过设置`responseType`参数将响应作为`Person`对象返回:

```
Person person = restTemplate.postForObject(createPersonUrl, request, Person.class);

assertNotNull(person);
assertNotNull(person.getName());
```

实际上，我们的请求处理器方法与`createPersonUrl` URI 相匹配，产生 JSON 格式的响应体。

但这并不是对我们的限制— `postForObject`能够自动将响应体转换成在`responseType`参数中指定的请求的 Java 类型(例如`String`、`Person`)。

### 4.2.用`postForEntity`发布 JSON

相比于`postForObject()`， **`postForEntity()`返回的响应为 [`ResponseEntity`](/web/20221105120039/https://www.baeldung.com/spring-response-entity) 对象。除此之外，这两种方法做同样的工作。**

假设我们想向我们的`Person` API 发出一个 POST 请求，创建一个新的`Person`对象，并以`ResponseEntity`的形式返回响应。

我们可以利用`postForEntity`方法来实现这一点:

```
@Test
public void givenDataIsJson_whenDataIsPostedByPostForEntity_thenResponseBodyIsNotNull()
  throws IOException {
    HttpEntity<String> request = 
      new HttpEntity<String>(personJsonObject.toString(), headers);

    ResponseEntity<String> responseEntityStr = restTemplate.
      postForEntity(createPersonUrl, request, String.class);
    JsonNode root = objectMapper.readTree(responseEntityStr.getBody());

    assertNotNull(responseEntityStr.getBody());
    assertNotNull(root.path("name").asText());
}
```

类似于`postForObject`，`postForEntity`具有`responseType`参数，用于将响应主体转换为请求的 Java 类型。

在这里，我们能够将响应体作为`ResponseEntity<String>`返回。

我们还可以通过将`responseType`参数设置为`Person.class`，将响应作为`ResponseEntity<Person>` 对象返回:

```
ResponseEntity<Person> responseEntityPerson = restTemplate.
  postForEntity(createPersonUrl, request, Person.class);

assertNotNull(responseEntityPerson.getBody());
assertNotNull(responseEntityPerson.getBody().getName());
```

### 4.3.用`postForLocation`发布 JSON

类似于`postForObject`和`postForEntity`方法，`postForLocation`也通过将给定的对象发送到给定的 URI 来创建新的资源。唯一的区别是它返回了`Location`头的值。

记住，我们已经在上面的`updatePerson` REST API 方法中看到了如何设置响应的`Location`头:

```
response.setHeader("Location", ServletUriComponentsBuilder.fromCurrentContextPath()
  .path("/findPerson/" + person.getId()).toUriString());
```

现在让我们假设**在更新了我们发布的`person`对象后，我们想要返回响应的`Location `头。**

我们可以通过使用`postForLocation`方法实现这一点:

```
@Test
public void givenDataIsJson_whenDataIsPostedByPostForLocation_thenResponseBodyIsTheLocationHeader() 
  throws JsonProcessingException {
    HttpEntity<String> request = new HttpEntity<String>(personJsonObject.toString(), headers);
    URI locationHeader = restTemplate.postForLocation(updatePersonUrl, request);

    assertNotNull(locationHeader);
}
```

## 5.结论

在本文中，我们探索了如何使用`RestTemplate`通过 JSON 发出 POST 请求。

和往常一样，所有的例子和代码片段都可以在 GitHub 上找到[。](https://web.archive.org/web/20221105120039/https://github.com/eugenp/tutorials/tree/master/spring-web-modules/spring-resttemplate-2)