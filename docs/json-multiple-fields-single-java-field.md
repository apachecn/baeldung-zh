# 将多个 JSON 字段映射到一个 Java 字段

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/json-multiple-fields-single-java-field>

## 1.概观

在本教程中，我们将看到如何使用 Jackson 和 Gson 将不同的 JSON 字段映射到一个 Java 字段上。

## 2.Maven 依赖性

为了使用[杰克森](https://web.archive.org/web/20220923223542/https://search.maven.org/search?q=a:jackson-databind%20AND%20g:com.fasterxml.jackson.core)和 [Gson](https://web.archive.org/web/20220923223542/https://search.maven.org/search?q=a:gson%20AND%20g:com.google.code.gson) 库，我们需要向 POM 添加以下依赖项:

```java
<dependency>
    <groupId>com.google.code.gson</groupId>
    <artifactId>gson</artifactId>
    <version>2.8.5</version>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-databind</artifactId>
    <version>2.13.0</version>
    <scope>test</scope>
</dependency>
```

## 3.样本 JSON

假设我们想将不同地点的天气细节放入 Java 应用程序中。我们发现有几个网站将天气数据作为 JSON 文档发布。但是，它们使用的格式略有不同:

```java
{
    "location": "London",
    "temp": 15,
    "weather": "Cloudy"
}
```

并且:

```java
{
    "place": "Lisbon",
    "temperature": 35,
    "outlook": "Sunny"
}
```

我们希望将这两种格式反序列化到同一个 Java 类中，命名为`Weather`:

```java
public class Weather {
    private String location;
    private int temp;
    private String outlook;
}
```

因此，让我们看看如何使用 Jackson 和 Gson 库来实现这一点。

## 4.利用杰克逊

**为了实现这一点，我们将利用 Jackson 的`@JsonProperty`和`@JsonAlias`注释。这将允许我们将多个 JSON 属性映射到同一个 Java 字段**。

首先，我们将使用`@JsonProperty`注释，以便 Jackson 知道要映射的 JSON 字段的名称。`@JsonProperty`注释中的值用于反序列化和序列化。

然后我们可以使用`@JsonAlias`注释。因此，Jackson 将知道 JSON 文档中映射到 Java 字段的其他字段的名称。`@JsonAlias`注释中的值仅用于反序列化:

```java
@JsonProperty("location")
@JsonAlias("place")
private String location;
@JsonProperty("temp")
@JsonAlias("temperature")
private int temp;

@JsonProperty("outlook")
@JsonAlias("weather")
private String outlook; 
```

现在我们已经添加了注释，让我们使用 Jackson 的`ObjectMapper`通过`Weather`类创建 Java 对象:

```java
@Test
public void givenTwoJsonFormats_whenDeserialized_thenWeatherObjectsCreated() throws Exception {

    ObjectMapper mapper = new ObjectMapper();

    Weather weather = mapper.readValue("{\n"  
      + "  \"location\": \"London\",\n" 
      + "  \"temp\": 15,\n" 
      + "  \"weather\": \"Cloudy\"\n" 
      + "}", Weather.class);

    assertEquals("London", weather.getLocation());
    assertEquals("Cloudy", weather.getOutlook());
    assertEquals(15, weather.getTemp());

    weather = mapper.readValue("{\n" 
      + "  \"place\": \"Lisbon\",\n" 
      + "  \"temperature\": 35,\n"
      + "  \"outlook\": \"Sunny\"\n"
      + "}", Weather.class);

    assertEquals("Lisbon", weather.getLocation());
    assertEquals("Sunny", weather.getOutlook());
    assertEquals(35, weather.getTemp());
}
```

## 5.使用 Gson

**现在，让我们对 Gson 进行同样的尝试。我们需要在`@SerializedName`注释中使用`value`和`alternate`参数。**

第一个将用作默认值，而第二个将用于指示我们想要映射的 JSON 字段的备用名称:

```java
@SerializedName(value="location", alternate="place")
private String location;
@SerializedName(value="temp", alternate="temperature")
private int temp;

@SerializedName(value="outlook", alternate="weather")
private String outlook; 
```

现在我们已经添加了注释，让我们测试我们的示例:

```java
@Test
public void givenTwoJsonFormats_whenDeserialized_thenWeatherObjectsCreated() throws Exception {

    Gson gson = new GsonBuilder().create();
    Weather weather = gson.fromJson("{\n" 
      + "  \"location\": \"London\",\n" 
      + "  \"temp\": 15,\n" 
      + "  \"weather\": \"Cloudy\"\n" 
      + "}", Weather.class);

    assertEquals("London", weather.getLocation());
    assertEquals("Cloudy", weather.getOutlook());
    assertEquals(15, weather.getTemp());

    weather = gson.fromJson("{\n"
      + "  \"place\": \"Lisbon\",\n"
      + "  \"temperature\": 35,\n"
      + "  \"outlook\": \"Sunny\"\n"
      + "}", Weather.class);

    assertEquals("Lisbon", weather.getLocation());
    assertEquals("Sunny", weather.getOutlook());
    assertEquals(35, weather.getTemp());

}
```

## 6.结论

我们看到，通过使用 Jackson 的`@JsonAlias`或 Gson 的`alternate`参数，我们可以很容易地将不同的 JSON 格式转换成同一个 Java 对象。

你可以在 GitHub 的[杰克森](https://web.archive.org/web/20220923223542/https://github.com/eugenp/tutorials/tree/master/jackson-modules/jackson-conversions-2)和 [Gson 项目中找到例子。](https://web.archive.org/web/20220923223542/https://github.com/eugenp/tutorials/tree/master/json-modules/gson)