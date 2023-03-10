# Spring Boot:自定义杰克森对象映射器

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-boot-customize-jackson-objectmapper>

## 1.概观

当使用 JSON 格式时，Spring Boot 将使用一个`ObjectMapper`实例来序列化响应和反序列化请求。

在本教程中，我们将了解配置序列化和反序列化选项的最常见方式。

要了解更多关于杰克逊的信息，请务必查看我们的[杰克逊教程](/web/20221129010833/https://www.baeldung.com/jackson)。

## 延伸阅读:

## [Spring JSON-P 与杰克逊](/web/20221129010833/https://www.baeldung.com/spring-jackson-jsonp)

The article is focused on showing how to use the new JSON-P support in Spring 4.1.[Read more](/web/20221129010833/https://www.baeldung.com/spring-jackson-jsonp) →

## [如何在 Spring MVC 中设置 JSON 内容类型](/web/20221129010833/https://www.baeldung.com/spring-mvc-set-json-content-type)

Learn different options for setting the content type in Spring MVC.[Read more](/web/20221129010833/https://www.baeldung.com/spring-mvc-set-json-content-type) →

## [杰克逊对象映射器简介](/web/20221129010833/https://www.baeldung.com/jackson-object-mapper-tutorial)

The article discusses Jackson's central ObjectMapper class, basic serialization and deserialization as well as configuring the two processes.[Read more](/web/20221129010833/https://www.baeldung.com/jackson-object-mapper-tutorial) →

## 2.默认配置

默认情况下，Spring Boot 配置将禁用以下功能:

*   `MapperFeature.DEFAULT_VIEW_INCLUSION`
*   `DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES`
*   `SerializationFeature.WRITE_DATES_AS_TIMESTAMPS`

让我们从一个简单的例子开始:

*   客户端将向我们的`/coffee?name=Lavazza`发送一个 GET 请求。
*   控制器将返回一个新的`Coffee`对象。
*   Spring 将使用`ObjectMapper`将我们的 POJO 序列化为 JSON。

我们将通过使用`String`和`LocalDateTime`对象来举例说明定制选项:

```java
public class Coffee {

    private String name;
    private String brand;
    private LocalDateTime date;

   //getters and setters
}
```

**我们还将定义一个简单的 REST 控制器来演示序列化**:

```java
@GetMapping("/coffee")
public Coffee getCoffee(
        @RequestParam(required = false) String brand,
        @RequestParam(required = false) String name) {
    return new Coffee()
      .setBrand(brand)
      .setDate(FIXED_DATE)
      .setName(name);
}
```

默认情况下，这将是调用 GET `http://lolcahost:8080/coffee?brand=Lavazza`时的响应:

```java
{
  "name": null,
  "brand": "Lavazza",
  "date": "2020-11-16T10:21:35.974"
}
```

我们希望排除`null`值，并有一个自定义的日期格式(dd-MM-yyyy HH:mm)。这是我们最后的回应:

```java
{
  "brand": "Lavazza",
  "date": "04-11-2020 10:34"
}
```

当使用 Spring Boot 时，我们可以选择定制默认的`ObjectMapper`或者覆盖它。我们将在接下来的章节中讨论这两个选项。

## 3.定制默认`ObjectMapper`

在这一节中，我们将看到如何定制 Spring Boot 使用的默认`ObjectMapper`。

### 3.1.应用程序属性和自定义 Jackson 模块

配置映射器最简单的方法是通过应用程序属性。

这是配置的一般结构:

```java
spring.jackson.<category_name>.<feature_name>=true,false
```

作为一个例子，我们将添加以下内容来禁用`SerializationFeature.WRITE_DATES_AS_TIMESTAMPS`:

```java
spring.jackson.serialization.write-dates-as-timestamps=false
```

除了提到的功能类别，我们还可以配置属性包含:

```java
spring.jackson.default-property-inclusion=always, non_null, non_absent, non_default, non_empty 
```

配置环境变量是最简单的方法。**这种方法的缺点是我们不能定制高级选项，比如为`LocalDateTime`定制日期格式。**

此时，我们将获得以下结果:

```java
{
  "brand": "Lavazza",
  "date": "2020-11-16T10:35:34.593"
}
```

为了实现我们的目标，我们将使用自定义日期格式注册一个新的`JavaTimeModule `:

```java
@Configuration
@PropertySource("classpath:coffee.properties")
public class CoffeeRegisterModuleConfig {

    @Bean
    public Module javaTimeModule() {
        JavaTimeModule module = new JavaTimeModule();
        module.addSerializer(LOCAL_DATETIME_SERIALIZER);
        return module;
    }
} 
```

此外，配置属性文件`coffee.properties`将包含以下内容:

```java
spring.jackson.default-property-inclusion=non_null
```

Spring Boot 将自动注册任何类型`com.fasterxml.jackson.databind.Module`的 bean。这是我们的最终结果:

```java
{
  "brand": "Lavazza",
  "date": "16-11-2020 10:43"
}
```

### 3.2.`Jackson2ObjectMapperBuilderCustomizer`

这个功能接口的目的是允许我们创建配置 beans。

它们将应用于通过`Jackson2ObjectMapperBuilder`创建的默认`ObjectMapper`:

```java
@Bean
public Jackson2ObjectMapperBuilderCustomizer jsonCustomizer() {
    return builder -> builder.serializationInclusion(JsonInclude.Include.NON_NULL)
      .serializers(LOCAL_DATETIME_SERIALIZER);
}
```

**配置 beans 以特定的顺序应用，我们可以使用`@Order `注释来控制。**如果我们想从不同的配置或模块中配置`ObjectMapper`，这种优雅的方法是合适的。

## 4.覆盖默认配置

如果我们想完全控制配置，有几个选项可以禁用自动配置，只允许应用我们的自定义配置。

让我们仔细看看这些选项。

### 4.1.`ObjectMapper`

覆盖默认配置的最简单方法是定义一个`ObjectMapper` bean，并将其标记为`@Primary`:

```java
@Bean
@Primary
public ObjectMapper objectMapper() {
    JavaTimeModule module = new JavaTimeModule();
    module.addSerializer(LOCAL_DATETIME_SERIALIZER);
    return new ObjectMapper()
      .setSerializationInclusion(JsonInclude.Include.NON_NULL)
      .registerModule(module);
}
```

当我们想要完全控制序列化过程并且不想允许外部配置时，我们应该使用这种方法。

### 4.2.`Jackson2ObjectMapperBuilder`

另一个干净的方法是定义一个`Jackson2ObjectMapperBuilder` bean `.`

实际上，Spring Boot 在构建`ObjectMapper`时默认使用这个构建器，并将自动选择已定义的构建器:

```java
@Bean
public Jackson2ObjectMapperBuilder jackson2ObjectMapperBuilder() {
    return new Jackson2ObjectMapperBuilder().serializers(LOCAL_DATETIME_SERIALIZER)
      .serializationInclusion(JsonInclude.Include.NON_NULL);
}
```

默认情况下，它将配置两个选项:

*   禁用`MapperFeature.DEFAULT_VIEW_INCLUSION`
*   禁用`DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES`

根据 [`Jackson2ObjectMapperBuilder`文档](https://web.archive.org/web/20221129010833/https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/http/converter/json/Jackson2ObjectMapperBuilder.html)，它还会注册一些模块，如果它们出现在类路径上的话:

*   jackson-datatype-jdk8:支持其他 Java 8 类型，如`Optional`
*   jackson-datatype-jsr310:支持 Java 8 日期和时间 API 类型
*   jackson-datatype-joda:支持 Joda-Time 类型
*   jackson-module-kotlin:支持 kotlin 类和数据类

**这种方法的优点是`Jackson2ObjectMapperBuilder`提供了一种简单直观的方式来构建`ObjectMapper`。**

### 4.3.`MappingJackson2HttpMessageConverter`

我们可以定义一个类型为`MappingJackson2HttpMessageConverter`的 bean，Spring Boot 会自动使用它:

```java
@Bean
public MappingJackson2HttpMessageConverter mappingJackson2HttpMessageConverter() {
    Jackson2ObjectMapperBuilder builder = new Jackson2ObjectMapperBuilder().serializers(LOCAL_DATETIME_SERIALIZER)
      .serializationInclusion(JsonInclude.Include.NON_NULL);
    return new MappingJackson2HttpMessageConverter(builder.build());
}
```

请务必查看我们的 [Spring Http 消息转换器](/web/20221129010833/https://www.baeldung.com/spring-httpmessageconverter-rest)文章以了解更多信息。

## 5.测试配置

为了测试我们的配置，我们将使用`TestRestTemplate` 并将对象序列化为`String`。

通过这种方式，我们可以验证我们的`Coffee` 对象是在没有`null`值的情况下使用自定义日期格式序列化的:

```java
@Test
public void whenGetCoffee_thenSerializedWithDateAndNonNull() {
    String formattedDate = DateTimeFormatter.ofPattern(CoffeeConstants.dateTimeFormat).format(FIXED_DATE);
    String brand = "Lavazza";
    String url = "/coffee?brand=" + brand;

    String response = restTemplate.getForObject(url, String.class);

    assertThat(response).isEqualTo("{\"brand\":\"" + brand + "\",\"date\":\"" + formattedDate + "\"}");
}
```

## 6.结论

在本文中，我们研究了在使用 Spring Boot 时配置 JSON 序列化选项的几种方法。

我们看到了两种不同的方法:配置默认选项或覆盖默认配置。

和往常一样，GitHub 上的[提供了这篇文章的完整源代码。](https://web.archive.org/web/20221129010833/https://github.com/eugenp/tutorials/tree/master/spring-boot-modules/spring-boot-data-2)