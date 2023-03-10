# 杰克逊-双向关系

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/jackson-bidirectional-relationships-and-infinite-recursion>

## 1。概述

在本教程中，我们将回顾在 Jackson 中处理**双向关系的最佳方式。**

我们将讨论 Jackson JSON 无限递归问题，然后——我们将看到如何序列化具有双向关系的实体，最后——我们将反序列化它们。

## 2。无限递归

首先，让我们看看杰克逊无限递归问题。在下面的例子中，我们有两个实体——“`User`”和“`Item`”——与**有一个简单的一对多关系**:

`User`实体:

```java
public class User {
    public int id;
    public String name;
    public List<Item> userItems;
}
```

`Item`实体:

```java
public class Item {
    public int id;
    public String itemName;
    public User owner;
}
```

当我们试图序列化一个"`Item`"的实例时，Jackson 会抛出一个`JsonMappingException`异常:

```java
@Test(expected = JsonMappingException.class)
public void givenBidirectionRelation_whenSerializing_thenException()
  throws JsonProcessingException {

    User user = new User(1, "John");
    Item item = new Item(2, "book", user);
    user.addItem(item);

    new ObjectMapper().writeValueAsString(item);
}
```

**满异常**是:

```java
com.fasterxml.jackson.databind.JsonMappingException:
Infinite recursion (StackOverflowError) 
(through reference chain: 
org.baeldung.jackson.bidirection.Item["owner"]
->org.baeldung.jackson.bidirection.User["userItems"]
->java.util.ArrayList[0]
->org.baeldung.jackson.bidirection.Item["owner"]
->…..
```

让我们在接下来的几节课程中，看看如何解决这个问题。

## 3。使用`@JsonManagedReference`、`@JsonBackReference`、

首先，让我们用`@JsonManagedReference`、`@JsonBackReference`来注释这个关系，以便让 Jackson 更好地处理这个关系:

下面是“`User`”实体:

```java
public class User {
    public int id;
    public String name;

    @JsonManagedReference
    public List<Item> userItems;
}
```

`Item`还有那个“:

```java
public class Item {
    public int id;
    public String itemName;

    @JsonBackReference
    public User owner;
}
```

现在让我们测试一下新的实体:

```java
@Test
public void givenBidirectionRelation_whenUsingJacksonReferenceAnnotationWithSerialization_thenCorrect() throws JsonProcessingException {
    final User user = new User(1, "John");
    final Item item = new Item(2, "book", user);
    user.addItem(item);

    final String itemJson = new ObjectMapper().writeValueAsString(item);
    final String userJson = new ObjectMapper().writeValueAsString(user);

    assertThat(itemJson, containsString("book"));
    assertThat(itemJson, not(containsString("John")));

    assertThat(userJson, containsString("John"));
    assertThat(userJson, containsString("userItems"));
    assertThat(userJson, containsString("book"));
}
```

以下是序列化项目对象的输出:

```java
{
 "id":2,
 "itemName":"book"
}
```

下面是序列化用户对象的输出:

```java
{
 "id":1,
 "name":"John",
 "userItems":[{
   "id":2,
   "itemName":"book"}]
}
```

请注意:

*   `@JsonManagedReference`是引用的前向部分——正常序列化的部分。
*   `@JsonBackReference`是引用的后半部分——它将从序列化中省略。
*   序列化的`Item`对象不包含对`User`对象的引用。

另外，请注意，我们不能切换注释。以下内容适用于序列化:

```java
@JsonBackReference
public List<Item> userItems;

@JsonManagedReference
public User owner;
```

但是当我们试图反序列化对象时会抛出一个异常，因为`@JsonBackReference`不能用于集合。

如果我们想让序列化的项目对象包含对用户的引用，我们需要使用`@JsonIdentityInfo`。我们将在下一节讨论这个问题。

## 4。使用`@JsonIdentityInfo`

现在，让我们看看如何使用`@JsonIdentityInfo`来帮助具有双向关系的实体的序列化。

我们将类级注释添加到我们的“`User`”实体中:

```java
@JsonIdentityInfo(
  generator = ObjectIdGenerators.PropertyGenerator.class, 
  property = "id")
public class User { ... }
```

和到"`Item`"实体:

```java
@JsonIdentityInfo(
  generator = ObjectIdGenerators.PropertyGenerator.class, 
  property = "id")
public class Item { ... }
```

测试时间:

```java
@Test
public void givenBidirectionRelation_whenUsingJsonIdentityInfo_thenCorrect()
  throws JsonProcessingException {

    User user = new User(1, "John");
    Item item = new Item(2, "book", user);
    user.addItem(item);

    String result = new ObjectMapper().writeValueAsString(item);

    assertThat(result, containsString("book"));
    assertThat(result, containsString("John"));
    assertThat(result, containsString("userItems"));
}
```

以下是序列化的输出:

```java
{
 "id":2,
 "itemName":"book",
 "owner":
    {
        "id":1,
        "name":"John",
        "userItems":[2]
    }
}
```

## 5。使用`@JsonIgnore`

或者，我们也可以使用`@JsonIgnore`注释来简单地**忽略关系**的一方，从而打破这个链。

在下面的示例中，我们将通过忽略序列化中的"`User`"属性"`userItems`"来防止无限递归:

这里是"`User`"实体:

```java
public class User {
    public int id;
    public String name;

    @JsonIgnore
    public List<Item> userItems;
}
```

这是我们的测试:

```java
@Test
public void givenBidirectionRelation_whenUsingJsonIgnore_thenCorrect()
  throws JsonProcessingException {

    User user = new User(1, "John");
    Item item = new Item(2, "book", user);
    user.addItem(item);

    String result = new ObjectMapper().writeValueAsString(item);

    assertThat(result, containsString("book"));
    assertThat(result, containsString("John"));
    assertThat(result, not(containsString("userItems")));
}
```

以下是序列化的结果:

```java
{
 "id":2,
 "itemName":"book",
 "owner":
    {
        "id":1,
        "name":"John"
    }
}
```

## 6。使用`@JsonView`

我们还可以使用更新的`@JsonView`注释来排除关系的一方。

在下面的例子中——我们使用**两个 JSON 视图——`Public`和`Internal`T5，其中`Internal`扩展了`Public`:**

```java
public class Views {
    public static class Public {}

    public static class Internal extends Public {}
}
```

我们将在`Public`视图—**中包含所有的`User`和`Item`字段，除了`User`字段`userItems`** 将包含在`Internal`视图中:

下面是我们的实体"`User`":

```java
public class User {
    @JsonView(Views.Public.class)
    public int id;

    @JsonView(Views.Public.class)
    public String name;

    @JsonView(Views.Internal.class)
    public List<Item> userItems;
}
```

而这里是我们的实体"`Item`":

```java
public class Item {
    @JsonView(Views.Public.class)
    public int id;

    @JsonView(Views.Public.class)
    public String itemName;

    @JsonView(Views.Public.class)
    public User owner;
}
```

当我们使用`Public`视图进行序列化时，它可以正确地工作——**，因为我们将`userItems`** 排除在序列化之外:

```java
@Test
public void givenBidirectionRelation_whenUsingPublicJsonView_thenCorrect() 
  throws JsonProcessingException {

    User user = new User(1, "John");
    Item item = new Item(2, "book", user);
    user.addItem(item);

    String result = new ObjectMapper().writerWithView(Views.Public.class)
      .writeValueAsString(item);

    assertThat(result, containsString("book"));
    assertThat(result, containsString("John"));
    assertThat(result, not(containsString("userItems")));
}
```

但是如果我们使用`Internal`视图进行序列化，就会抛出`JsonMappingException`,因为所有的字段都包含在内:

```java
@Test(expected = JsonMappingException.class)
public void givenBidirectionRelation_whenUsingInternalJsonView_thenException()
  throws JsonProcessingException {

    User user = new User(1, "John");
    Item item = new Item(2, "book", user);
    user.addItem(item);

    new ObjectMapper()
      .writerWithView(Views.Internal.class)
      .writeValueAsString(item);
}
```

## 7。使用自定义串行器

接下来，让我们看看如何使用自定义序列化程序来序列化具有双向关系的实体。

在以下示例中，我们将使用自定义序列化程序来序列化“`User`”属性“`userItems`”:

下面是“`User`”实体:

```java
public class User {
    public int id;
    public String name;

    @JsonSerialize(using = CustomListSerializer.class)
    public List<Item> userItems;
}
```

而这里是“`CustomListSerializer`”:

```java
public class CustomListSerializer extends StdSerializer<List<Item>>{

   public CustomListSerializer() {
        this(null);
    }

    public CustomListSerializer(Class<List> t) {
        super(t);
    }

    @Override
    public void serialize(
      List<Item> items, 
      JsonGenerator generator, 
      SerializerProvider provider) 
      throws IOException, JsonProcessingException {

        List<Integer> ids = new ArrayList<>();
        for (Item item : items) {
            ids.add(item.id);
        }
        generator.writeObject(ids);
    }
}
```

现在让我们测试一下序列化程序，看看产生了哪种正确的输出:

```java
@Test
public void givenBidirectionRelation_whenUsingCustomSerializer_thenCorrect()
  throws JsonProcessingException {
    User user = new User(1, "John");
    Item item = new Item(2, "book", user);
    user.addItem(item);

    String result = new ObjectMapper().writeValueAsString(item);

    assertThat(result, containsString("book"));
    assertThat(result, containsString("John"));
    assertThat(result, containsString("userItems"));
}
```

以及**用定制串行化器串行化的最终输出**:

```java
{
 "id":2,
 "itemName":"book",
 "owner":
    {
        "id":1,
        "name":"John",
        "userItems":[2]
    }
}
```

## 8。用`@JsonIdentityInfo`反序列化

现在，让我们看看如何使用`@JsonIdentityInfo`反序列化具有双向关系的实体。

这里是"`User`"实体:

```java
@JsonIdentityInfo(
  generator = ObjectIdGenerators.PropertyGenerator.class, 
  property = "id")
public class User { ... }
```

和“`Item`”实体:

```java
@JsonIdentityInfo(
  generator = ObjectIdGenerators.PropertyGenerator.class, 
  property = "id")
public class Item { ... }
```

现在让我们编写一个快速测试——从我们想要解析的一些手动 JSON 数据开始，以正确构造的实体结束:

```java
@Test
public void givenBidirectionRelation_whenDeserializingWithIdentity_thenCorrect() 
  throws JsonProcessingException, IOException {
    String json = 
      "{\"id\":2,\"itemName\":\"book\",\"owner\":{\"id\":1,\"name\":\"John\",\"userItems\":[2]}}";

    ItemWithIdentity item
      = new ObjectMapper().readerFor(ItemWithIdentity.class).readValue(json);

    assertEquals(2, item.id);
    assertEquals("book", item.itemName);
    assertEquals("John", item.owner.name);
}
```

## 9。使用自定义反序列化器

最后，让我们使用一个定制的反序列化器来反序列化具有双向关系的实体。

在下面的例子中，我们将使用一个定制的反序列化器来解析“`User`”属性“`userItems`”:

下面是"`User`"实体:

```java
public class User {
    public int id;
    public String name;

    @JsonDeserialize(using = CustomListDeserializer.class)
    public List<Item> userItems;
}
```

而这里是我们的“`CustomListDeserializer`”:

```java
public class CustomListDeserializer extends StdDeserializer<List<Item>>{

    public CustomListDeserializer() {
        this(null);
    }

    public CustomListDeserializer(Class<?> vc) {
        super(vc);
    }

    @Override
    public List<Item> deserialize(
      JsonParser jsonparser, 
      DeserializationContext context) 
      throws IOException, JsonProcessingException {

        return new ArrayList<>();
    }
}
```

简单的测试是:

```java
@Test
public void givenBidirectionRelation_whenUsingCustomDeserializer_thenCorrect()
  throws JsonProcessingException, IOException {
    String json = 
      "{\"id\":2,\"itemName\":\"book\",\"owner\":{\"id\":1,\"name\":\"John\",\"userItems\":[2]}}";

    Item item = new ObjectMapper().readerFor(Item.class).readValue(json);

    assertEquals(2, item.id);
    assertEquals("book", item.itemName);
    assertEquals("John", item.owner.name);
}
```

## 10。结论

在本教程中，我们演示了如何使用 Jackson 来序列化/反序列化具有双向关系的实体。

所有这些例子和代码片段**的实现可以在我们的 [GitHub 项目](https://web.archive.org/web/20220628090852/https://github.com/eugenp/tutorials/tree/master/jackson-modules/jackson-annotations#readme "Github Project covering all Jackson examples")** 中找到——这是一个基于 Maven 的项目，所以它应该很容易导入和运行。