# Jackson 对象映射器简介

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/jackson-object-mapper-tutorial>

## 1。概述

本教程着重于理解 Jackson `ObjectMapper` 类，以及如何将 Java 对象序列化为 JSON，并将 JSON 字符串反序列化为 Java 对象。

要了解更多关于杰克逊图书馆的信息，杰克逊教程是一个很好的起点。

## 延伸阅读:

## [与杰克逊的传承](/web/20220707143834/https://www.baeldung.com/jackson-inheritance)

This tutorial will demonstrate how to handle inclusion of subtype metadata and ignoring properties inherited from superclasses with Jackson.[Read more](/web/20220707143834/https://www.baeldung.com/jackson-inheritance) →

## [杰克逊 JSON 观点](/web/20220707143834/https://www.baeldung.com/jackson-json-view-annotation)

How to use the @JsonView annotation in Jackson to perfectly control the serialization of your objects (without and with Spring).[Read more](/web/20220707143834/https://www.baeldung.com/jackson-json-view-annotation) →

## [Jackson–定制串行器](/web/20220707143834/https://www.baeldung.com/jackson-custom-serialization)

Control your JSON output with Jackson 2 by using a Custom Serializer.[Read more](/web/20220707143834/https://www.baeldung.com/jackson-custom-serialization) →

## 2。依赖性

让我们首先向`pom.xml`添加以下依赖项:

```java
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-databind</artifactId>
    <version>2.13.0</version>
</dependency> 
```

此依赖项还会将下列库临时添加到类路径中:

1.  `jackson-annotations`
2.  `jackson-core`

对于`[jackson-databind](https://web.archive.org/web/20220707143834/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22com.fasterxml.jackson.core%22%20AND%20a%3A%22jackson-databind%22)`，总是使用 Maven 中央存储库中的最新版本。

## 3。`ObjectMapper`读写使用

让我们从基本的读写操作开始。

**简单的`ObjectMapper`API 是一个很好的切入点。**我们可以用它将 JSON 内容解析或反序列化成 Java 对象。

另外，在编写方面，**我们可以使用`writeValue` API 将任何 Java 对象序列化为 JSON 输出。**

在本文中，我们将使用下面的带有两个字段的`Car`类作为序列化或反序列化的对象:

```java
public class Car {

    private String color;
    private String type;

    // standard getters setters
}
```

### 3.1。Java 对象到 JSON

让我们看第一个使用`ObjectMapper`类的`writeValue`方法将 Java 对象序列化为 JSON 的例子:

```java
ObjectMapper objectMapper = new ObjectMapper();
Car car = new Car("yellow", "renault");
objectMapper.writeValue(new File("target/car.json"), car); 
```

文件中上述内容的输出将是:

```java
{"color":"yellow","type":"renault"} 
```

`ObjectMapper`类的方法`writeValueAsString`和`writeValueAsBytes`从 Java 对象生成 JSON，并将生成的 JSON 作为字符串或字节数组返回:

```java
String carAsString = objectMapper.writeValueAsString(car); 
```

### 3.2。Java 对象的 JSON

下面是一个使用`ObjectMapper` 类将 JSON 字符串转换成 Java 对象的简单例子:

```java
String json = "{ \"color\" : \"Black\", \"type\" : \"BMW\" }";
Car car = objectMapper.readValue(json, Car.class); 
```

`readValue()`函数也接受其他形式的输入，比如包含 JSON 字符串的文件:

```java
Car car = objectMapper.readValue(new File("src/test/resources/json_car.json"), Car.class);
```

或者一个 URL:

```java
Car car = 
  objectMapper.readValue(new URL("file:src/test/resources/json_car.json"), Car.class);
```

### 3.3。JSON 去杰克森`JsonNode`

或者，可以将 JSON 解析成一个`JsonNode`对象，并用于从特定节点检索数据:

```java
String json = "{ \"color\" : \"Black\", \"type\" : \"FIAT\" }";
JsonNode jsonNode = objectMapper.readTree(json);
String color = jsonNode.get("color").asText();
// Output: color -> Black 
```

### 3.4。从 JSON 数组字符串创建 Java 列表

我们可以使用`TypeReference`将数组形式的 JSON 解析成 Java 对象列表:

```java
String jsonCarArray = 
  "[{ \"color\" : \"Black\", \"type\" : \"BMW\" }, { \"color\" : \"Red\", \"type\" : \"FIAT\" }]";
List<Car> listCar = objectMapper.readValue(jsonCarArray, new TypeReference<List<Car>>(){}); 
```

### 3.5。从 JSON 字符串创建 Java 映射

类似地，我们可以将 JSON 解析成 Java `Map`:

```java
String json = "{ \"color\" : \"Black\", \"type\" : \"BMW\" }";
Map<String, Object> map 
  = objectMapper.readValue(json, new TypeReference<Map<String,Object>>(){}); 
```

## 4。高级功能

Jackson 库的最大优势之一是高度可定制的序列化和反序列化过程。

在这一节中，我们将介绍一些高级特性，其中输入或输出的 JSON 响应可能与生成或使用响应的对象不同。

### 4.1。配置序列化或反序列化功能

在将 JSON 对象转换为 Java 类时，如果 JSON 字符串有一些新字段，默认过程将导致一个异常:

```java
String jsonString 
  = "{ \"color\" : \"Black\", \"type\" : \"Fiat\", \"year\" : \"1970\" }"; 
```

上例中的 JSON 字符串在默认解析过程中对 Java 对象的`Class Car`将导致`UnrecognizedPropertyException` 异常。

**通过`configure`方法，我们可以扩展默认流程来忽略新字段**:

```java
objectMapper.configure(DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES, false);
Car car = objectMapper.readValue(jsonString, Car.class);

JsonNode jsonNodeRoot = objectMapper.readTree(jsonString);
JsonNode jsonNodeYear = jsonNodeRoot.get("year");
String year = jsonNodeYear.asText(); 
```

还有一个选项基于`FAIL_ON_NULL_FOR_PRIMITIVES`，它定义了是否允许原始值的`null` 值:

```java
objectMapper.configure(DeserializationFeature.FAIL_ON_NULL_FOR_PRIMITIVES, false); 
```

类似地，`FAIL_ON_NUMBERS_FOR_ENUM` 控制是否允许将枚举值序列化/反序列化为数字:

```java
objectMapper.configure(DeserializationFeature.FAIL_ON_NUMBERS_FOR_ENUMS, false);
```

您可以在[官方网站](https://web.archive.org/web/20220707143834/https://github.com/FasterXML/jackson-databind/wiki/Serialization-Features)上找到序列化和反序列化特性的完整列表。

### 4.2。创建自定义序列化程序或反序列化程序

`ObjectMapper`类的另一个重要特性是能够注册定制的[串行化器](/web/20220707143834/https://www.baeldung.com/jackson-custom-serialization)和[去串行化器](/web/20220707143834/https://www.baeldung.com/jackson-deserialization)。

当输入或输出 JSON 响应在结构上不同于必须序列化或反序列化到的 Java 类时，自定义序列化程序和反序列化程序非常有用。

下面是一个定制 JSON 序列化器的例子:

```java
public class CustomCarSerializer extends StdSerializer<Car> {

    public CustomCarSerializer() {
        this(null);
    }

    public CustomCarSerializer(Class<Car> t) {
        super(t);
    }

    @Override
    public void serialize(
      Car car, JsonGenerator jsonGenerator, SerializerProvider serializer) {
        jsonGenerator.writeStartObject();
        jsonGenerator.writeStringField("car_brand", car.getType());
        jsonGenerator.writeEndObject();
    }
} 
```

可以像这样调用这个自定义序列化程序:

```java
ObjectMapper mapper = new ObjectMapper();
SimpleModule module = 
  new SimpleModule("CustomCarSerializer", new Version(1, 0, 0, null, null, null));
module.addSerializer(Car.class, new CustomCarSerializer());
mapper.registerModule(module);
Car car = new Car("yellow", "renault");
String carJson = mapper.writeValueAsString(car); 
```

下面是`Car`在客户端的样子(作为 JSON 输出):

```java
var carJson = {"car_brand":"renault"} 
```

这里有一个定制 JSON 反序列化器的例子:

```java
public class CustomCarDeserializer extends StdDeserializer<Car> {

    public CustomCarDeserializer() {
        this(null);
    }

    public CustomCarDeserializer(Class<?> vc) {
        super(vc);
    }

    @Override
    public Car deserialize(JsonParser parser, DeserializationContext deserializer) {
        Car car = new Car();
        ObjectCodec codec = parser.getCodec();
        JsonNode node = codec.readTree(parser);

        // try catch block
        JsonNode colorNode = node.get("color");
        String color = colorNode.asText();
        car.setColor(color);
        return car;
    }
} 
```

可以通过以下方式调用此自定义反序列化程序:

```java
String json = "{ \"color\" : \"Black\", \"type\" : \"BMW\" }";
ObjectMapper mapper = new ObjectMapper();
SimpleModule module =
  new SimpleModule("CustomCarDeserializer", new Version(1, 0, 0, null, null, null));
module.addDeserializer(Car.class, new CustomCarDeserializer());
mapper.registerModule(module);
Car car = mapper.readValue(json, Car.class); 
```

### 4.3。处理日期格式

`java.util.Date`的默认序列化产生一个数字，即纪元时间戳(UTC 自 1970 年 1 月 1 日以来的毫秒数)。但这不是很容易让人读懂，需要进一步转换才能以一种人类可读的格式显示。

让我们用`datePurchased`属性包装到目前为止在`Request`类中使用的`Car`实例:

```java
public class Request 
{
    private Car car;
    private Date datePurchased;

    // standard getters setters
} 
```

要控制日期的字符串格式并将其设置为例如`yyyy-MM-dd HH:mm a z`，请考虑下面的代码片段:

```java
ObjectMapper objectMapper = new ObjectMapper();
DateFormat df = new SimpleDateFormat("yyyy-MM-dd HH:mm a z");
objectMapper.setDateFormat(df);
String carAsString = objectMapper.writeValueAsString(request);
// output: {"car":{"color":"yellow","type":"renault"},"datePurchased":"2016-07-03 11:43 AM CEST"} 
```

要了解更多关于与杰克逊约会的序列化，请阅读[我们更深入的报道](/web/20220707143834/https://www.baeldung.com/jackson-serialize-dates)。

### 4.4。处理托收

通过`DeserializationFeature`类可以获得的另一个小而有用的特性是从 JSON 数组响应中生成我们想要的集合类型的能力。

例如，我们可以将结果生成为数组:

```java
String jsonCarArray = 
  "[{ \"color\" : \"Black\", \"type\" : \"BMW\" }, { \"color\" : \"Red\", \"type\" : \"FIAT\" }]";
ObjectMapper objectMapper = new ObjectMapper();
objectMapper.configure(DeserializationFeature.USE_JAVA_ARRAY_FOR_JSON_ARRAY, true);
Car[] cars = objectMapper.readValue(jsonCarArray, Car[].class);
// print cars
```

或者作为`List`:

```java
String jsonCarArray = 
  "[{ \"color\" : \"Black\", \"type\" : \"BMW\" }, { \"color\" : \"Red\", \"type\" : \"FIAT\" }]";
ObjectMapper objectMapper = new ObjectMapper();
List<Car> listCar = objectMapper.readValue(jsonCarArray, new TypeReference<List<Car>>(){});
// print cars
```

更多关于处理杰克逊收藏的信息请点击[这里](/web/20220707143834/https://www.baeldung.com/jackson-collection-array)。

## 5。结论

Jackson 是一个坚实而成熟的 Java JSON 序列化/反序列化库。`ObjectMapper` API 提供了一种简单的方法来解析和生成 JSON 响应对象，并且非常灵活。本文讨论了使该库如此受欢迎的主要特性。

本文附带的源代码可以在 GitHub 上找到。