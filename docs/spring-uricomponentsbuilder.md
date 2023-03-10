# 春季 UriComponentsBuilder 指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-uricomponentsbuilder>

## 1。简介

在本教程中，我们将关注 Spring `UriComponentsBuilder.` 更具体地说，我们将描述各种实际的实现示例。

构建器与`UriComponents`类协同工作——URI 组件的不可变容器。

一个新的`UriComponentsBuilder`类通过提供对准备 URI 的所有方面的细粒度控制来帮助创建`UriComponents`实例，包括构造、从模板变量扩展和编码。

## 2。Maven 依赖关系

为了使用构建器，我们需要在我们的`pom.xml`的`dependencies`中包含以下部分:

```java
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-web</artifactId>
    <version>5.2.2.RELEASE</version>
</dependency> 
```

最新版本可以在这里找到[。](https://web.archive.org/web/20220627143654/https://search.maven.org/search?q=a:spring-web%20AND%20g:org.springframework)

这个依赖项只涉及 Spring Web，所以不要忘记为完整的 Web 应用程序添加`spring-context`。

我们当然也需要为项目设置日志记录——更多信息请点击[这里](/web/20220627143654/https://www.baeldung.com/java-logging-intro)。

## 3。用例

`UriComponentsBuilder`有许多实际的用例，从相应的 URI 组件中不允许的字符的上下文编码开始，到动态替换部分 URL 结束。

`UriComponentsBuilder` 的最大优势之一是**我们可以将其直接注入控制器方法**:

```java
@RequestMapping(method = RequestMethod.POST)
public ResponseEntity createCustomer(UriComponentsBuilder builder) {
    // implementation
}
```

让我们开始逐一描述有用的例子。我们将使用 JUnit 框架立即测试我们的实现。

### 3.1。构建 URI

先说最简单的。我们想使用`UriComponentsBuilder`来创建简单的链接:

```java
@Test
public void constructUri() {
    UriComponents uriComponents = UriComponentsBuilder.newInstance()
      .scheme("http").host("www.baeldung.com").path("/junit-5").build();

    assertEquals("/junit-5", uriComponents.toUriString());
}
```

正如我们可能观察到的，我们创建了一个新的`UriComponentsBuilder`实例，然后我们提供了 scheme 类型、主机和到请求目的地的路径。

当我们想重定向到网站的其他部分/链接时，这个简单的例子可能会很有用。

### 3.2。构建编码 URI

除了创建一个简单的链接之外，我们可能还想对最终结果进行编码。让我们看看实际情况:

```java
@Test
public void constructUriEncoded() {
    UriComponents uriComponents = UriComponentsBuilder.newInstance()
      .scheme("http").host("www.baeldung.com").path("/junit 5").build().encode();

    assertEquals("/junit%205", uriComponents.toUriString());
}
```

本例中的不同之处在于，我们想要在单词`junit`和数字`5`之间添加空格。根据 [RFC 3986](https://web.archive.org/web/20220627143654/https://www.ietf.org/rfc/rfc3986.txt) ，这是不可能的。我们需要使用`encode()`方法对链接进行编码以获得有效的结果。

### 3.3。从模板构建 URI

URI 的大多数组件中都允许使用 URI 模板，但是它们的值仅限于特定的元素，我们称之为模板。让我们看看这个例子来阐明:

```java
@Test
public void constructUriFromTemplate() {
    UriComponents uriComponents = UriComponentsBuilder.newInstance()
      .scheme("http").host("www.baeldung.com").path("/{article-name}")
      .buildAndExpand("junit-5");

    assertEquals("/junit-5", uriComponents.toUriString());
}
```

这个例子的不同之处在于我们声明路径的方式以及我们如何构建最终的 URI。将被关键字替换的模板在`path()`方法中用括号–`{…},` 表示。用于生成最终链接的关键字在名为`buildAndExpand(…)`的方法中使用。

请注意，可能会有一个以上的关键字被替换。此外，到 URI 的路径可以是相对的。

当我们想要将模型对象传递给 Spring 控制器时，这个例子将非常有用，我们将基于它来构建一个 URI。

### 3.4。用查询参数构造 URI

另一个非常有用的例子是用查询参数创建 URI。

我们需要使用来自`UriComponentsBuilder`的`query()`来指定 URI 查询参数。让我们看下面的例子:

```java
@Test
public void constructUriWithQueryParameter() {
    UriComponents uriComponents = UriComponentsBuilder.newInstance()
      .scheme("http").host("www.google.com")
      .path("/").query("q={keyword}").buildAndExpand("baeldung");

     assertEquals("http://www.google.com/?q=baeldung", uriComponents.toUriString());
}
```

该查询将被添加到链接的主要部分。我们可以提供多个查询参数，使用括号`{…}.` 它们将被名为`buildAndExpand(…)`的方法中的关键字所替换。

**`UriComponentsBuilder`的这个实现可能用于构建——例如——REST API 的查询语言。**

### 3.5。用正则表达式扩展 URI

最后一个例子展示了带有正则表达式验证的 URI 的构造。只有正则表达式验证成功，我们才能展开`uriComponents` :

```java
@Test
public void expandWithRegexVar() {
    String template = "/myurl/{name:[a-z]{1,5}}/show";
    UriComponents uriComponents = UriComponentsBuilder.fromUriString(template)
      .build();
    uriComponents = uriComponents.expand(Collections.singletonMap("name", "test"));

    assertEquals("/myurl/test/show", uriComponents.getPath());
}
```

在上面提到的例子中，我们可以看到，链接的中间部分只需要有从`a-z`开始的字母和在`1-5`之间的一个范围内的长度。

同样，我们使用`singletonMap`，用值`test`替换关键字`name`。

当我们允许用户动态地指定链接时，这个例子特别有用，但是我们想提供一种安全性，在我们的 web 应用程序中只有有效的链接才起作用。

## 4。结论

本教程展示了`UriComponentsBuilder`的有用例子。

`UriComponentsBuilder`的主要优势是使用 URI 模板变量的灵活性，以及将其直接注入 Spring 控制器方法的可能性。

所有示例和配置都可以在 [GitHub](https://web.archive.org/web/20220627143654/https://github.com/eugenp/tutorials/tree/master/spring-web-modules/spring-rest-http-2) 上找到。