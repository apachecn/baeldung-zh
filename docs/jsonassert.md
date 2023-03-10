# JSONassert 简介

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/jsonassert>

## 1。概述

在本文中，我们将看看 [JSONAssert 库](https://web.archive.org/web/20220628093343/http://jsonassert.skyscreamer.org/)——一个专注于理解 JSON 数据并使用该数据编写复杂的 [JUnit](https://web.archive.org/web/20220628093343/http://junit.org/junit4/) 测试的库。

## 2。Maven 依赖关系

首先，让我们添加 Maven 依赖项:

```java
<dependency>
    <groupId>org.skyscreamer</groupId>
    <artifactId>jsonassert</artifactId>
    <version>1.5.0</version>
</dependency>
```

请点击查看最新版本的库[。](https://web.archive.org/web/20220628093343/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.skyscreamer%22%20AND%20a%3A%22jsonassert%22)

## 3。使用简单的 JSON 数据

### 3.1。使用`LENIENT`模式

让我们从一个简单的 JSON 字符串比较开始我们的测试:

```java
String actual = "{id:123, name:\"John\"}";
JSONAssert.assertEquals(
  "{id:123,name:\"John\"}", actual, JSONCompareMode.LENIENT);
```

测试将作为预期的 JSON 字符串通过，而实际的 JSON 字符串是相同的。

比较**模式`LENIENT`意味着即使实际 JSON 包含扩展字段，测试仍然会通过:**

```java
String actual = "{id:123, name:\"John\", zip:\"33025\"}";
JSONAssert.assertEquals(
  "{id:123,name:\"John\"}", actual, JSONCompareMode.LENIENT);
```

正如我们所看到的，`real`变量包含一个额外的字段`zip`，它不在预期的`String`中。尽管如此，考验还是会过去的。

这个概念在应用程序开发中很有用。这意味着我们的 API 可以增长，根据需要返回额外的字段，而不会破坏现有的测试。

### 3.2。使用`STRICT`模式

使用`STRICT`比较模式可以很容易地改变前一小节中提到的行为:

```java
String actual = "{id:123,name:\"John\"}";
JSONAssert.assertNotEquals(
  "{name:\"John\"}", actual, JSONCompareMode.STRICT);
```

请注意上面例子中`assertNotEquals()`的使用。

### 3.3。用`Boolean`代替`JSONCompareMode`

也可以使用重载方法定义比较模式，该方法采用`boolean`而不是`JSONCompareMode`，其中`LENIENT = false`和`STRICT = true`:

```java
String actual = "{id:123,name:\"John\",zip:\"33025\"}";
JSONAssert.assertEquals(
  "{id:123,name:\"John\"}", actual, JSONCompareMode.LENIENT);
JSONAssert.assertEquals(
  "{id:123,name:\"John\"}", actual, false);

actual = "{id:123,name:\"John\"}";
JSONAssert.assertNotEquals(
  "{name:\"John\"}", actual, JSONCompareMode.STRICT);
JSONAssert.assertNotEquals(
  "{name:\"John\"}", actual, true);
```

### 3.4。逻辑比较

如前所述，`JSONAssert`对数据进行逻辑比较。这意味着在处理 JSON 对象时，元素的顺序并不重要:

```java
String result = "{id:1,name:\"John\"}";
JSONAssert.assertEquals(
  "{name:\"John\",id:1}", result, JSONCompareMode.STRICT);
JSONAssert.assertEquals(
  "{name:\"John\",id:1}", result, JSONCompareMode.LENIENT);
```

无论严格与否，上述测试在两种情况下都会通过。

逻辑比较的另一个示例可以通过对相同的值使用不同的类型来演示:

```java
JSONObject expected = new JSONObject();
JSONObject actual = new JSONObject();
expected.put("id", Integer.valueOf(12345));
actual.put("id", Double.valueOf(12345));

JSONAssert.assertEquals(expected, actual, JSONCompareMode.LENIENT);
```

这里要注意的第一件事是，我们使用了`JSONObject`而不是字符串，就像我们在前面的例子中所做的那样。接下来的事情是，我们已经用`Integer`表示`expected`，用`Double`表示`actual`。不管类型如何，测试都将通过，因为它们的逻辑值 12345 是相同的。

即使在我们有嵌套对象表示的情况下，这个库也能很好地工作:

```java
String result = "{id:1,name:\"Juergen\", 
  address:{city:\"Hollywood\", state:\"LA\", zip:91601}}";
JSONAssert.assertEquals("{id:1,name:\"Juergen\", 
  address:{city:\"Hollywood\", state:\"LA\", zip:91601}}", result, false);
```

### 3.5。带有用户指定消息的断言

所有的`assertEquals()`和`assertNotEquals()`方法都接受一个`String`消息作为第一个参数。该消息通过在测试失败的情况下提供有意义的消息，为我们的测试用例提供了一些定制:

```java
String actual = "{id:123,name:\"John\"}";
String failureMessage = "Only one field is expected: name";
try {
    JSONAssert.assertEquals(failureMessage, 
      "{name:\"John\"}", actual, JSONCompareMode.STRICT);
} catch (AssertionError ae) {
    assertThat(ae.getMessage()).containsIgnoringCase(failureMessage);
}
```

在任何失败的情况下，整个错误消息将更有意义:

```java
Only one field is expected: name 
Unexpected: id
```

第一行是用户指定的消息，第二行是库提供的附加消息。

## 4。使用 JSON 数组

与 JSON 对象相比，JSON 数组的比较规则略有不同。

### 4.1。数组中元素的顺序

第一个区别是**在`STRICT`比较模式**中，数组中元素的顺序必须完全相同。然而，对于`LENIENT`比较模式，顺序并不重要:

```java
String result = "[Alex, Barbera, Charlie, Xavier]";
JSONAssert.assertEquals(
  "[Charlie, Alex, Xavier, Barbera]", result, JSONCompareMode.LENIENT);
JSONAssert.assertEquals(
  "[Alex, Barbera, Charlie, Xavier]", result, JSONCompareMode.STRICT);
JSONAssert.assertNotEquals(
  "[Charlie, Alex, Xavier, Barbera]", result, JSONCompareMode.STRICT);
```

这在 API 返回排序元素数组的场景中非常有用，我们希望验证响应是否排序。

### 4.2。数组中的扩展元素

另一个区别是在处理 JSON 数组时不允许使用**扩展元素:**

```java
String result = "[1,2,3,4,5]";
JSONAssert.assertEquals(
  "[1,2,3,4,5]", result, JSONCompareMode.LENIENT);
JSONAssert.assertNotEquals(
  "[1,2,3]", result, JSONCompareMode.LENIENT);
JSONAssert.assertNotEquals(
  "[1,2,3,4,5,6]", result, JSONCompareMode.LENIENT);
```

上面的例子清楚地表明，即使使用`LENIENT`比较模式，预期数组中的项目也必须与真实数组中的项目完全匹配。添加或删除，甚至一个元素，都会导致失败。

### 4.3。阵列特定操作

我们还有一些其他的技术来进一步验证数组的内容。

假设我们想要验证数组的大小。这可以通过使用具体的语法作为期望值来实现:

```java
String names = "{names:[Alex, Barbera, Charlie, Xavier]}";
JSONAssert.assertEquals(
  "{names:[4]}", 
  names, 
  new ArraySizeComparator(JSONCompareMode.LENIENT));
```

`String` `“{names:[4]}”`指定数组的预期大小。

让我们来看看另一种比较技术:

```java
String ratings = "{ratings:[3.2,3.5,4.1,5,1]}";
JSONAssert.assertEquals(
  "{ratings:[1,5]}", 
  ratings, 
  new ArraySizeComparator(JSONCompareMode.LENIENT));
```

上面的示例验证了数组中所有元素的值必须在[1，5]之间，包括 1 和 5。如果有任何值小于 1 或大于 5，上述测试将失败。

## 5。高级对比示例

考虑我们的 API 返回多个`id`的用例，每个都是一个`Integer`值。这意味着所有的`id`都可以用一个简单的正则表达式“`\d`来验证。

上述正则表达式可以与一个`CustomComparator`组合，并应用于所有`id`的所有值。如果任何一个`id`与正则表达式不匹配，测试将失败:

```java
JSONAssert.assertEquals("{entry:{id:x}}", "{entry:{id:1, id:2}}", 
  new CustomComparator(
  JSONCompareMode.STRICT, 
  new Customization("entry.id", 
  new RegularExpressionValueMatcher<Object>("\\d"))));

JSONAssert.assertNotEquals("{entry:{id:x}}", "{entry:{id:1, id:as}}", 
  new CustomComparator(JSONCompareMode.STRICT, 
  new Customization("entry.id", 
  new RegularExpressionValueMatcher<Object>("\\d"))));
```

上例中的“`{id:x}`”只是一个占位符——`x`可以被任何东西替换。因为这是正则表达式模式'【T2]'将被应用的地方。由于`id`本身在另一个字段`entry`内，所以`Customization`指定了`id`的位置，这样`CustomComparator`就可以进行比较。

## 6。结论

在这篇简短的文章中，我们研究了 JSONAssert 可以发挥作用的各种场景。我们从一个超级简单的例子开始，然后转向更复杂的比较。

当然，和往常一样，这里讨论的所有例子的完整源代码可以在 GitHub 上找到[。](https://web.archive.org/web/20220628093343/https://github.com/eugenp/tutorials/tree/master/libraries-testing)