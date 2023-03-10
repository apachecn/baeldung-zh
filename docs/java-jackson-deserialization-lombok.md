# 杰克逊与龙目岛的反序列化

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-jackson-deserialization-lombok>

## 1。概述

通常情况下，当使用 Lombok 的[项目时，我们会希望将我们的数据相关类与 JSON 框架结合起来，如](https://web.archive.org/web/20221125202732/https://projectlombok.org/)[Jackson](/web/20221125202732/https://www.baeldung.com/jackson)。鉴于 JSON 广泛存在于大多数现代 API 和数据服务中，这一点尤其正确。

在这个快速教程中，**我们将看看如何配置我们的 Lombok [构建器类](/web/20221125202732/https://www.baeldung.com/lombok-builder)与 Jackson** 无缝协作。

## 2。依赖性

我们需要从添加到我们的`pom.xml`中的`[org.projectlombok](https://web.archive.org/web/20221125202732/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22org.projectlombok%22)`开始:

```java
<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
    <version>1.18.24</version>
</dependency>
```

当然，我们还需要 [`jackson-databind`](https://web.archive.org/web/20221125202732/https://search.maven.org/search?q=g:com.fasterxml.jackson.core) 依赖关系:

```java
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-databind</artifactId>
    <version>2.14.1</version>
</dependency>
```

## 3。一个简单的果域

让我们继续定义一个支持 Lombok 的类，用一个 id 和一个名称来表示一种水果:

```java
@Data
@Builder
@Jacksonized
public class Fruit {
    private String name;
    private int id;
}
```

让我们浏览一下 POJO 的关键注释:

*   首先，我们从向我们的类添加`@Data`注释开始——这将生成通常与简单 POJO 相关的所有样板文件，比如 getters 和 setters
*   然后，我们添加了 [`@Builder`](https://web.archive.org/web/20221125202732/http://baeldung.com/lombok-builder) 注释——一种使用[构建器模式](/web/20221125202732/https://www.baeldung.com/creational-design-patterns#builder)创建对象的有用机制
*   最后也是最重要的，我们添加了`@Jacksonized`注释

简单地说，`@Jacksonized`注释是`@Builder`的附加注释。**使用这个注释让我们可以自动配置生成的构建器类来处理 Jackson 的反序列化**。

值得注意的是，该注释仅在同时存在`@Builder`或 [`@SuperBuilder`](/web/20221125202732/https://www.baeldung.com/lombok-builder-inheritance) 注释时有效。

最后，我们应该提到，虽然`@Jacksonized`是在 Lombok v1.18.14\. **中引入的，但它仍然被认为是一个[实验特性](https://web.archive.org/web/20221125202732/https://projectlombok.org/features/experimental/)。**

## 4.反序列化和序列化

现在已经定义了我们的域模型，让我们继续编写一个单元测试来使用 Jackson 反序列化一个水果:

```java
@Test
public void withFruitJSON_thenDeserializeSucessfully() throws IOException {
    String json = "{\"name\":\"Apple\",\"id\":101}";

    Fruit fruit = newObjectMapper().readValue(json, Fruit.class);
    assertEquals(new Fruit("Apple", 101), fruit);
}
```

`ObjectMapper`的简单的`readValue()` API 已经足够好了。我们可以用它将一个 JSON 水果串反序列化成一个`Fruit` Java 对象。

同样，我们可以使用`writeValue()` API 将`Fruit`对象序列化为 JSON 输出:

```java
@Test
void withFruitObject_thenSerializeSucessfully() throws IOException {
    Fruit fruit = Fruit.builder()
      .id(101)
      .name("Apple")
      .build();

    String json = newObjectMapper().writeValueAsString(fruit);
    assertEquals("{\"name\":\"Apple\",\"id\":101}", json);
}
```

测试显示了我们如何使用 Lombok builder API 构建一个`Fruit`,以及序列化的 Java 对象与预期的 JSON 字符串匹配。

## 5。使用定制的构建器

有时我们可能需要使用定制的构建器实现，而不是 Lombok 为我们生成的实现。例如，**当我们的 bean 的属性名与 JSON 字符串**中的字段名不同时。

假设我们想要反序列化以下 JSON 字符串:

```java
{
    "id": 5,
    "name": "Bob"
}
```

但是我们 POJO 上的属性不匹配:

```java
@Data
@Builder(builderClassName = "EmployeeBuilder")
@JsonDeserialize(builder = Employee.EmployeeBuilder.class)
@AllArgsConstructor
public class Employee {

    private int identity;
    private String firstName;

}
```

**在这个场景中，我们可以将`@JsonDeserialize` 注释与`@JsonPOJOBuilder`注释**一起使用，我们可以将其插入到生成的构建器类中，以覆盖 Jackson 的默认值:

```java
@JsonPOJOBuilder(buildMethodName = "createEmployee", withPrefix = "construct")
public static class EmployeeBuilder {

    private int idValue;
    private String nameValue;

    public EmployeeBuilder constructId(int id) {
        idValue = id;
        return this;
    }

    public EmployeeBuilder constructName(String name) {
        nameValue = name;
        return this;
    }

    public Employee createEmployee() {
        return new Employee(idValue, nameValue);
    }
}
```

然后我们可以像以前一样编写一个测试:

```java
@Test
public void withEmployeeJSON_thenDeserializeSucessfully() throws IOException {
    String json = "{\"id\":5,\"name\":\"Bob\"}";
    Employee employee = newObjectMapper().readValue(json, Employee.class);

    assertEquals(5, employee.getIdentity());
    assertEquals("Bob", employee.getFirstName());
}
```

结果显示，尽管属性名不匹配，但已经从 JSON 源成功地重新创建了一个新的`Employee`数据对象。

## 6。结论

在这篇短文中，我们看到了两种简单的方法来配置我们的 Lombok [构建器类](/web/20221125202732/https://www.baeldung.com/lombok-builder)以无缝地与 Jackson 一起工作。

如果没有`@Jacksonized` 注释`,` ，我们将不得不专门定制我们的构建器类。然而，使用`@Jacksonized` 让我们可以使用 Lombok 生成的构建器类。

和往常一样，GitHub 上的[提供了这篇文章的完整源代码。](https://web.archive.org/web/20221125202732/https://github.com/eugenp/tutorials/tree/master/lombok-modules/lombok-2)