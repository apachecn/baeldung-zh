# 杰克逊例外——问题和解决方案

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/jackson-exception>

## 1。概述

在本教程中，我们将回顾**最常见的 Jackson 异常** — `JsonMappingException`和`UnrecognizedPropertyException`。

最后，我们将简要讨论杰克逊“没有这样的方法”的错误。

## 延伸阅读:

## [Jackson–自定义序列化器](/web/20220707143834/https://www.baeldung.com/jackson-custom-serialization)

用 Jackson 2 通过使用自定义序列化器来控制你的 JSON 输出。[阅读更多](/web/20220707143834/https://www.baeldung.com/jackson-custom-serialization) →

## [杰克逊注解举例](/web/20220707143834/https://www.baeldung.com/jackson-annotations)

杰克逊的核心基本上就是一套注解——确保你理解好这些。[阅读更多](/web/20220707143834/https://www.baeldung.com/jackson-annotations)→

## [Jackson 中自定义反序列化入门](/web/20220707143834/https://www.baeldung.com/jackson-deserialization)

使用 Jackson 将自定义 JSON 映射到任意 java 实体图，对反序列化过程拥有完全控制权。[阅读更多](/web/20220707143834/https://www.baeldung.com/jackson-deserialization) →

## 2。`JsonMappingException`:无法构造的实例

### 2.1。问题

首先，我们来看看 JsonMappingException:不能构造的实例。

如果 **Jackson 不能创建类**的实例，就会抛出这个异常，如果类是`abstract`或者它只是一个`interface`，就会发生这种情况。

这里我们将尝试从类`Zoo`中反序列化一个实例，该实例具有属性`animal`和`abstract`类型`Animal`:

```java
public class Zoo {
    public Animal animal;

    public Zoo() { }
}

abstract class Animal {
    public String name;

    public Animal() { }
}

class Cat extends Animal {
    public int lives;

    public Cat() { }
}
```

当我们试图将 JSON `String`反序列化为 Zoo 实例时，它抛出 JsonMappingException:不能构造实例:

```java
@Test(expected = JsonMappingException.class)
public void givenAbstractClass_whenDeserializing_thenException() 
  throws IOException {
    String json = "{"animal":{"name":"lacy"}}";
    ObjectMapper mapper = new ObjectMapper();

    mapper.reader().forType(Zoo.class).readValue(json);
}
```

这是**完整异常**:

```java
com.fasterxml.jackson.databind.JsonMappingException: 
Can not construct instance of org.baeldung.jackson.exception.Animal,
  problem: abstract types either need to be mapped to concrete types, 
  have custom deserializer, 
  or be instantiated with additional type information
  at 
[Source: {"animal":{"name":"lacy"}}; line: 1, column: 2] 
(through reference chain: org.baeldung.jackson.exception.Zoo["animal"])
	at c.f.j.d.JsonMappingException.from(JsonMappingException.java:148)
```

### 2.2。解决方案

我们可以用一个简单的注释来解决这个问题——抽象类上的`@JsonDeserialize`:

```java
@JsonDeserialize(as = Cat.class)
abstract class Animal {...}
```

注意，如果我们有不止一个抽象类的子类型，我们应该考虑包含子类型信息，如文章[与 Jackson](/web/20220707143834/https://www.baeldung.com/jackson-inheritance) 中所示。

## 3。`JsonMappingException`:没有合适的建造者

### 3.1。问题

现在我们来看看常见的 JsonMappingException:没有合适的构造函数` found for type`。

如果 **Jackson 不能访问构造函数，就会抛出这个异常。**

在下面的例子中，类`User`没有默认的构造函数:

```java
public class User {
    public int id;
    public String name;

    public User(int id, String name) {
        this.id = id;
        this.name = name;
    }
}
```

当我们试图将一个 JSON 字符串反序列化给用户时，会抛出 JsonMappingException:找不到合适的构造函数:

```java
@Test(expected = JsonMappingException.class)
public void givenNoDefaultConstructor_whenDeserializing_thenException() 
  throws IOException {
    String json = "{"id":1,"name":"John"}";
    ObjectMapper mapper = new ObjectMapper();

    mapper.reader().forType(User.class).readValue(json);
}
```

这是**完整异常**:

```java
com.fasterxml.jackson.databind.JsonMappingException: 
No suitable constructor found for type 
[simple type, class org.baeldung.jackson.exception.User]:
 can not instantiate from JSON object (need to add/enable type information?)
 at [Source: {"id":1,"name":"John"}; line: 1, column: 2]
        at c.f.j.d.JsonMappingException.from(JsonMappingException.java:148)
```

### 3.2。解决方案

为了解决这个问题，我们只需添加一个默认的构造函数:

```java
public class User {
    public int id;
    public String name;

    public User() {
        super();
    }

    public User(int id, String name) {
        this.id = id;
        this.name = name;
    }
}
```

现在，当我们进行反序列化时，该过程将正常工作:

```java
@Test
public void givenDefaultConstructor_whenDeserializing_thenCorrect() 
  throws IOException {

    String json = "{"id":1,"name":"John"}";
    ObjectMapper mapper = new ObjectMapper();

    User user = mapper.reader()
      .forType(User.class).readValue(json);
    assertEquals("John", user.name);
}
```

## 4。`JsonMappingException`:根名称与预期的不匹配

### 4.1。问题

接下来，我们来看看 JsonMappingException: Root Name 与预期不符。

如果 JSON 与 Jackson 所寻找的不完全匹配，就会抛出这个异常。

例如，主 JSON 可以包装为:

```java
@Test(expected = JsonMappingException.class)
public void givenWrappedJsonString_whenDeserializing_thenException()
  throws IOException {
    String json = "{"user":{"id":1,"name":"John"}}";

    ObjectMapper mapper = new ObjectMapper();
    mapper.enable(DeserializationFeature.UNWRAP_ROOT_VALUE);

    mapper.reader().forType(User.class).readValue(json);
}
```

这是**完整异常**:

```java
com.fasterxml.jackson.databind.JsonMappingException:
Root name 'user' does not match expected ('User') for type
 [simple type, class org.baeldung.jackson.dtos.User]
 at [Source: {"user":{"id":1,"name":"John"}}; line: 1, column: 2]
   at c.f.j.d.JsonMappingException.from(JsonMappingException.java:148) 
```

### 4.2。解决方案

我们可以使用注释`@JsonRootName`来解决这个问题:

```java
@JsonRootName(value = "user")
public class UserWithRoot {
    public int id;
    public String name;
}
```

当我们尝试反序列化包装的 JSON 时，它可以正常工作:

```java
@Test
public void 
  givenWrappedJsonStringAndConfigureClass_whenDeserializing_thenCorrect() 
  throws IOException {

    String json = "{"user":{"id":1,"name":"John"}}";

    ObjectMapper mapper = new ObjectMapper();
    mapper.enable(DeserializationFeature.UNWRAP_ROOT_VALUE);

    UserWithRoot user = mapper.reader()
      .forType(UserWithRoot.class)
      .readValue(json);
    assertEquals("John", user.name);
}
```

## 5。`JsonMappingException`:找不到类的序列化程序

### 5.1。问题

现在让我们来看看 JsonMappingException:没有为类找到序列化程序。

如果我们试图序列化一个实例，而它的属性和 getters 是私有的，就会抛出这个异常。

我们将尝试序列化一个`UserWithPrivateFields`:

```java
public class UserWithPrivateFields {
    int id;
    String name;
}
```

当我们尝试序列化`UserWithPrivateFields`的实例时，抛出 JsonMappingException:找不到类的序列化程序:

```java
@Test(expected = JsonMappingException.class)
public void givenClassWithPrivateFields_whenSerializing_thenException() 
  throws IOException {
    UserWithPrivateFields user = new UserWithPrivateFields(1, "John");

    ObjectMapper mapper = new ObjectMapper();
    mapper.writer().writeValueAsString(user);
}
```

这是**完整异常**:

```java
com.fasterxml.jackson.databind.JsonMappingException: 
No serializer found for class org.baeldung.jackson.exception.UserWithPrivateFields
 and no properties discovered to create BeanSerializer 
(to avoid exception, disable SerializationFeature.FAIL_ON_EMPTY_BEANS) )
  at c.f.j.d.ser.impl.UnknownSerializer.failForEmpty(UnknownSerializer.java:59)
```

### 5.2。解决方案

我们可以通过配置`ObjectMapper`可见性来解决这个问题:

```java
@Test
public void givenClassWithPrivateFields_whenConfigureSerializing_thenCorrect() 
  throws IOException {

    UserWithPrivateFields user = new UserWithPrivateFields(1, "John");

    ObjectMapper mapper = new ObjectMapper();
    mapper.setVisibility(PropertyAccessor.FIELD, Visibility.ANY);

    String result = mapper.writer().writeValueAsString(user);
    assertThat(result, containsString("John"));
}
```

或者我们可以使用注释`@JsonAutoDetect`:

```java
@JsonAutoDetect(fieldVisibility = Visibility.ANY)
public class UserWithPrivateFields { ... }
```

当然，如果我们可以选择修改类的源代码，我们也可以添加 getters 供 Jackson 使用。

## 6。`JsonMappingException`:无法反序列化的实例

### 6.1。问题

接下来，我们来看看 JsonMappingException:无法反序列化的实例。

如果反序列化时使用了错误的类型，就会抛出这个异常。

在这个例子中，我们试图反序列化一个`User`的`List`:

```java
@Test(expected = JsonMappingException.class)
public void givenJsonOfArray_whenDeserializing_thenException() 
  throws JsonProcessingException, IOException {

    String json 
      = "[{"id":1,"name":"John"},{"id":2,"name":"Adam"}]";
    ObjectMapper mapper = new ObjectMapper();
    mapper.reader().forType(User.class).readValue(json);
}
```

这里是**完整异常**:

```java
com.fasterxml.jackson.databind.JsonMappingException:
Can not deserialize instance of 
  org.baeldung.jackson.dtos.User out of START_ARRAY token
  at [Source: [{"id":1,"name":"John"},{"id":2,"name":"Adam"}]; line: 1, column: 1]
  at c.f.j.d.JsonMappingException.from(JsonMappingException.java:148)
```

### 6.2。解决方案

我们可以通过将类型从`User`改为`List<User>`来解决这个问题:

```java
@Test
public void givenJsonOfArray_whenDeserializing_thenCorrect() 
  throws JsonProcessingException, IOException {

    String json
      = "[{"id":1,"name":"John"},{"id":2,"name":"Adam"}]";

    ObjectMapper mapper = new ObjectMapper();
    List<User> users = mapper.reader()
      .forType(new TypeReference<List<User>>() {})
      .readValue(json);

    assertEquals(2, users.size());
}
```

## 7。`UnrecognizedPropertyException`

### 7.1。问题

现在让我们看看`UnrecognizedPropertyException`。

如果在反序列化时 JSON 字符串中有一个**未知属性，就会抛出这个异常。**

我们将尝试用额外的属性“`checked`”反序列化一个 JSON 字符串:

```java
@Test(expected = UnrecognizedPropertyException.class)
public void givenJsonStringWithExtra_whenDeserializing_thenException() 
  throws IOException {

    String json = "{"id":1,"name":"John", "checked":true}";

    ObjectMapper mapper = new ObjectMapper();
    mapper.reader().forType(User.class).readValue(json);
}
```

这是**完整异常**:

```java
com.fasterxml.jackson.databind.exc.UnrecognizedPropertyException:
Unrecognized field "checked" (class org.baeldung.jackson.dtos.User),
 not marked as ignorable (2 known properties: "id", "name"])
 at [Source: {"id":1,"name":"John", "checked":true}; line: 1, column: 38]
 (through reference chain: org.baeldung.jackson.dtos.User["checked"])
  at c.f.j.d.exc.UnrecognizedPropertyException.from(
    UnrecognizedPropertyException.java:51)
```

### 7.2。解决方案

我们可以通过配置`ObjectMapper`来解决这个问题:

```java
@Test
public void givenJsonStringWithExtra_whenConfigureDeserializing_thenCorrect() 
  throws IOException {

    String json = "{"id":1,"name":"John", "checked":true}";

    ObjectMapper mapper = new ObjectMapper();
    mapper.disable(DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES);

    User user = mapper.reader().forType(User.class).readValue(json);
    assertEquals("John", user.name);
}
```

或者我们可以使用注释`@JsonIgnoreProperties`:

```java
@JsonIgnoreProperties(ignoreUnknown = true)
public class User {...}
```

## 8。`JsonParseException`:意外字符(“’(代码 39))

### 8.1。问题

接下来，我们来讨论一下`JsonParseException: Unexpected character (”' (code 39))`。

如果要反序列化的 JSON 字符串包含单引号而不是双引号，就会抛出这个异常。

我们将尝试反序列化包含单引号的 JSON 字符串:

```java
@Test(expected = JsonParseException.class)
public void givenStringWithSingleQuotes_whenDeserializing_thenException() 
  throws JsonProcessingException, IOException {

    String json = "{'id':1,'name':'John'}";
    ObjectMapper mapper = new ObjectMapper();

    mapper.reader()
      .forType(User.class).readValue(json);
}
```

下面是**完整异常**:

```java
com.fasterxml.jackson.core.JsonParseException:
Unexpected character (''' (code 39)): 
  was expecting double-quote to start field name
  at [Source: {'id':1,'name':'John'}; line: 1, column: 3]
  at c.f.j.core.JsonParser._constructError(JsonParser.java:1419)
```

### 8.2。解决方案

我们可以通过配置`ObjectMapper`允许单引号来解决这个问题:

```java
@Test
public void 
  givenStringWithSingleQuotes_whenConfigureDeserializing_thenCorrect() 
  throws JsonProcessingException, IOException {

    String json = "{'id':1,'name':'John'}";

    JsonFactory factory = new JsonFactory();
    factory.enable(JsonParser.Feature.ALLOW_SINGLE_QUOTES);
    ObjectMapper mapper = new ObjectMapper(factory);

    User user = mapper.reader().forType(User.class)
      .readValue(json);

    assertEquals("John", user.name);
}
```

## 9。杰克逊`NoSuchMethodError`

最后，让我们快速讨论一下杰克逊“没有这样的方法”的错误。

当抛出异常时，通常是因为我们的类路径中有多个(不兼容的)Jackson jars 版本。

这是**完整异常**:

```java
java.lang.NoSuchMethodError:
com.fasterxml.jackson.core.JsonParser.getValueAsString()Ljava/lang/String;
 at c.f.j.d.deser.std.StringDeserializer.deserialize(StringDeserializer.java:24)
```

## 10。结论

在这篇文章中，我们深入探讨了最常见的杰克逊问题——异常和错误——寻找每个问题的潜在原因和解决方案。

所有这些例子和代码片段**的实现都可以在[GitHub](https://web.archive.org/web/20220707143834/https://github.com/eugenp/tutorials/tree/master/jackson-modules/jackson-exceptions "Github Project exemplifying these Jackson exception")T3 上找到。这是一个基于 Maven 的项目，因此应该很容易导入和运行。**