# 探索 Spring Boot TestRestTemplate

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-boot-testresttemplate>

## 1。概述

这篇文章探究了 Spring Boot `TestRestTemplate`。它可以被视为[rest template](/web/20220913185116/https://www.baeldung.com/rest-template)指南的后续，我们强烈建议在关注`TestRestTemplate`之前阅读该指南。`TestRestTemplate`可以被认为是`RestTemplate`的一个有吸引力的替代品。

## 2。Maven 依赖关系

要使用`TestRestTemplate`，您需要有一个适当的依赖关系，如:

```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-test</artifactId>
    <version>2.2.2.RELEASE</version>
</dependency>
```

你可以在 [Maven Central](https://web.archive.org/web/20220913185116/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.springframework.boot%22%20AND%20a%3A%22spring-boot-test%22) 上找到最新版本。

## 3。`TestRestTemplate`和`RestTemplate`

这两种客户端都非常适合编写集成测试，并且能够很好地处理与 HTTP APIs 的通信。

例如，它们为我们提供了相同的方法标准方法、头和其他 HTTP 构造。

并且所有这些操作都在 RestTemplate 指南中有很好的描述，所以我们在这里不再赘述。

下面是一个简单的 GET 请求示例:

```java
TestRestTemplate testRestTemplate = new TestRestTemplate();
ResponseEntity<String> response = testRestTemplate.
  getForEntity(FOO_RESOURCE_URL + "/1", String.class);

Assertions.assertEquals(response.getStatusCode(), HttpStatus.OK); 
```

尽管这两个类非常相似，`TestRestTemplate`并没有扩展`RestTemplate`，而是提供了一些非常令人兴奋的新特性。

## 4。`TestRestTemplate`有什么新内容？

### 4.1。具有基本认证证书的构造器

`TestRestTemplate`提供了一个构造函数，我们可以用它**创建一个带有基本认证**的指定凭证的模板。

使用此实例执行的所有请求都将使用提供的凭据进行身份验证:

```java
TestRestTemplate testRestTemplate
 = new TestRestTemplate("user", "passwd");
ResponseEntity<String> response = testRestTemplate.
  getForEntity(URL_SECURED_BY_AUTHENTICATION, String.class);

Assertions.assertEquals(response.getStatusCode(), HttpStatus.OK); 
```

### 4.2。带`HttpClientOption`的建造师

`TestRestTemplate`还使我们能够使用`HttpClientOption which`来定制底层的 Apache HTTP 客户端，它是`TestRestTemplate`中的一个 enum，具有以下选项:`ENABLE_COOKIES, ENABLE_REDIRECTS`和`SSL`。

让我们看一个简单的例子:

```java
TestRestTemplate testRestTemplate = new TestRestTemplate("user", 
  "passwd", TestRestTemplate.HttpClientOption.ENABLE_COOKIES);
ResponseEntity<String> response = testRestTemplate.
  getForEntity(URL_SECURED_BY_AUTHENTICATION, String.class);

Assertions.assertEquals(response.getStatusCode(), HttpStatus.OK);
```

在上面的例子中，我们将选项与基本认证一起使用。

如果我们不需要认证，我们仍然可以用一个简单的构造函数创建一个模板:

`TestRestTemplate(TestRestTemplate.HttpClientOption.ENABLE_COOKIES)`

### 4.3。新方法

构造函数不仅可以用指定的凭据创建模板。我们还可以在创建模板后添加凭证。`TestRestTemplate`为我们提供了一个方法`withBasicAuth()` ,它将凭证添加到一个已经存在的模板中:

```java
TestRestTemplate testRestTemplate = new TestRestTemplate();
ResponseEntity<String> response = testRestTemplate.withBasicAuth(
  "user", "passwd").getForEntity(URL_SECURED_BY_AUTHENTICATION, 
  String.class);

Assertions.assertEquals(response.getStatusCode(), HttpStatus.OK); 
```

## 5。同时使用 **`TestRestTemplate`和`RestTemplate`**

`TestRestTemplate` 可以作为`RestTemplate`的包装器，例如，如果我们因为处理遗留代码而被迫使用它。您可以在下面看到如何创建这样一个简单的包装器:

```java
RestTemplateBuilder restTemplateBuilder = new RestTemplateBuilder();
restTemplateBuilder.configure(restTemplate);
TestRestTemplate testRestTemplate = new TestRestTemplate(restTemplateBuilder);
ResponseEntity<String> response = testRestTemplate.getForEntity(
  FOO_RESOURCE_URL + "/1", String.class);

Assertions.assertEquals(response.getStatusCode(), HttpStatus.OK);
```

## 6。结论

`TestRestTemplate`不是`RestTemplate`的扩展，而是简化集成测试并在测试过程中方便认证的替代方案。它有助于 Apache HTTP 客户端的定制，但也可以用作`RestTemplate`的包装器。

你可以在 GitHub 上查看本文[中提供的例子。](https://web.archive.org/web/20220913185116/https://github.com/eugenp/tutorials/tree/master/spring-web-modules/spring-resttemplate)