# 在 Java 中创建自定义注释

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-custom-annotation>

## 1.介绍

Java 注释是一种向源代码添加元数据信息的机制。它们是 JDK5 中添加的 Java 的强大部分。注释提供了使用 XML 描述符和标记接口的替代方法。

虽然我们可以将它们附加到包、类、接口、方法和字段中，但是注释本身对程序的执行没有任何影响。

在本教程中，我们将重点关注如何创建和处理自定义注释。我们可以在我们关于注释基础的文章中阅读更多关于注释的内容。

## 延伸阅读:

## [Java 中的抽象类](/web/20220626075452/https://www.baeldung.com/java-abstract-class)

Learn how and when to use abstract classes as part of a class hierarchy in Java.[Read more](/web/20220626075452/https://www.baeldung.com/java-abstract-class) →

## [Java 中的标记接口](/web/20220626075452/https://www.baeldung.com/java-marker-interfaces)

Learn about Java marker interfaces and how they compare to typical interfaces and annotations[Read more](/web/20220626075452/https://www.baeldung.com/java-marker-interfaces) →

## 2.创建自定义注释

我们将创建三个定制注释，目标是将一个对象序列化为一个 JSON 字符串。

我们将在类级别使用第一个，向编译器表明我们的对象可以被序列化。然后我们将把第二个应用到我们想要包含在 JSON 字符串中的字段。

最后，我们将在方法级别使用第三个注释，来指定我们将用来初始化对象的方法。

### 2.1.类级注释示例

创建定制注释的第一步是使用`@interface`关键字声明它

```
public @interface JsonSerializable {
}
```

下一步是**添加元注释来指定自定义注释的范围和目标**:

```
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.Type)
public @interface JsonSerializable {
}
```

正如我们所见，我们的第一个注释**具有运行时可见性，我们可以将其应用于类型(类)**。此外，它没有方法，因此作为一个简单的标记来标记可以序列化为 JSON 的类。

### 2.2.字段级注释示例

同样，我们创建第二个注释来标记将要包含在生成的 JSON 中的字段:

```
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.FIELD)
public @interface JsonElement {
    public String key() default "";
}
```

该注释声明了一个名为“key”的字符串参数和一个空字符串作为默认值。

当用方法创建自定义注释时，我们应该注意这些**方法必须没有参数，并且不能抛出异常**。另外，**返回类型被限制为原语、字符串、类、枚举、注释和这些类型的数组，**、**默认值不能为空**。

### 2.3.方法级注释示例

让我们想象一下，在将一个对象序列化为 JSON 字符串之前，我们想要执行一些方法来初始化一个对象。因此，我们将创建一个注释来标记这个方法:

```
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface Init {
}
```

我们声明了一个具有运行时可见性的公共注释，可以应用到类的方法中。

### 2.4.应用注释

现在让我们看看如何使用我们的自定义注释。例如，假设我们有一个类型为`Person`的对象，我们希望将其序列化为一个 JSON 字符串。此类型有一个方法，它将名和姓的第一个字母大写。我们希望在序列化对象之前调用此方法:

```
@JsonSerializable
public class Person {

    @JsonElement
    private String firstName;

    @JsonElement
    private String lastName;

    @JsonElement(key = "personAge")
    private String age;

    private String address;

    @Init
    private void initNames() {
        this.firstName = this.firstName.substring(0, 1).toUpperCase() 
          + this.firstName.substring(1);
        this.lastName = this.lastName.substring(0, 1).toUpperCase() 
          + this.lastName.substring(1);
    }

    // Standard getters and setters
}
```

通过使用我们的定制注释，我们表明我们可以将一个`Person`对象序列化为一个 JSON 字符串。此外，输出应该只包含该对象的`firstName`、`lastName`和`age`字段。此外，我们希望在序列化之前调用`initNames()`方法。

通过将`@JsonElement`注释的`key`参数设置为“personAge”，我们表示将使用这个名称作为 JSON 输出中字段的标识符。

为了便于演示，我们将 `initNames()`设为私有，所以我们不能通过手动调用来初始化我们的对象，我们的构造函数也没有使用它。

## 3.处理注释

到目前为止，我们已经看到了如何创建定制注释，以及如何使用它们来修饰`Person`类。现在**我们将看看如何通过使用 Java 的反射 API 来利用它们。**

第一步将检查我们的对象是否是`null`，以及它的类型是否有`@JsonSerializable`注释:

```
private void checkIfSerializable(Object object) {
    if (Objects.isNull(object)) {
        throw new JsonSerializationException("The object to serialize is null");
    }

    Class<?> clazz = object.getClass();
    if (!clazz.isAnnotationPresent(JsonSerializable.class)) {
        throw new JsonSerializationException("The class " 
          + clazz.getSimpleName() 
          + " is not annotated with JsonSerializable");
    }
}
```

然后我们寻找任何带有@Init 注释的方法，并执行它来初始化对象的字段:

```
private void initializeObject(Object object) throws Exception {
    Class<?> clazz = object.getClass();
    for (Method method : clazz.getDeclaredMethods()) {
        if (method.isAnnotationPresent(Init.class)) {
            method.setAccessible(true);
            method.invoke(object);
        }
    }
 }
```

`method`的召唤。`setAccessible` ( `true)`允许我们执行私有的`initNames()` 方法`.`

初始化之后，我们遍历对象的字段，检索 JSON 元素的键和值，并将它们放在一个 map 中。然后，我们从映射中创建 JSON 字符串:

```
private String getJsonString(Object object) throws Exception {	
    Class<?> clazz = object.getClass();
    Map<String, String> jsonElementsMap = new HashMap<>();
    for (Field field : clazz.getDeclaredFields()) {
        field.setAccessible(true);
        if (field.isAnnotationPresent(JsonElement.class)) {
            jsonElementsMap.put(getKey(field), (String) field.get(object));
        }
    }		

    String jsonString = jsonElementsMap.entrySet()
        .stream()
        .map(entry -> "\"" + entry.getKey() + "\":\"" 
          + entry.getValue() + "\"")
        .collect(Collectors.joining(","));
    return "{" + jsonString + "}";
}
```

同样，我们使用了`field`。`setAccessible` ( `tru` `e` `)`因为`Person`对象的字段是私有的。

我们的 JSON 序列化程序类结合了上述所有步骤:

```
public class ObjectToJsonConverter {
    public String convertToJson(Object object) throws JsonSerializationException {
        try {
            checkIfSerializable(object);
            initializeObject(object);
            return getJsonString(object);
        } catch (Exception e) {
            throw new JsonSerializationException(e.getMessage());
        }
    }
}
```

最后，我们运行一个单元测试来验证我们的对象是否按照自定义注释的定义进行了序列化:

```
@Test
public void givenObjectSerializedThenTrueReturned() throws JsonSerializationException {
    Person person = new Person("soufiane", "cheouati", "34");
    ObjectToJsonConverter serializer = new ObjectToJsonConverter(); 
    String jsonString = serializer.convertToJson(person);
    assertEquals(
      "{\"personAge\":\"34\",\"firstName\":\"Soufiane\",\"lastName\":\"Cheouati\"}",
      jsonString);
}
```

## 4.结论

在本文中，我们学习了如何创建不同类型的定制注释。然后我们讨论了如何用它们来装饰我们的物品。最后，我们看了如何使用 Java 的反射 API 来处理它们。

和往常一样，完整的代码可以在 [GitHub](https://web.archive.org/web/20220626075452/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-annotations) 上找到。