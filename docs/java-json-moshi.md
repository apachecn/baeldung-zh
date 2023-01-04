# 魔石 Json 简介

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-json-moshi>

## 1。简介

在本教程中，我们将看一下 [Moshi](https://web.archive.org/web/20221022081830/https://github.com/square/moshi) ，这是一个现代的 Java JSON 库，它将在我们的代码中轻松地提供强大的 JSON 序列化和反序列化。

Moshi 的 API 比其他库(如 Jackson 或 Gson)要小，但没有牺牲功能。这使得它更容易集成到我们的应用程序中，并让我们编写更多的可测试代码。它也是一个较小的依赖，这对于某些场景可能很重要——比如为 Android 开发。

## 2。将 Moshi 添加到我们的构建中

在使用它之前，我们首先需要将 [Moshi JSON 依赖项](https://web.archive.org/web/20221022081830/https://search.maven.org/search?q=g:com.squareup.moshi)添加到我们的`pom.xml`文件中:

```java
<dependency>
    <groupId>com.squareup.moshi</groupId>
    <artifactId>moshi</artifactId>
    <version>1.9.2</version>
</dependency>
<dependency>
    <groupId>com.squareup.moshi</groupId>
    <artifactId>moshi-adapters</artifactId>
    <version>1.9.2</version>
</dependency>
```

`com.squareup.moshi:moshi`依赖项是主库，`com.squareup.moshi:moshi-adapters`依赖项是一些标准类型适配器——我们将在后面更详细地探讨。

## 3。与 Moshi 和 JSON 合作

Moshi 允许我们将任何 Java 值转换成 JSON，并在我们出于任何原因需要的任何地方转换回来——例如，为了文件存储，编写 REST APIs，我们可能有任何需要。

Moshi 使用一个`JsonAdapter`类的概念。这是一种类型安全机制，用于将特定的类序列化为 JSON 字符串，并将 JSON 字符串反序列化回正确的类型:

```java
public class Post {
    private String title;
    private String author;
    private String text;
    // constructor, getters and setters
}

Moshi moshi = new Moshi.Builder().build();
JsonAdapter<Post> jsonAdapter = moshi.adapter(Post.class);
```

一旦我们构建了我们的`JsonAdapter`，我们可以在任何需要的时候使用它，以便使用`toJson()` 方法将我们的值转换成 JSON:

```java
Post post = new Post("My Post", "Baeldung", "This is my post");
String json = jsonAdapter.toJson(post);
// {"author":"Baeldung","text":"This is my post","title":"My Post"}
```

当然，我们可以用相应的`fromJson()`方法将 JSON 转换回预期的 Java 类型:

```java
Post post = jsonAdapter.fromJson(json);
// new Post("My Post", "Baeldung", "This is my post");
```

## 4。标准 Java 类型

Moshi 内置了对标准 Java 类型的支持，完全按照预期在 JSON 之间进行转换。这包括:

*   [所有图元](/web/20221022081830/https://www.baeldung.com/java-primitives)–`int, float, char`等。
*   [所有 Java 盒装等价物](/web/20221022081830/https://www.baeldung.com/java-primitives-vs-objects)–`Integer, Float, Character`等。
*   [T2`String`](/web/20221022081830/https://www.baeldung.com/java-string)
*   [枚举](/web/20221022081830/https://www.baeldung.com/a-guide-to-java-enums)
*   这些类型的[数组](/web/20221022081830/https://www.baeldung.com/java-arrays-guide)
*   [这些类型的标准 Java 集合](/web/20221022081830/https://www.baeldung.com/java-collections)—`List, Set, Map`

除此之外，Moshi 还将自动处理任何任意的 Java bean，将其转换为 JSON 对象，其中的值使用与任何其他类型相同的规则进行转换。这显然意味着 Java bean 中的 Java bean 被正确地序列化到我们需要的深度。

然后,`moshi-adapters`依赖关系让我们可以访问一些额外的转换规则，包括:

*   一个稍微强大一点的枚举适配器——在从 JSON 读取未知值时支持回退值
*   支持 [RFC-3339 格式](https://web.archive.org/web/20221022081830/https://tools.ietf.org/html/rfc3339)的`java.util.Date `适配器

在使用这些支持之前，需要向一个`Moshi`实例注册它们。当我们添加对我们自己的自定义类型的支持时，我们很快就会看到这种确切的模式:

```java
Moshi moshi = new Moshi.builder()
  .add(new Rfc3339DateJsonAdapter())
  .add(CurrencyCode.class, EnumJsonAdapter.create(CurrencyCode.class).withUnknownFallback(CurrencyCode.USD))
  .build()
```

## 5。Moshi 中的自定义类型

到目前为止，我们已经完全支持将任何 Java 对象序列化和反序列化到 JSON 中。但是这并没有给我们太多的控制 JSON 看起来像什么，序列化 Java 对象的字面意思是按原样写对象中的每个字段。这是可行的，但并不总是我们想要的。

相反，我们可以为自己的类型编写自己的适配器，并对这些类型的序列化和反序列化的工作方式进行精确控制。

### 5.1。简单转换

简单的例子是 Java 类型和 JSON 类型之间的转换——例如字符串。当我们需要以特定格式表示复杂数据时，这非常有用。

例如，假设我们有一个表示文章作者的 Java 类型:

```java
public class Author {
    private String name;
    private String email;
    // constructor, getters and setters
}
```

毫不费力，这将序列化为一个包含两个字段的 JSON 对象——`name`和`email`。不过，我们希望将它序列化为一个字符串，将姓名和电子邮件地址组合在一起。

我们通过编写一个包含用`@ToJson`注释的方法的标准类来做到这一点:

```java
public class AuthorAdapter {
    @ToJson
    public String toJson(Author author) {
        return author.name + " <" + author.email + ">";
    }
}
```

显然，我们也需要走另一条路。我们需要将字符串解析回我们的`Author`对象。这是通过添加一个用`@FromJson`注释的方法来实现的:

```java
@FromJson
public Author fromJson(String author) {
    Pattern pattern = Pattern.compile("^(.*) <(.*)>$");
    Matcher matcher = pattern.matcher(author);
    return matcher.find() ? new Author(matcher.group(1), matcher.group(2)) : null;
}
```

一旦完成，我们需要实际利用这一点。我们在创建我们的`Moshi`时通过将适配器添加到我们的`Moshi.Builder`中来实现这一点:

```java
Moshi moshi = new Moshi.Builder()
  .add(new AuthorAdapter())
  .build();
JsonAdapter<Post> jsonAdapter = moshi.adapter(Post.class);
```

现在我们可以立即开始将这些对象与 JSON 相互转换，并获得我们想要的结果:

```java
Post post = new Post("My Post", new Author("Baeldung", "[[email protected]](/web/20221022081830/https://www.baeldung.com/cdn-cgi/l/email-protection)"), "This is my post");
String json = jsonAdapter.toJson(post);
// {"author":"Baeldung <[[email protected]](/web/20221022081830/https://www.baeldung.com/cdn-cgi/l/email-protection)>","text":"This is my post","title":"My Post"}

Post post = jsonAdapter.fromJson(json);
// new Post("My Post", new Author("Baeldung", "[[email protected]](/web/20221022081830/https://www.baeldung.com/cdn-cgi/l/email-protection)"), "This is my post");
```

### 5.2。复杂转换

这些转换是在 Java beans 和 JSON 原语类型之间进行的。我们也可以转换成结构化的 JSON——本质上是让我们将 Java 类型转换成不同的结构，以便在 JSON 中呈现。

例如，我们可能需要将日期/时间值呈现为三个不同的值——日期、时间和时区。

使用 Moshi，我们所要做的就是编写一个表示所需输出的 Java 类型，然后我们的`@ToJson`方法可以返回这个新的 Java 对象，Moshi 将使用它的标准规则将它转换成 JSON:

```java
public class JsonDateTime {
    private String date;
    private String time;
    private String timezone;

    // constructor, getters and setters
}
public class JsonDateTimeAdapter {
    @ToJson
    public JsonDateTime toJson(ZonedDateTime input) {
        String date = input.toLocalDate().toString();
        String time = input.toLocalTime().toString();
        String timezone = input.getZone().toString();
        return new JsonDateTime(date, time, timezone);
    }
}
```

正如我们所料，通过编写一个`@FromJson`方法来实现另一种方式，该方法接受我们新的 JSON 结构化类型并返回我们想要的类型:

```java
@FromJson
public ZonedDateTime fromJson(JsonDateTime input) {
    LocalDate date = LocalDate.parse(input.getDate());
    LocalTime time = LocalTime.parse(input.getTime());
    ZoneId timezone = ZoneId.of(input.getTimezone());
    return ZonedDateTime.of(date, time, timezone);
}
```

然后，我们能够完全像上面那样使用它将我们的`ZonedDateTime`转换成我们的结构化输出，然后再转换回来:

```java
Moshi moshi = new Moshi.Builder()
  .add(new JsonDateTimeAdapter())
  .build();
JsonAdapter<ZonedDateTime> jsonAdapter = moshi.adapter(ZonedDateTime.class);

String json = jsonAdapter.toJson(ZonedDateTime.now());
// {"date":"2020-02-17","time":"07:53:27.064","timezone":"Europe/London"}

ZonedDateTime now = jsonAdapter.fromJson(json);
// 2020-02-17T07:53:27.064Z[Europe/London]
```

### 5.3。替代型适配器

有时我们希望为单个字段使用一个替代的适配器，而不是基于字段的类型。

例如，我们可能会遇到这样一种情况，我们需要将日期和时间呈现为来自纪元的毫秒数，而不是 ISO-8601 字符串。

Moshi 允许我们通过使用一个特殊的注释来实现这一点，然后我们可以将它应用于我们的字段和适配器:

```java
@Retention(RUNTIME)
@Target({ElementType.FIELD, ElementType.PARAMETER, ElementType.METHOD})
@JsonQualifier
public @interface EpochMillis {}
```

其中的关键部分是`@JsonQualifier`注释，它允许 Moshi 将任何用此注释的字段绑定到适当的适配器方法。

接下来，我们需要编写一个适配器。像往常一样，我们有一个`@FromJson`和一个`@ToJson`方法在我们的类型和 JSON 之间进行转换:

```java
public class EpochMillisAdapter {
    @ToJson
    public Long toJson(@EpochMillis Instant input) {
        return input.toEpochMilli();
    }
    @FromJson
    @EpochMillis
    public Instant fromJson(Long input) {
        return Instant.ofEpochMilli(input);
    }
}
```

这里，我们使用了对`@ToJson`方法的输入参数和`@FromJson`方法的返回值的注释。

Moshi 现在可以使用这个适配器或任何同样标注了`@EpochMillis`的字段:

```java
public class Post {
    private String title;
    private String author;
    @EpochMillis Instant posted;
    // constructor, getters and setters
}
```

现在，我们可以根据需要将带注释的类型转换成 JSON，或者反过来:

```java
Moshi moshi = new Moshi.Builder()
  .add(new EpochMillisAdapter())
  .build();
JsonAdapter<Post> jsonAdapter = moshi.adapter(Post.class);

String json = jsonAdapter.toJson(new Post("Introduction to Moshi Json", "Baeldung", Instant.now()));
// {"author":"Baeldung","posted":1582095384793,"title":"Introduction to Moshi Json"}

Post post = jsonAdapter.fromJson(json);
// new Post("Introduction to Moshi Json", "Baeldung", Instant.now())
```

## 6。高级 JSON 处理

现在我们可以在 JSON 和我们的类型之间进行转换，并且我们可以控制这种转换的方式。不过，在我们的处理过程中，有时我们可能需要做一些更高级的事情，Moshi 很容易做到这一点。

### 6.1。重命名 JSON 字段

有时，我们需要 JSON 和 Java beans 有不同的字段名。这可能很简单，只需要在 Java 中使用`camelCase`，在 JSON 中使用`snake_case`，或者完全重命名字段以匹配所需的模式。

我们可以使用`@Json`注释为我们控制的任何 bean 中的任何字段赋予一个新名称:

```java
public class Post {
    private String title;
    @Json(name = "authored_by")
    private String author;
    // constructor, getters and setters
}
```

一旦我们这样做了，Moshi 立即意识到这个字段在 JSON 中有一个不同的名称:

```java
Moshi moshi = new Moshi.Builder()
  .build();
JsonAdapter<Post> jsonAdapter = moshi.adapter(Post.class);

Post post = new Post("My Post", "Baeldung");

String json = jsonAdapter.toJson(post);
// {"authored_by":"Baeldung","title":"My Post"}

Post post = jsonAdapter.fromJson(json);
// new Post("My Post", "Baeldung")
```

### 6.2。瞬态场

在某些情况下，我们可能有不应该包含在 JSON 中的字段。Moshi 使用标准的`transient`限定符来表示这些字段不会被序列化或反序列化:

```java
public static class Post {
    private String title;
    private transient String author;
    // constructor, getters and setters
}
```

然后我们会看到，在序列化和反序列化时，该字段都被完全忽略:

```java
Moshi moshi = new Moshi.Builder()
  .build();
JsonAdapter<Post> jsonAdapter = moshi.adapter(Post.class);

Post post = new Post("My Post", "Baeldung");

String json = jsonAdapter.toJson(post);
// {"title":"My Post"}

Post post = jsonAdapter.fromJson(json);
// new Post("My Post", null)

Post post = jsonAdapter.fromJson("{\"author\":\"Baeldung\",\"title\":\"My Post\"}");
// new Post("My Post", null)
```

### 6.3。默认值

有时我们解析的 JSON 并不包含 Java Bean 中每个字段的值。这很好，魔石会尽全力做正确的事。

当反序列化我们的 JSON 时，Moshi 不能使用任何形式的参数构造函数，但是它可以使用无参数构造函数(如果有的话)。

这将允许我们在 JSON 被序列化之前预先填充 bean，为我们的字段提供任何必需的默认值:

```java
public class Post {
    private String title;
    private String author;
    private String posted;

    public Post() {
        posted = Instant.now().toString();
    }
    // getters and setters
}
```

如果我们解析的 JSON 缺少`title`或`author`字段，那么这些字段将以值`null`结束。如果我们缺少`posted`字段，那么它将具有当前日期和时间:

```java
Moshi moshi = new Moshi.Builder()
  .build();
JsonAdapter<Post> jsonAdapter = moshi.adapter(Post.class);

String json = "{\"title\":\"My Post\"}";
Post post = jsonAdapter.fromJson(json);
// new Post("My Post", null, "2020-02-19T07:27:01.141Z");
```

### 6.4。解析 JSON 数组

到目前为止，我们所做的一切都假设我们正在将单个 JSON 对象序列化和反序列化为单个 Java bean。这是一个非常常见的情况，但不是唯一的情况。有时我们还想处理值的集合，这些值在我们的 JSON 中表示为一个数组。

当数组嵌套在我们的 beans 中时，什么都不用做。魔石就行了。当整个 JSON 是一个数组时，我们必须做更多的工作来实现这一点，这仅仅是因为 Java 泛型的一些限制。我们需要构造我们的`JsonAdapter`,让它知道自己正在反序列化一个通用集合，以及这个集合是什么。

Moshi 提供了一些帮助来构建一个`java.lang.reflect.Type`，当我们构建它时，我们可以将它提供给`JsonAdapter`,这样我们就可以提供这个额外的通用信息:

```java
Moshi moshi = new Moshi.Builder()
  .build();
Type type = Types.newParameterizedType(List.class, String.class);
JsonAdapter<List<String>> jsonAdapter = moshi.adapter(type);
```

完成后，我们的适配器完全按照预期工作，遵循这些新的通用界限:

```java
String json = jsonAdapter.toJson(Arrays.asList("One", "Two", "Three"));
// ["One", "Two", "Three"]

List<String> result = jsonAdapter.fromJson(json);
// Arrays.asList("One", "Two", "Three");
```

## 7。总结

我们已经看到了 Moshi 库如何使 Java 类与 JSON 之间的相互转换变得非常容易，以及它有多么灵活。我们可以在任何需要在 Java 和 JSON 之间转换的地方使用这个库——无论是从文件、数据库列还是 REST APIs 加载和保存。为什么不试试呢？

像往常一样，这篇文章的源代码可以在 GitHub 上找到[。](https://web.archive.org/web/20221022081830/https://github.com/eugenp/tutorials/tree/master/json-modules/json-2)