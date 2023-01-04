# http messagenatwritableexception:没有预设内容类型的[class …]转换器

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-no-converter-with-preset>

## 1.概观

在这篇短文中，我们将仔细研究 Spring 异常，`“HttpMessageNotWritableException: no converter for [class …] with preset Content-Type”`。

首先，我们将揭示异常背后的主要原因。然后，我们将深入兔子洞，看看如何使用实际示例再现它，最后，如何解决它。

## 2.原因

在深入研究细节之前，让我们试着理解这个异常意味着什么。

异常的堆栈跟踪说明了一切:它告诉我们 Spring **没有找到合适的** [`**HttpMessageConverter**`](/web/20220524061721/https://www.baeldung.com/spring-httpmessageconverter-rest) **能够将 Java 对象转换成 HTTP 响应** `.`

基本上，Spring 依靠`“Accept”`头来检测它需要响应的媒体类型。

因此，**使用没有预注册消息转换器的媒体类型将导致 Spring 失败，并出现异常**。

## 3.再现异常

现在我们知道了是什么导致 Spring 抛出我们的异常，让我们看看如何使用一个实际的例子来重现它。

让我们创建一个处理程序方法，假装指定一个没有注册`HttpMessageConverter`的媒体类型(用于响应)。

例如，让我们使用`APPLICATION_XML_VALUE` 或`“application/xml”`:

```java
@GetMapping(value = "/student/v3/{id}", produces = MediaType.APPLICATION_XML_VALUE)
public ResponseEntity<Student> getV3(@PathVariable("id") int id) {
    return ResponseEntity.ok(new Student(id, "Robert", "Miller", "BB"));
}
```

接下来，让我们向`http://localhost:8080/api/student/v3/1` 发送请求，看看会发生什么:

```java
curl http://localhost:8080/api/student/v3/1
```

端点发回以下响应:

```java
{"timestamp":"2022-02-01T18:23:37.490+00:00","status":500,"error":"Internal Server Error","path":"/api/student/v3/1"}
```

事实上，查看日志，Spring 失败了，出现了`HttpMessageNotWritableException`异常:

```java
[org.springframework.http.converter.HttpMessageNotWritableException: No converter for [class com.baeldung.boot.noconverterfound.model.Student] with preset Content-Type 'null']
```

所以，抛出异常是因为**没有`HttpMessageConverter` 能够将`Student`对象与 XML** 进行封送和解封。

最后，让我们创建一个测试用例来确认 Spring 抛出带有指定消息的`HttpMessageNotWritableException` :

```java
@Test
public void whenConverterNotFound_thenThrowException() throws Exception {
    String url = "/api/student/v3/1";

    this.mockMvc.perform(get(url))
      .andExpect(status().isInternalServerError())
      .andExpect(result -> assertThat(result.getResolvedException()).isInstanceOf(HttpMessageNotWritableException.class))
      .andExpect(result -> assertThat(result.getResolvedException()
        .getMessage()).contains("No converter for [class com.baeldung.boot.noconverterfound.model.Student] with preset Content-Type"));
}
```

## 4.**解决方案**

只有一种方法可以解决这个异常——使用注册了消息转换器的媒体类型。

Spring Boot 依靠自动配置来注册[内置消息转换器](/web/20220524061721/https://www.baeldung.com/spring-httpmessageconverter-rest#2-the-default-message-converters)。

例如，如果类路径中存在 [Jackson 2](/web/20220524061721/https://www.baeldung.com/jackson) 依赖项，那么**会自动注册`MappingJackson2HttpMessageConverter `。**

话虽如此，并且知道 Spring Boot 在 [web starter](/web/20220524061721/https://www.baeldung.com/spring-boot-starters#Starter) 中包含了 Jackson，让我们用`APPLICATION_JSON_VALUE`媒体类型创建一个新的端点:

```java
@GetMapping(value = "/student/v2/{id}", produces = MediaType.APPLICATION_JSON_VALUE)
public ResponseEntity<Student> getV2(@PathVariable("id") int id) {
    return ResponseEntity.ok(new Student(id, "Kevin", "Cruyff", "AA"));
}
```

现在，让我们创建一个测试用例来确认一切正常:

```java
@Test
public void whenJsonConverterIsFound_thenReturnResponse() throws Exception {
    String url = "/api/student/v2/1";

    this.mockMvc.perform(get(url))
      .andExpect(status().isOk())
      .andExpect(content().json("{'id':1,'firstName':'Kevin','lastName':'Cruyff', 'grade':'AA'}"));
}
```

正如我们所看到的，Spring 没有抛出`HttpMessageNotWritableException`，这要感谢`MappingJackson2HttpMessageConverter`，它在引擎盖下处理了`Student`对象到 JSON 的转换。

## 5.结论

在这个简短的教程中，我们详细讨论了是什么原因导致 Spring 抛出`“HttpMessageNotWritableException No converter for [class …] with preset Content-Type”`。

在这个过程中，我们展示了如何产生异常以及如何在实践中修复它。

与往常一样，GitHub 上的[提供了示例的完整源代码。](https://web.archive.org/web/20220524061721/https://github.com/eugenp/tutorials/tree/master/spring-boot-modules/spring-boot-data-2)