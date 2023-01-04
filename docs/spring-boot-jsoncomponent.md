# 在 Spring Boot 使用@JsonComponent

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-boot-jsoncomponent>

## 1。概述

这篇简短的文章关注的是如何在 Spring Boot 中使用`[@JsonComponent](https://web.archive.org/web/20220627081729/https://docs.spring.io/spring-boot/docs/current/api/org/springframework/boot/jackson/JsonComponent.html)`注释。

注释允许我们将带注释的类公开为 Jackson 序列化器和/或反序列化器，而不需要手动将其添加到`ObjectMapper`中。

这是核心 Spring Boot 模块的一部分，因此在普通的 Spring Boot 应用程序中不需要额外的依赖。

## 2。序列化

让我们从下面的`User`对象开始，它包含了一种喜爱的颜色:

```java
public class User {
    private Color favoriteColor;

    // standard getters/constructors
} 
```

如果我们使用 Jackson 和默认设置序列化这个对象，我们会得到:

```java
{
  "favoriteColor": {
    "red": 0.9411764740943909,
    "green": 0.9725490212440491,
    "blue": 1.0,
    "opacity": 1.0,
    "opaque": true,
    "hue": 208.00000000000003,
    "saturation": 0.05882352590560913,
    "brightness": 1.0
  }
} 
```

我们可以通过打印 RGB 值来使 JSON 更加简洁，可读性更强——例如，在 CSS 中使用。

在这种情况下，我们只需要创建一个实现`JsonSerializer`的类:

```java
@JsonComponent
public class UserJsonSerializer extends JsonSerializer<User> {

    @Override
    public void serialize(User user, JsonGenerator jsonGenerator, 
      SerializerProvider serializerProvider) throws IOException, 
      JsonProcessingException {

        jsonGenerator.writeStartObject();
        jsonGenerator.writeStringField(
          "favoriteColor", 
          getColorAsWebColor(user.getFavoriteColor()));
        jsonGenerator.writeEndObject();
    }

    private static String getColorAsWebColor(Color color) {
        int r = (int) Math.round(color.getRed() * 255.0);
        int g = (int) Math.round(color.getGreen() * 255.0);
        int b = (int) Math.round(color.getBlue() * 255.0);
        return String.format("#%02x%02x%02x", r, g, b);
    }
} 
```

有了这个序列化器，生成的 JSON 被简化为:

```java
{"favoriteColor":"#f0f8ff"} 
```

由于有了`@JsonComponent`注释，序列化程序被注册在 Spring Boot 应用程序的 Jackson `ObjectMapper`中。我们可以用下面的 JUnit 测试来测试这一点:

```java
@JsonTest
@RunWith(SpringRunner.class)
public class UserJsonSerializerTest {

    @Autowired
    private ObjectMapper objectMapper;

    @Test
    public void testSerialization() throws JsonProcessingException {
        User user = new User(Color.ALICEBLUE);
        String json = objectMapper.writeValueAsString(user);

        assertEquals("{\"favoriteColor\":\"#f0f8ff\"}", json);
    }
} 
```

## 3。反序列化

继续同一个例子，我们可以编写一个反序列化器，将 web color `String`转换成 JavaFX Color 对象:

```java
@JsonComponent
public class UserJsonDeserializer extends JsonDeserializer<User> {

    @Override
    public User deserialize(JsonParser jsonParser, 
      DeserializationContext deserializationContext) throws IOException, 
      JsonProcessingException {

        TreeNode treeNode = jsonParser.getCodec().readTree(jsonParser);
        TextNode favoriteColor
          = (TextNode) treeNode.get("favoriteColor");
        return new User(Color.web(favoriteColor.asText()));
    }
} 
```

让我们测试新的反序列化器，并确保一切按预期工作:

```java
@JsonTest
@RunWith(SpringRunner.class)
public class UserJsonDeserializerTest {

    @Autowired
    private ObjectMapper objectMapper;

    @Test
    public void testDeserialize() throws IOException {
        String json = "{\"favoriteColor\":\"#f0f8ff\"}"
        User user = objectMapper.readValue(json, User.class);

        assertEquals(Color.ALICEBLUE, user.getFavoriteColor());
    }
} 
```

## 4。`Serializer`和`Deserializer`在一个档次

如果需要，我们可以通过使用两个内部类并在封闭类上添加`@JsonComponent`来连接一个类中的序列化器和反序列化器:

```java
@JsonComponent
public class UserCombinedSerializer {

    public static class UserJsonSerializer 
      extends JsonSerializer<User> {

        @Override
        public void serialize(User user, JsonGenerator jsonGenerator, 
          SerializerProvider serializerProvider) throws IOException, 
          JsonProcessingException {

            jsonGenerator.writeStartObject();
            jsonGenerator.writeStringField(
              "favoriteColor", getColorAsWebColor(user.getFavoriteColor()));
            jsonGenerator.writeEndObject();
        }

        private static String getColorAsWebColor(Color color) {
            int r = (int) Math.round(color.getRed() * 255.0);
            int g = (int) Math.round(color.getGreen() * 255.0);
            int b = (int) Math.round(color.getBlue() * 255.0);
            return String.format("#%02x%02x%02x", r, g, b);
        }
    }

    public static class UserJsonDeserializer 
      extends JsonDeserializer<User> {

        @Override
        public User deserialize(JsonParser jsonParser, 
          DeserializationContext deserializationContext)
          throws IOException, JsonProcessingException {

            TreeNode treeNode = jsonParser.getCodec().readTree(jsonParser);
            TextNode favoriteColor = (TextNode) treeNode.get(
              "favoriteColor");
            return new User(Color.web(favoriteColor.asText()));
        }
    }
} 
```

## 5。结论

这篇快速教程展示了如何通过利用带有`@JsonComponent`注释的组件扫描，在 Spring Boot 应用程序中快速添加 Jackson 串行器/解串器。

代码片段可以在 GitHub 上找到[。](https://web.archive.org/web/20220627081729/https://github.com/eugenp/tutorials/tree/master/spring-boot-modules/spring-boot-data)