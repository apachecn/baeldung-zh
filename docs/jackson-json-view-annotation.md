# 杰克逊 JSON 观点

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/jackson-json-view-annotation>

## 1。概述

在本教程中，我们将介绍如何使用 Jackson JSON 视图来序列化/反序列化对象，定制视图，最后介绍如何开始与 Spring 集成。

## 2。使用 JSON 视图序列化

首先，让我们来看一个简单的例子——**用`@JsonView`** 序列化一个对象。

以下是我们的观点:

```java
public class Views {
    public static class Public {
    }
}
```

和“`User`”实体:

```java
public class User {
    public int id;

    @JsonView(Views.Public.class)
    public String name;
}
```

现在让我们使用我们的视图序列化一个“`User`”实例:

```java
@Test
public void whenUseJsonViewToSerialize_thenCorrect() 
  throws JsonProcessingException {

    User user = new User(1, "John");

    ObjectMapper mapper = new ObjectMapper();
    mapper.disable(MapperFeature.DEFAULT_VIEW_INCLUSION);

    String result = mapper
      .writerWithView(Views.Public.class)
      .writeValueAsString(user);

    assertThat(result, containsString("John"));
    assertThat(result, not(containsString("1")));
}
```

请注意，因为我们是在特定视图活动的情况下序列化的，所以我们只看到了正确的字段被序列化。

理解这一点也很重要——默认情况下——所有没有明确标记为视图一部分的属性都是序列化的。我们正在用便利的`DEFAULT_VIEW_INCLUSION`特性禁用这种行为。

## 3。使用多个 JSON 视图

接下来，让我们看看如何使用多个 JSON 视图，每个视图都有不同的字段，如下例所示:

这里我们必须看到`Internal`扩展了`Public`的视图，内部视图扩展了公共视图:

```java
public class Views {
    public static class Public {
    }

    public static class Internal extends Public {
    }
}
```

这里是我们的实体“`Item`”，其中只有字段`id`和`name`包含在`Public`视图中:

```java
public class Item {

    @JsonView(Views.Public.class)
    public int id;

    @JsonView(Views.Public.class)
    public String itemName;

    @JsonView(Views.Internal.class)
    public String ownerName;
}
```

如果我们使用`Public`视图来序列化——只有`id`和`name`会被序列化到 JSON:

```java
@Test
public void whenUsePublicView_thenOnlyPublicSerialized() 
  throws JsonProcessingException {

    Item item = new Item(2, "book", "John");

    ObjectMapper mapper = new ObjectMapper();
    String result = mapper
      .writerWithView(Views.Public.class)
      .writeValueAsString(item);

    assertThat(result, containsString("book"));
    assertThat(result, containsString("2"));

    assertThat(result, not(containsString("John")));
}
```

但是如果我们使用`Internal`视图来执行序列化，所有字段都将成为 JSON 输出的一部分:

```java
@Test
public void whenUseInternalView_thenAllSerialized() 
  throws JsonProcessingException {

    Item item = new Item(2, "book", "John");

    ObjectMapper mapper = new ObjectMapper();
    String result = mapper
      .writerWithView(Views.Internal.class)
      .writeValueAsString(item);

    assertThat(result, containsString("book"));
    assertThat(result, containsString("2"));

    assertThat(result, containsString("John"));
}
```

## 4。使用 JSON 视图反序列化

现在，让我们看看如何使用 JSON 视图来反序列化对象——特别是一个`User`实例:

```java
@Test
public void whenUseJsonViewToDeserialize_thenCorrect() 
  throws IOException {
    String json = "{"id":1,"name":"John"}";

    ObjectMapper mapper = new ObjectMapper();
    User user = mapper
      .readerWithView(Views.Public.class)
      .forType(User.class)
      .readValue(json);

    assertEquals(1, user.getId());
    assertEquals("John", user.getName());
}
```

注意我们如何使用给定的视图使用`readerWithView()` API 创建一个`ObjectReader`。

## 5。定制 JSON 视图

接下来——让我们看看如何定制 JSON 视图。在下一个例子中，我们希望序列化结果中的`User``name`大写。

我们将使用`BeanPropertyWriter`和`BeanSerializerModifier`来定制我们的 JSON 视图。首先——这里是将`User`转换成大写字母的`BeanPropertyWriter`T3:

```java
public class UpperCasingWriter extends BeanPropertyWriter {
    BeanPropertyWriter _writer;

    public UpperCasingWriter(BeanPropertyWriter w) {
        super(w);
        _writer = w;
    }

    @Override
    public void serializeAsField(Object bean, JsonGenerator gen, 
      SerializerProvider prov) throws Exception {
        String value = ((User) bean).name;
        value = (value == null) ? "" : value.toUpperCase();
        gen.writeStringField("name", value);
    }
}
```

这里是用我们的自定义`UpperCasingWriter`设置`User`名称`BeanPropertyWriter`的`BeanSerializerModifier`:

```java
public class MyBeanSerializerModifier extends BeanSerializerModifier{

    @Override
    public List<BeanPropertyWriter> changeProperties(
      SerializationConfig config, BeanDescription beanDesc, 
      List<BeanPropertyWriter> beanProperties) {
        for (int i = 0; i < beanProperties.size(); i++) {
            BeanPropertyWriter writer = beanProperties.get(i);
            if (writer.getName() == "name") {
                beanProperties.set(i, new UpperCasingWriter(writer));
            }
        }
        return beanProperties;
    }
}
```

现在，让我们使用修改后的序列化程序来序列化一个`User`实例:

```java
@Test
public void whenUseCustomJsonViewToSerialize_thenCorrect() 
  throws JsonProcessingException {
    User user = new User(1, "John");
    SerializerFactory serializerFactory = BeanSerializerFactory.instance
      .withSerializerModifier(new MyBeanSerializerModifier());

    ObjectMapper mapper = new ObjectMapper();
    mapper.setSerializerFactory(serializerFactory);

    String result = mapper
      .writerWithView(Views.Public.class)
      .writeValueAsString(user);

    assertThat(result, containsString("JOHN"));
    assertThat(result, containsString("1"));
}
```

## 6。通过 Spring 使用 JSON 视图

最后——让我们快速看一下在 **Spring 框架**中使用 JSON 视图。我们可以利用`@JsonView`注释在 API 级别定制我们的 JSON 响应。

在下面的例子中，我们使用了`Public`视图来响应:

```java
@JsonView(Views.Public.class)
@RequestMapping("/items/{id}")
public Item getItemPublic(@PathVariable int id) {
    return ItemManager.getById(id);
}
```

回应是:

```java
{"id":2,"itemName":"book"}
```

当我们使用如下的`Internal`视图时:

```java
@JsonView(Views.Internal.class)
@RequestMapping("/items/internal/{id}")
public Item getItemInternal(@PathVariable int id) {
    return ItemManager.getById(id);
}
```

这就是答案:

```java
{"id":2,"itemName":"book","ownerName":"John"}
```

如果你想更深入地使用 Spring 4.1 的视图，你应该看看 Spring 4.1 中的杰克逊改进。

## 7。结论

在这个快速教程中，我们看了 Jackson JSON 视图和@JsonView 注释。我们展示了如何使用 JSON 视图对序列化/反序列化过程进行细粒度控制——使用单个或多个视图。本教程的完整代码可以在 GitHub 上找到。