# 与杰克逊一起将蛇案反序列化为骆驼案

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/jackson-deserialize-snake-to-camel-case>

## 1.概观

JSON 对象中的字段名可以有多种格式。当我们想把它们加载到 POJOs 中时，我们可能会遇到一个问题，Java 代码中的属性名与 JSON 中的命名约定不匹配。

在这个简短的教程中，我们将看到如何使用[Jackson](/web/20220524023433/https://www.baeldung.com/jackson)T3**将 snake case JSON 反序列化为 camel case 字段。**

## 2.安装杰克逊

让我们首先将 [Jackson 依赖项](https://web.archive.org/web/20220524023433/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22com.fasterxml.jackson.core%22%20AND%20a%3A%22jackson-databind%22)添加到我们的`pom.xml`文件中:

```
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-databind</artifactId>
    <version>2.13</version>
</dependency>
```

## 3.用默认值反序列化

让我们考虑一个例子`User`类:

```
public class User {
    private String firstName;
    private String lastName;

    // standard getters and setters
}
```

让我们尝试加载这个 JSON，它使用 Snake Case 命名标准(小写名称用`_`分隔):

```
{
    "first_name": "Jackie",
    "last_name": "Chan"
}
```

首先，我们需要使用`ObjectMapper`来反序列化这个 JSON:

```
ObjectMapper objectMapper = new ObjectMapper();
User user = objectMapper.readValue(JSON, User.class); 
```

然而，当我们尝试这样做时，我们得到一个错误:

```
com.fasterxml.jackson.databind.exc.UnrecognizedPropertyException: Unrecognized field "first_name" (class com.baeldung.jackson.snakecase.User), not marked as ignorable (2 known properties: "lastName", "firstName"])
```

不幸的是，Jackson 无法将 JSON 中的名称与`User.`中的字段名称完全匹配

接下来，我们将学习解决这个问题的三种方法。

## 4.使用`@JsonProperty`注释

我们可以在类的字段上使用`@JsonProperty`注释，将字段映射到 JSON 中的确切名称:

```
public class UserWithPropertyNames {
    @JsonProperty("first_name")
    private String firstName;

    @JsonProperty("last_name")
    private String lastName;

    // standard getters and setters
}
```

现在我们可以将 JSON 反序列化为一个`UserWithPropertyNames`:

```
ObjectMapper objectMapper = new ObjectMapper();
UserWithPropertyNames user = objectMapper.readValue(JSON, UserWithPropertyNames.class);
assertEquals("Jackie", user.getFirstName());
assertEquals("Chan", user.getLastName());
```

## 5.使用`@JsonNaming`注释

接下来，我们可以在类上使用`@JsonNaming`注释，并且**所有字段将使用 snake case** 进行反序列化:

```
@JsonNaming(PropertyNamingStrategy.SnakeCaseStrategy.class)
public class UserWithSnakeStrategy {
    private String firstName;
    private String lastName;

    // standard getters and setters
}
```

然后再次反序列化我们的 JSON:

```
ObjectMapper objectMapper = new ObjectMapper();
UserWithSnakeStrategy user = objectMapper.readValue(JSON, UserWithSnakeStrategy.class);
assertEquals("Jackie", user.getFirstName());
assertEquals("Chan", user.getLastName());
```

## 6.配置`ObjectMapper`

最后，我们可以使用`ObjectMapper`上的`setPropertyNamingStrategy`方法来为所有序列化配置它:

```
ObjectMapper objectMapper = new ObjectMapper()
  .setPropertyNamingStrategy(PropertyNamingStrategy.SNAKE_CASE);
User user = objectMapper.readValue(JSON, User.class);
assertEquals("Jackie", user.getFirstName());
assertEquals("Chan", user.getLastName());
```

正如我们看到的，我们现在可以将 JSON 反序列化为原始的`User`对象，即使`User`类没有任何注释。

我们应该注意，还有其他命名约定(例如，kebab 案例)，上述解决方案也适用于它们。

## 7.结论

在本文中，我们已经看到了使用 Jackson 将 snake case JSON 反序列化为 camel case 字段的不同方法。

首先，我们显式命名字段。然后我们在 POJO 上设置一个命名策略。

最后，我们向`ObjectMapper`添加了一个全局配置。

和往常一样，本文的示例代码可以在 GitHub 上找到[。](https://web.archive.org/web/20220524023433/https://github.com/eugenp/tutorials/tree/master/jackson-modules/jackson-conversions-2)