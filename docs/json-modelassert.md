# JSON 的 ModelAssert 库指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/json-modelassert>

## 1.概观

在为使用 JSON 的软件编写自动化测试时，我们经常需要将 JSON 数据与一些期望值进行比较。

在某些情况下，我们可以将实际和预期的 JSON 视为字符串，并执行字符串比较，但这有许多限制。

在本教程中，我们将看看如何使用 [ModelAssert](https://web.archive.org/web/20220824101529/https://github.com/webcompere/model-assert) 编写 JSON 值之间的断言和比较。我们将看到如何在 JSON 文档中构造单个值的断言，以及如何比较文档。我们还将讨论如何处理无法预测其确切值的字段，比如日期或 GUIDs。

## 2.入门指南

ModelAssert 是一个数据断言库，语法类似于 [AssertJ](/web/20220824101529/https://www.baeldung.com/introduction-to-assertj) ，功能堪比 [JSONAssert](/web/20220824101529/https://www.baeldung.com/jsonassert) 。它基于 [Jackson](/web/20220824101529/https://www.baeldung.com/jackson) 进行 JSON 解析，并使用 [JSON 指针](/web/20220824101529/https://www.baeldung.com/json-pointer)表达式来描述文档中字段的路径。

让我们首先为这个 JSON 编写一些简单的断言:

```java
{
   "name": "Baeldung",
   "isOnline": true,
   "topics": [ "Java", "Spring", "Kotlin", "Scala", "Linux" ]
}
```

### 2.1.属国

首先，让我们将[模型断言](https://web.archive.org/web/20220824101529/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22uk.org.webcompere%22%20AND%20a%3A%22model-assert%22)添加到我们的`pom.xml`中:

```java
<dependency>
    <groupId>uk.org.webcompere</groupId>
    <artifactId>model-assert</artifactId>
    <version>1.0.0</version>
    <scope>test</scope>
</dependency>
```

### 2.2.断言 JSON 对象中的字段

让我们假设 JSON 作为一个`String,`返回给我们，我们想检查一下`name`字段是否等于`Baeldung`:

```java
assertJson(jsonString)
  .at("/name").isText("Baeldung");
```

`assertJson`方法将从各种来源读取 JSON，包括`String`、`File`、`Path,`和杰克森的`JsonNode`。返回的对象是一个断言，我们可以使用 fluent DSL(特定于领域的语言)来添加条件。

`at`方法描述了我们希望在文档中进行字段断言的地方。然后，`isText`指定我们期望一个值为`Baeldung`的文本节点。

我们可以通过使用稍微长一点的 JSON 指针表达式在`topics`数组中断言一个路径:

```java
assertJson(jsonString)
  .at("/topics/1").isText("Spring");
```

虽然我们可以一个接一个地编写字段断言，**但是我们也可以将它们组合成一个断言**:

```java
assertJson(jsonString)
  .at("/name").isText("Baeldung")
  .at("/topics/1").isText("Spring");
```

### 2.3.为什么字符串比较不起作用

我们经常想要将整个 JSON 文档与另一个文档进行比较。字符串比较虽然在某些情况下是可能的，但通常会被不相关的 JSON 格式问题抓住**:**

```java
String expected = loadFile(EXPECTED_JSON_PATH);
assertThat(jsonString)
  .isEqualTo(expected);
```

像这样的失败消息很常见:

```java
org.opentest4j.AssertionFailedError: 
expected: "{
    "name": "Baeldung",
    "isOnline": true,
    "topics": [ "Java", "Spring", "Kotlin", "Scala", "Linux" ]
}"
but was : "{"name": "Baeldung","isOnline": true,"topics": [ "Java", "Spring", "Kotlin", "Scala", "Linux" ]}"
```

### 2.4.语义比较树

**要做一个整篇文档的对比，我们可以用`isEqualTo`** :

```java
assertJson(jsonString)
  .isEqualTo(EXPECTED_JSON_PATH);
```

在这个实例中，实际的 JSON 的字符串由`assertJson`加载，预期的 JSON 文档——由`Path`描述的文件——被加载到`isEqualTo`中。该比较是基于数据进行的。

### 2.5.不同格式

ModelAssert 还支持可以被 Jackson 转换成`JsonNode`的 Java 对象，以及`yaml`格式。

```java
Map<String, String> map = new HashMap<>();
map.put("name", "baeldung");

assertJson(map)
  .isEqualToYaml("name: baeldung");
```

对于`yaml`处理，`isEqualToYaml`方法用于指示字符串或文件的格式。这需要`assertYaml`如果来源是`yaml`:

```java
assertYaml("name: baeldung")
  .isEqualTo(map);
```

## 3.字段断言

到目前为止，我们已经看到了一些基本的断言。让我们看看 DSL 的更多内容。

### 3.1.在任何节点断言

ModelAssert 的 DSL 允许针对树中的任何节点添加几乎所有可能的条件。这是因为 JSON 树可能在任何级别包含任何类型的节点。

让我们看一些我们可能添加到示例 JSON 的根节点的断言:

```java
assertJson(jsonString)
  .isNotNull()
  .isNotNumber()
  .isObject()
  .containsKey("name");
```

因为断言对象的接口上有这些可用的方法，所以我们的 IDE 会在我们按下`“.”`键时建议我们可以添加的各种断言。

在这个例子中，我们添加了许多不必要的条件，因为最后一个条件已经暗示了一个非空对象。

最常见的是，我们从根节点使用 JSON 指针表达式，以便在树的较低节点上执行断言:

```java
assertJson(jsonString)
  .at("/topics").hasSize(5);
```

这个断言使用`hasSize`来检查`topic`字段中的数组是否有五个元素。`hasSize`方法对对象、数组和字符串进行操作。对象的大小是键的数量，字符串的大小是字符的数量，数组的大小是元素的数量。

我们需要对字段做出的大多数断言取决于字段的确切类型。当我们试图编写特定类型的断言时，我们可以使用方法`number`、`array`、`text`、`booleanNode`和`object`来进入断言的更具体的子集。这是可选的，但可以更有表现力:

```java
assertJson(jsonString)
  .at("/isOnline").booleanNode().isTrue();
```

当我们在`booleanNode`之后按下 IDE 中的`“.”`键时，我们只能看到布尔节点的自动完成选项。

### 3.2.文本节点

当我们断言文本节点时，我们可以使用`isText`来用一个精确的值进行比较。或者，我们可以使用`textContains`断言一个子串:

```java
assertJson(jsonString)
  .at("/name").textContains("ael");
```

我们也可以通过`matches`使用[正则表达式](/web/20220824101529/https://www.baeldung.com/regular-expressions-java):

```java
assertJson(jsonString)
  .at("/name").matches("[A-Z].+");
```

这个例子断言`name`以一个大写字母开始。

### 3.3.数字节点

对于数字节点，DSL 提供了一些有用的数字比较:

```java
assertJson("{count: 12}")
  .at("/count").isBetween(1, 25);
```

我们还可以指定我们期望的 Java 数值类型:

```java
assertJson("{height: 6.3}")
  .at("/height").isGreaterThanDouble(6.0);
```

`isEqualTo`方法是为整树匹配保留的，所以为了比较数值相等，我们使用`isNumberEqualTo`:

```java
assertJson("{height: 6.3}")
  .at("/height").isNumberEqualTo(6.3);
```

### 3.4.数组节点

我们可以用`isArrayContaining`测试数组的内容:

```java
assertJson(jsonString)
  .at("/topics").isArrayContaining("Scala", "Spring");
```

这将测试给定值的存在，并允许实际数组包含附加项。如果我们希望断言一个更精确的匹配，我们可以使用`isArrayContainingExactlyInAnyOrder`:

```java
assertJson(jsonString)
   .at("/topics")
   .isArrayContainingExactlyInAnyOrder("Scala", "Spring", "Java", "Linux", "Kotlin");
```

我们也可以使这种要求的确切顺序:

```java
assertJson(ACTUAL_JSON)
  .at("/topics")
  .isArrayContainingExactly("Java", "Spring", "Kotlin", "Scala", "Linux");
```

这是断言包含原始值的数组内容的好方法。当数组包含对象时，我们可能希望使用`isEqualTo `来代替。

## 4.整树匹配

虽然我们可以用多个特定于字段的条件构造断言来检查 JSON 文档中的内容，但是我们经常需要将整个文档与另一个文档进行比较。

使用`isEqualTo`方法(或`isNotEqualTo`)来比较整棵树。这可以与`at`结合，在进行比较之前移动到实际的子树:

```java
assertJson(jsonString)
  .at("/topics")
  .isEqualTo("[ \"Java\", \"Spring\", \"Kotlin\", \"Scala\", \"Linux\" ]");
```

当 JSON 包含以下数据时，整树比较可能会遇到问题:

*   相同，但顺序不同
*   包括一些无法预测的值

`where` a 方法用于定制下一个`isEqualTo`操作来绕过这些。

### 4.1.添加键顺序约束

让我们看两个看起来相同的 JSON 文档:

```java
String actualJson = "{a:{d:3, c:2, b:1}}";
String expectedJson = "{a:{b:1, c:2, d:3}}";
```

我们应该注意，这并不是严格的 JSON 格式。ModelAssert 允许我们使用 JSON 的 JavaScript 符号，以及通常引用字段名的 wire 格式。

这两个文档在`“a”`下面有完全相同的键，但是它们的顺序不同。这些断言会失败，因为 **ModelAssert 默认为严格的键顺序**。

我们可以通过添加一个`where`配置来放宽按键顺序规则:

```java
assertJson(actualJson)
  .where().keysInAnyOrder()
  .isEqualTo(expectedJson);
```

这允许树中的任何对象具有与预期文档不同的键顺序，并且仍然匹配。

我们可以将此规则局限于特定路径:

```java
assertJson(actualJson)
  .where()
    .at("/a").keysInAnyOrder()
  .isEqualTo(expectedJson);
```

这将`keysInAnyOrder`限制为根对象中的`“a”`字段。

**定制比较规则的能力使我们能够处理许多无法完全控制或预测生成的确切文档的情况**。

### 4.2.放松数组约束

如果我们的数组中值的顺序可以变化，那么我们可以放松整个比较的数组排序约束:

```java
String actualJson = "{a:[1, 2, 3, 4, 5]}";
String expectedJson = "{a:[5, 4, 3, 2, 1]}";

assertJson(actualJson)
  .where().arrayInAnyOrder()
  .isEqualTo(expectedJson);
```

或者我们可以将约束限制在一条路径上，就像我们对`keysInAnyOrder`所做的那样。

### 4.3.忽略路径

也许我们的实际文档包含一些不感兴趣或不可预测的字段。我们可以添加一个规则来忽略该路径:

```java
String actualJson = "{user:{name: \"Baeldung\", url:\"http://www.baeldung.com\"}}";
String expectedJson = "{user:{name: \"Baeldung\"}}";

assertJson(actualJson)
  .where()
    .at("/user/url").isIgnored()
  .isEqualTo(expectedJson);
```

我们应该注意到，我们表示的路径是**总是在实际的**中的 JSON 指针中。

实际中的额外字段`“url”`现在被忽略。

### 4.4.忽略任何 GUID

到目前为止，我们只添加了使用`at`的规则，以便在文档的特定位置定制比较。

`path`语法允许我们使用通配符来描述我们的规则在哪里适用。当我们将一个`at`或`path`条件添加到我们比较的`where`、**中时，我们还可以提供上面的任何字段断言**来代替与预期文档的并行比较。

假设我们有一个`id`字段出现在文档的多个地方，并且是一个我们无法预测的 GUID。

我们可以用路径规则忽略这个字段:

```java
String actualJson = "{user:{credentials:[" +
  "{id:\"a7dc2567-3340-4a3b-b1ab-9ce1778f265d\",role:\"Admin\"}," +
  "{id:\"09da84ba-19c2-4674-974f-fd5afff3a0e5\",role:\"Sales\"}]}}";
String expectedJson = "{user:{credentials:" +
  "[{id:\"???\",role:\"Admin\"}," +
  "{id:\"???\",role:\"Sales\"}]}}";

assertJson(actualJson)
  .where()
    .path("user","credentials", ANY, "id").isIgnored()
  .isEqualTo(expectedJson);
```

这里，我们的期望值可以是字段`id`的任何值，因为我们忽略了 JSON 指针从`“/user/credentials”`开始，然后有一个节点(数组索引)并以`“/id”`结束的任何字段。

### 4.5.匹配任何 GUID

忽略我们无法预测的字段是一种选择。更好的方法是通过类型来匹配这些节点，也可以通过它们必须满足的其他条件来匹配。让我们切换到强制这些 GUID 匹配 GUID 的模式，并让`id`节点出现在树的任何叶节点:

```java
assertJson(actualJson)
  .where()
    .path(ANY_SUBTREE, "id").matches(GUID_PATTERN)
  .isEqualTo(expectedJson);
```

`ANY_SUBTREE`通配符匹配路径表达式部分之间的任意数量的节点。`GUID_PATTERN`来自 ModelAssert `Patterns`类，它包含一些常见的正则表达式来匹配数字和日期戳之类的东西。

### 4.6.定制`isEqualTo`

`where`与`path`或`at`表达式的组合允许我们覆盖树中任何地方的比较。我们或者为匹配的对象或数组添加内置规则，或者指定特定的替代断言，用于比较中的单个路径或路径类。

如果我们有一个通用的配置，在各种比较中重用，我们可以将其提取到一个方法中:

```java
private static <T> WhereDsl<T> idsAreGuids(WhereDsl<T> where) {
    return where.path(ANY_SUBTREE, "id").matches(GUID_PATTERN);
}
```

然后，我们可以使用`configuredBy`将该配置添加到特定的断言中:

```java
assertJson(actualJson)
  .where()
    .configuredBy(where -> idsAreGuids(where))
  .isEqualTo(expectedJson);
```

## 5.与其他库的兼容性

ModelAssert 是为互操作性而构建的。到目前为止，我们已经看到了 AssertJ 风格的断言。这些可以有多个条件，当第一个条件不满足时，它们就会失败。

然而，有时我们需要产生一个 matcher 对象用于其他类型的测试。

### 5.1.汉堡匹配器

Hamcrest 是一个被许多工具支持的主要断言助手库。**我们可以使用 ModelAssert 的 DSL 来生成一个 Hamcrest 匹配器**:

```java
Matcher<String> matcher = json()
  .at("/name").hasValue("Baeldung");
```

`json`方法用于描述一个匹配器，它将接受一个包含 JSON 数据的`String`。我们也可以使用`jsonFile`来产生一个`Matcher`，它期望断言一个`File`的内容。ModelAssert 中的`JsonAssertions`类包含多个像这样的构建器方法来开始构建一个 Hamcrest 匹配器。

用于表达比较的 DSL 与`assertJson`相同，但是只有在使用匹配器时才会执行比较。

因此，我们可以将 ModelAssert 与 Hamcrest 的`MatcherAssert`一起使用:

```java
MatcherAssert.assertThat(jsonString, json()
  .at("/name").hasValue("Baeldung")
  .at("/topics/1").isText("Spring"));
```

### 5.2.与 Spring Mock MVC 一起使用

在 Spring Mock MVC 中使用[响应体验证时，我们可以使用 Spring 内置的`jsonPath`断言。然而，Spring 也允许我们使用](/web/20220824101529/https://www.baeldung.com/integration-testing-in-spring#2-verify-response-body) [Hamcrest 匹配器来断言作为响应内容返回的字符串](https://web.archive.org/web/20220824101529/https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/test/web/servlet/result/ContentResultMatchers.html#string-org.hamcrest.Matcher-)。这意味着我们可以使用 ModelAssert 执行复杂的内容断言。

### 5.3.与 Mockito 一起使用

Mockito 已经可以与 Hamcrest 互操作。不过，ModelAssert 也提供了一个原生的 [`ArgumentMatcher`](/web/20220824101529/https://www.baeldung.com/mockito-argument-matchers) 。这既可以用来设置存根的行为，也可以用来验证对它们的调用:

```java
public interface DataService {
    boolean isUserLoggedIn(String userDetails);
}

@Mock
private DataService mockDataService;

@Test
void givenUserIsOnline_thenIsLoggedIn() {
    given(mockDataService.isUserLoggedIn(argThat(json()
      .at("/isOnline").isTrue()
      .toArgumentMatcher())))
      .willReturn(true);

    assertThat(mockDataService.isUserLoggedIn(jsonString))
      .isTrue();

    verify(mockDataService)
      .isUserLoggedIn(argThat(json()
        .at("/name").isText("Baeldung")
        .toArgumentMatcher()));
}
```

在这个例子中，Mockito `argThat`被用于 mock 和`verify`的设置。在里面，我们为匹配器使用了 Hamcrest 风格的构建器—`json`。然后我们给它添加条件，转换成莫奇托的`ArgumentMatcher`，结尾是`toArgumentMatcher`。

## 6.结论

在本文中，我们看到了在测试中对 JSON 进行语义比较的必要性。

我们看到了如何使用 ModelAssert 在 JSON 文档中的单个节点以及整个树中构建断言。然后，我们看到了如何定制树比较，以允许不可预测或不相关的差异。

最后，我们看到了如何将 ModelAssert 与 Hamcrest 和其他库一起使用。

和往常一样，本教程的示例代码可以在 GitHub 上的[处获得。](https://web.archive.org/web/20220824101529/https://github.com/eugenp/tutorials/tree/master/libraries-testing)