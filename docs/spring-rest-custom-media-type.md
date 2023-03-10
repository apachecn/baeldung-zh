# Spring REST API 的自定义媒体类型

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-rest-custom-media-type>

## 1。概述

在本教程中，我们将看看如何定义自定义媒体类型，并通过 Spring REST 控制器来生成它们。

使用自定义媒体类型的一个很好的用例是对 API 进行版本控制。

## 2。API–版本 1

让我们从一个简单的例子开始——一个通过 id 公开单个资源的 API。

我们将从向客户端公开的资源的版本 1 开始。为了做到这一点，我们将使用一个定制的 HTTP 头–`“application/vnd.baeldung.api.v1+json”`。

客户端将通过`Accept` 头请求此自定义媒体类型。

这是我们简单的终点:

```java
@RequestMapping(
  method = RequestMethod.GET, 
  value = "/public/api/items/{id}", 
  produces = "application/vnd.baeldung.api.v1+json"
)
@ResponseBody
public BaeldungItem getItem( @PathVariable("id") String id ) {
    return new BaeldungItem("itemId1");
}
```

注意这里的`produces`参数——指定这个 API 能够处理的定制媒体类型。

现在，`BaeldungItem`资源——它只有一个字段—`itemId`:

```java
public class BaeldungItem {
    private String itemId;

    // standard getters and setters
}
```

最后但同样重要的是，让我们为端点编写一个集成测试:

```java
@Test
public void givenServiceEndpoint_whenGetRequestFirstAPIVersion_then200() {
    given()
      .accept("application/vnd.baeldung.api.v1+json")
    .when()
      .get(URL_PREFIX + "/public/api/items/1")
    .then()
      .contentType(ContentType.JSON).and().statusCode(200);
}
```

## 3.API–版本 2

现在让我们假设我们需要改变我们用资源向客户机公开的细节。

我们曾经公开一个原始 id——假设现在我们需要隐藏它，而公开一个名称，以获得更多的灵活性。

理解这种变化不是向后兼容的，这一点很重要；基本上，这是一个突破性的变化。

这是我们新的资源定义:

```java
public class BaeldungItemV2 {
    private String itemName;

    // standard getters and setters
}
```

因此，我们在这里需要做的是——将我们的 API 迁移到第二个版本。

我们将通过**创建下一版本的自定义媒体类型**并定义一个新端点来实现这一点:

```java
@RequestMapping(
  method = RequestMethod.GET, 
  value = "/public/api/items/{id}", 
  produces = "application/vnd.baeldung.api.v2+json"
)
@ResponseBody
public BaeldungItemV2 getItemSecondAPIVersion(@PathVariable("id") String id) {
    return new BaeldungItemV2("itemName");
}
```

因此，我们现在有了完全相同的端点，但是能够处理新的 V2 操作。

当客户端请求`“application/vnd.baeldung.api.v1+json”`时，Spring 将委托给旧的操作，客户端将收到一个带有`itemId`字段(V1)的`BaeldungItem`。

但是当客户端现在将`Accept`头设置为`“application/vnd.baeldung.api.v2+json” –` 时，他们将正确地点击新操作，并通过`itemName`字段(V2)取回资源:

```java
@Test
public void givenServiceEndpoint_whenGetRequestSecondAPIVersion_then200() {
    given()
      .accept("application/vnd.baeldung.api.v2+json")
    .when()
      .get(URL_PREFIX + "/public/api/items/2")
    .then()
      .contentType(ContentType.JSON).and().statusCode(200);
}
```

注意测试是如何相似的，但是使用不同的`Accept`标题。

## 4.类级别的自定义媒体类型

最后，让我们来谈谈媒体类型的班级范围的定义，这也是可能的:

```java
@RestController
@RequestMapping(
  value = "/", 
  produces = "application/vnd.baeldung.api.v1+json"
)
public class CustomMediaTypeController
```

正如所料， `@RequestMapping`注释很容易在类级别上工作，并允许我们指定`value`、`produces`和`consumes`参数。

## 5.结论

本文举例说明了定义自定义媒体类型在公共 API 版本控制中的作用。

所有这些例子和代码片段的实现都可以在 GitHub 项目中找到。