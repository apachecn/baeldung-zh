# 探索 Spring 5 WebFlux URL 匹配

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-5-mvc-url-matching>

## 1。概述

Spring 5 为[带来了一个新的](https://web.archive.org/web/20220626193507/https://jira.spring.io/browse/SPR-14544) `PathPatternParser`用于解析 URI 模板模式`.` 这是对之前使用的`AntPathMatcher`的替代。

`AntPathMatcher`是 Ant 风格的路径模式匹配的实现。`PathPatternParser` 将路径分解成一个`PathElements`的链表。这个`PathElements`链被`PathPattern`类用来快速匹配模式。

在`PathPatternParser`中，还引入了对新的 URI 变量语法的支持。

在本文中，**我们将介绍 Spring 5.0 WebFlux** 中引入的新的/更新的 URL 模式匹配器，以及自 Spring 的旧版本以来就存在的那些。

## 2。Spring 5.0 中新的 URL 模式匹配器

Spring 5.0 版本增加了一个非常容易使用的 URI 变量语法:{*foo}来捕获模式末尾的任意数量的路径段。

### 2.1。使用处理程序方法的 URI 变量语法{*foo}

让我们看看 URI 变量模式 `{*foo}`的一个例子，另一个例子使用了`@GetMapping`和一个处理程序方法。无论我们在`“/spring5”`后的路径中给出什么，都将存储在路径变量“id”中:

```java
@GetMapping("/spring5/{*id}")
public String URIVariableHandler(@PathVariable String id) {
    return id;
}
```

```java
@Test
public void whenMultipleURIVariablePattern_thenGotPathVariable() {

    client.get()
      .uri("/spring5/baeldung/tutorial")
      .exchange()
      .expectStatus()
      .is2xxSuccessful()
      .expectBody()
      .equals("/baeldung/tutorial");

    client.get()
      .uri("/spring5/baeldung")
      .exchange()
      .expectStatus()
      .is2xxSuccessful()
      .expectBody()
      .equals("/baeldung");
}
```

### 2.2。URI 变量语法{*foo}使用`RouterFunction`

让我们看一个使用`RouterFunction`的新 URI 变量路径模式的例子:

```java
private RouterFunction<ServerResponse> routingFunction() {
    return route(GET("/test/{*id}"), 
      serverRequest -> ok().body(fromValue(serverRequest.pathVariable("id"))));
}
```

在这种情况下，我们在“/test”之后写的任何路径都将被捕获到路径变量“id”中。因此，它的测试用例可能是:

```java
@Test
public void whenMultipleURIVariablePattern_thenGotPathVariable() 
  throws Exception {

    client.get()
      .uri("/test/ab/cd")
      .exchange()
      .expectStatus()
      .isOk()
      .expectBody(String.class)
      .isEqualTo("/ab/cd");
}
```

### 2.3。使用 URI 变量语法{*foo}访问资源

如果我们想要访问资源，那么我们需要编写与前面例子中相似的路径模式。

所以假设我们的模式是:`“/files/{*filepaths}”.`在这种情况下，如果路径是`/files/hello.txt,` ，那么路径变量`“filepaths”`的值将是“/hello.txt”，反之，如果路径是`/files/test/test.txt,` ， `“filepaths”`=“/test/test . txt”的值。

我们访问`/files/`目录下文件资源的路由函数:

```java
private RouterFunction<ServerResponse> routingFunction() { 
    return RouterFunctions.resources(
      "/files/{*filepaths}", 
      new ClassPathResource("files/"))); 
} 
```

假设我们的文本文件 `hello.txt`和`test.txt`分别包含`“hello”`和`“test”`。这可以用一个 JUnit 测试用例来演示:

```java
@Test 
public void whenMultipleURIVariablePattern_thenGotPathVariable() 
  throws Exception { 
      client.get() 
        .uri("/files/test/test.txt") 
        .exchange() 
        .expectStatus() 
        .isOk() 
        .expectBody(String.class) 
        .isEqualTo("test");

      client.get() 
        .uri("/files/hello.txt") 
        .exchange() 
        .expectStatus() 
        .isOk() 
        .expectBody(String.class) 
        .isEqualTo("hello"); 
}
```

## 3。以前版本的现有 URL 模式

现在让我们来看一看所有其他的 URL 模式匹配器，它们已经被老版本的 Spring 所支持。所有这些模式都适用于`RouterFunction`和`@GetMapping`的处理程序方法。

### 3.1。'?'恰好匹配一个字符

如果我们将路径模式指定为:`“/t?` st `“,`，这将匹配路径如:`“/test”`和`“/tast”,`，但不匹配`“/tst”`和`“/teest”.`

使用`RouterFunction`的示例代码及其 JUnit 测试用例:

```java
private RouterFunction<ServerResponse> routingFunction() { 
    return route(GET("/t?st"), 
      serverRequest -> ok().body(fromValue("Path /t?st is accessed"))); 
}

@Test
public void whenGetPathWithSingleCharWildcard_thenGotPathPattern()   
  throws Exception {

      client.get()
        .uri("/test")
        .exchange()
        .expectStatus()
        .isOk()
        .expectBody(String.class)
        .isEqualTo("Path /t?st is accessed");
} 
```

### 3.2。* '匹配路径段中的 0 个或多个字符

如果我们将路径模式指定为:`“/baeldung/*Id”,`这将匹配路径模式`like:”/baeldung/Id”, “/baeldung/tutorialId”,` "/baeldung/articleId "，等等:

```java
private RouterFunction<ServerResponse> routingFunction() { 
    returnroute(
      GET("/baeldung/*Id"), 
      serverRequest -> ok().body(fromValue("/baeldung/*Id path was accessed"))); }

@Test
public void whenGetMultipleCharWildcard_thenGotPathPattern() 
  throws Exception {
      client.get()
        .uri("/baeldung/tutorialId")
        .exchange()
        .expectStatus()
        .isOk()
        .expectBody(String.class)
        .isEqualTo("/baeldung/*Id path was accessed");
} 
```

### 3.3。 '匹配 0 个或多个路径段，直到路径结束**

在这种情况下，模式匹配不限于单个路径段。如果我们将模式指定为`“/resources/**”,`，它会将所有路径匹配到`“/resources/”:`之后的任意数量的路径段

```java
private RouterFunction<ServerResponse> routingFunction() { 
    return RouterFunctions.resources(
      "/resources/**", 
      new ClassPathResource("resources/"))); 
}

@Test
public void whenAccess_thenGot() throws Exception {
    client.get()
      .uri("/resources/test/test.txt")
      .exchange()
      .expectStatus()
      .isOk()
      .expectBody(String.class)
      .isEqualTo("content of file test.txt");
} 
```

### 3.4。`‘{baeldung:[a-z]+}'`路径变量中的正则表达式

我们还可以为 path 变量的值指定一个正则表达式。因此，如果我们的模式类似于`“/{baeldung:[a-z]+}”,`，路径变量`“baeldung”`的值将是匹配给定正则表达式的任何路径段:

```java
private RouterFunction<ServerResponse> routingFunction() { 
    return route(GET("/{baeldung:[a-z]+}"), 
      serverRequest ->  ok()
        .body(fromValue("/{baeldung:[a-z]+} was accessed and "
        + "baeldung=" + serverRequest.pathVariable("baeldung")))); 
}

@Test
public void whenGetRegexInPathVarible_thenGotPathVariable() 
  throws Exception {

      client.get()
        .uri("/abcd")
        .exchange()
        .expectStatus()
        .isOk()
        .expectBody(String.class)
        .isEqualTo("/{baeldung:[a-z]+} was accessed and "
          + "baeldung=abcd");
} 
```

### 3.5。`‘/{var1}_{var2}'`同一路径段中的多个路径变量

Spring 5 确保了只有在用分隔符分隔时，单个路径段中才允许多个路径变量。只有这样，Spring 才能区分两个不同的路径变量:

```java
private RouterFunction<ServerResponse> routingFunction() { 

    return route(
      GET("/{var1}_{var2}"),
      serverRequest -> ok()
        .body(fromValue( serverRequest.pathVariable("var1") + " , " 
        + serverRequest.pathVariable("var2"))));
 }

@Test
public void whenGetMultiplePathVaribleInSameSegment_thenGotPathVariables() 
  throws Exception {
      client.get()
        .uri("/baeldung_tutorial")
        .exchange()
        .expectStatus()
        .isOk()
        .expectBody(String.class)
        .isEqualTo("baeldung , tutorial");
}
```

## 4.结论

在本文中，我们介绍了 Spring 5 中新的 URL 匹配器，以及 Spring 旧版本中可用的 URL 匹配器。

和往常一样，我们讨论的所有例子的实现都可以在 GitHub 上找到[。](https://web.archive.org/web/20220626193507/https://github.com/eugenp/tutorials/tree/master/spring-5-reactive-modules/spring-5-reactive)