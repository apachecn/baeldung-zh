# 将一个 JSON POST 映射到多个 Spring MVC 参数

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-mvc-json-param-mapping>

## 1.概观

当使用 Spring 对 JSON 反序列化的默认支持时，我们被迫将传入的 JSON 映射到单个请求处理程序参数。然而，有时我们更喜欢更细粒度的方法签名。

在本教程中，我们将学习如何使用自定义的`HandlerMethodArgumentResolver`将 JSON POST 反序列化为多个强类型参数。

## 2.问题是

首先，让我们看看 Spring MVC 默认的 JSON 反序列化方法的局限性。

### 2.1.默认的`@RequestBody`行为

让我们从一个示例 JSON 主体开始:

```java
{
   "firstName" : "John",
   "lastName"  :"Smith",
   "age" : 10,
   "address" : {
      "streetName" : "Example Street",
      "streetNumber" : "10A",
      "postalCode" : "1QW34",
      "city" : "Timisoara",
      "country" : "Romania"
   }
}
```

接下来，让我们创建匹配 JSON 输入的[dto](/web/20220929070049/https://www.baeldung.com/java-dto-pattern):

```java
public class UserDto {
    private String firstName;
    private String lastName;
    private String age;
    private AddressDto address;

    // getters and setters
}
```

```java
public class AddressDto {

    private String streetName;
    private String streetNumber;
    private String postalCode;
    private String city;
    private String country;

    // getters and setters
}
```

最后，我们将使用[标准方法](/web/20220929070049/https://www.baeldung.com/spring-mvc-send-json-parameters)，通过`@RequestBody`注释将 JSON 请求反序列化为`UserDto`:

```java
@Controller
@RequestMapping("/user")
public class UserController {

    @PostMapping("/process")
    public ResponseEntity process(@RequestBody UserDto user) {
        /* business processing */
        return ResponseEntity.ok()
            .body(user.toString());
    }
}
```

### 2.2.限制

上述标准解决方案的主要好处是，我们不必手动将 JSON POST 反序列化为 UserDto 对象。

然而，**整个 JSON POST 必须映射到一个请求参数。**这意味着**我们必须为每个预期的 JSON 结构**创建一个单独的 POJO，用专门用于这个目的的类污染我们的代码库。

当我们只需要 JSON 属性的一个子集时，这个结果尤其明显。在上面的请求处理程序中，我们只需要用户的`firstName`和`city`属性，但是我们被迫反序列化整个`UserDto`。

虽然 Spring 允许我们使用`Map`或`ObjectNode`作为参数，而不是使用自己开发的 DTO，但两者都是单参数选项。和 DTO 一样，所有东西都包装在一起。由于`Map` 和`ObjectNode`内容是`String`值，我们必须自己将它们编组为对象。这些选项使我们不必声明单一用途的 dto，但却带来了更多的复杂性。

## 3.**风俗`HandlerMethodArgumentResolver`**

让我们来看看解决上述限制的方法。我们可以使用 Spring MVC 的`HandlerMethodArgumentResolver`来允许我们在请求处理程序中声明所需的 JSON 属性作为参数。

### 3.1.创建控制器

首先，让我们创建一个自定义注释，我们可以使用它将请求处理程序参数映射到 JSON 路径:

```java
@Target(ElementType.PARAMETER)
@Retention(RetentionPolicy.RUNTIME)
public @interface JsonArg {
    String value() default "";
}
```

接下来，我们将创建一个请求处理程序，它使用注释将`firstName`和`city`映射为独立的参数，这些参数与来自 JSON POST 主体的属性相关联:

```java
@Controller
@RequestMapping("/user")
public class UserController {
    @PostMapping("/process/custom")
    public ResponseEntity process(@JsonArg("firstName") String firstName,
      @JsonArg("address.city") String city) {
        /* business processing */
        return ResponseEntity.ok()
            .body(String.format("{\"firstName\": %s, \"city\" : %s}", firstName, city));
    }
}
```

### 3.2.创建自定义`HandlerMethodArgumentResolver`

在 Spring MVC 决定了哪个请求处理程序应该处理传入的请求之后，它会尝试自动解析参数。这包括遍历 Spring 上下文中实现`HandlerMethodArgumentResolver`接口的所有 beans，以防能够解析任何 Spring MVC 不能自动完成的参数。

让我们定义一个`HandlerMethodArgumentResolver`的实现，它将处理所有用`@JsonArg`标注的请求处理器参数:

```java
public class JsonArgumentResolver implements HandlerMethodArgumentResolver {

    private static final String JSON_BODY_ATTRIBUTE = "JSON_REQUEST_BODY";

    @Override
    public boolean supportsParameter(MethodParameter parameter) {
        return parameter.hasParameterAnnotation(JsonArg.class);
    }

    @Override
    public Object resolveArgument(
      MethodParameter parameter, ModelAndViewContainer mavContainer, NativeWebRequest webRequest,
      WebDataBinderFactory binderFactory) 
      throws Exception {
        String body = getRequestBody(webRequest);
        String jsonPath = Objects.requireNonNull(
          Objects.requireNonNull(parameter.getParameterAnnotation(JsonArg.class)).value());
        Class<?> parameterType = parameter.getParameterType();
        return JsonPath.parse(body).read(jsonPath, parameterType);
    }

    private String getRequestBody(NativeWebRequest webRequest) {
        HttpServletRequest servletRequest = Objects.requireNonNull(
          webRequest.getNativeRequest(HttpServletRequest.class));
        String jsonBody = (String) servletRequest.getAttribute(JSON_BODY_ATTRIBUTE);
        if (jsonBody == null) {
            try {
                jsonBody = IOUtils.toString(servletRequest.getInputStream());
                servletRequest.setAttribute(JSON_BODY_ATTRIBUTE, jsonBody);
            } catch (IOException e) {
                throw new RuntimeException(e);
            }
        }
        return jsonBody;
    }
}
```

**Spring 使用`supportsParameter()` 方法来检查这个类是否可以解析给定的参数。**因为我们希望我们的处理程序处理任何用`@JsonArg`标注的参数，如果给定的参数有那个标注，我们返回`true`。

接下来，在`resolveArgument()`方法中，我们提取 JSON 主体，然后将其作为属性附加到请求中，这样我们就可以在后续调用中直接访问它。然后，我们从`@JsonArg`注释中获取 JSON 路径，并使用反射来获取参数的类型。有了 JSON 路径和参数类型信息，我们可以将 JSON 主体的离散部分反序列化为富对象。

### 3.3.注册自定义`HandlerMethodArgumentResolver`

为了让 Spring MVC 使用我们的`JsonArgumentResolver`，我们需要注册它:

```java
@Configuration
public class WebConfig implements WebMvcConfigurer {

    @Override
    public void addArgumentResolvers(List<HandlerMethodArgumentResolver> argumentResolvers) {
        JsonArgumentResolver jsonArgumentResolver = new JsonArgumentResolver();
        argumentResolvers.add(jsonArgumentResolver);
    }
}
```

我们的`JsonArgumentResolver`现在将处理所有用`@JsonArgs`标注的请求处理程序参数。**我们需要确保`@JsonArgs`值是一个有效的 JSON 路径，但是这个过程比`@RequestBody`方法简单，后者要求每个 JSON 结构都有一个单独的 POJO。**

### 3.4.对自定义类型使用参数

为了说明这也适用于定制 Java 类，让我们用强类型 POJO 参数定义一个请求处理程序:

```java
@PostMapping("/process/custompojo")
public ResponseEntity process(
  @JsonArg("firstName") String firstName, @JsonArg("lastName") String lastName,
  @JsonArg("address") AddressDto address) {
    /* business processing */
    return ResponseEntity.ok()
      .body(String.format("{\"firstName\": %s, \"lastName\": %s, \"address\" : %s}",
        firstName, lastName, address));
}
```

我们现在可以将`AddressDto` 映射为一个单独的参数。

### 3.5.测试自定义`JsonArgumentResolver`

让我们编写一个测试用例来证明`JsonArgumentResolver`按预期工作:

```java
@Test
void whenSendingAPostJSON_thenReturnFirstNameAndCity() throws Exception {

    String jsonString = "{\"firstName\":\"John\",\"lastName\":\"Smith\",\"age\":10,\"address\":{\"streetName\":\"Example Street\",\"streetNumber\":\"10A\",\"postalCode\":\"1QW34\",\"city\":\"Timisoara\",\"country\":\"Romania\"}}";

    mockMvc.perform(post("/user/process/custom").content(jsonString)
      .contentType(MediaType.APPLICATION_JSON)
      .accept(MediaType.APPLICATION_JSON))
      .andExpect(status().isOk())
      .andExpect(MockMvcResultMatchers.jsonPath("$.firstName").value("John"))
      .andExpect(MockMvcResultMatchers.jsonPath("$.city").value("Timisoara"));
}
```

接下来，让我们编写一个测试，调用第二个端点将 JSON 直接解析成 POJOs:

```java
@Test
void whenSendingAPostJSON_thenReturnUserAndAddress() throws Exception {
    String jsonString = "{\"firstName\":\"John\",\"lastName\":\"Smith\",\"address\":{\"streetName\":\"Example Street\",\"streetNumber\":\"10A\",\"postalCode\":\"1QW34\",\"city\":\"Timisoara\",\"country\":\"Romania\"}}";
    ObjectMapper mapper = new ObjectMapper();
    UserDto user = mapper.readValue(jsonString, UserDto.class);
    AddressDto address = user.getAddress();

    String mvcResult = mockMvc.perform(post("/user/process/custompojo").content(jsonString)
      .contentType(MediaType.APPLICATION_JSON)
      .accept(MediaType.APPLICATION_JSON))
      .andExpect(status().isOk())
      .andReturn()
      .getResponse()
      .getContentAsString();

    assertEquals(String.format("{\"firstName\": %s, \"lastName\": %s, \"address\" : %s}",
      user.getFirstName(), user.getLastName(), address), mvcResult);
}
```

## 4.结论

在本文中，我们查看了 Spring MVC 默认反序列化行为的一些限制，然后学习了如何使用自定义的`HandlerMethodArgumentResolver`来克服它们。

和往常一样，这些例子的代码可以在 GitHub 的[上找到。](https://web.archive.org/web/20220929070049/https://github.com/eugenp/tutorials/tree/master/spring-web-modules/spring-mvc-basics-5)