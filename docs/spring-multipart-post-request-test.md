# 测试 Spring 多部分 POST 请求

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-multipart-post-request-test>

## 1.概观

在这个快速教程中，我们将看到如何在 Spring 中使用 [`MockMvc`](https://web.archive.org/web/20220626193709/https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/test/web/servlet/MockMvc.html) 测试多部分 POST 请求。

## 2.Maven 依赖性

在我们开始之前，让我们在我们的`pom.xml`中添加最新的 [JUnit](https://web.archive.org/web/20220626193709/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22junit%22%20AND%20a%3A%22junit%22) 和 [Spring test](https://web.archive.org/web/20220626193709/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.springframework%22%20AND%20a%3A%22spring-test%22) 依赖项:

```java
<dependency>
    <groupId>junit</groupId>
    <artifactId>junit</artifactId>
    <version>4.13</version>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-test</artifactId>
    <version>5.1.16.RELEASE</version>
    <scope>test</scope>
</dependency>
```

## 3.测试多部分 POST 请求

让我们在 REST 控制器中创建一个简单的端点:

```java
@PostMapping(path = "/upload")
public ResponseEntity<String> uploadFile(@RequestParam("file") MultipartFile file) {
    return file.isEmpty() ?
      new ResponseEntity<String>(HttpStatus.NOT_FOUND) : new ResponseEntity<String>(HttpStatus.OK);
}
```

这里，`uploadFile`方法接受多部分 POST 请求。在这个方法中，如果文件存在，我们发送状态码 200；否则，我们将发送状态代码 404。

现在，让我们使用`MockMvc`来测试上面的方法。

首先，让我们自动连接我们的单元测试类中的 [`WebApplicationContext`](/web/20220626193709/https://www.baeldung.com/integration-testing-in-spring#2-the-webapplicationcontext-object) :

```java
@Autowired
private WebApplicationContext webApplicationContext;
```

现在，让我们编写一个方法来测试上面定义的多部分 POST 请求:

```java
@Test
public void whenFileUploaded_thenVerifyStatus() 
  throws Exception {
    MockMultipartFile file 
      = new MockMultipartFile(
        "file", 
        "hello.txt", 
        MediaType.TEXT_PLAIN_VALUE, 
        "Hello, World!".getBytes()
      );

    MockMvc mockMvc 
      = MockMvcBuilders.webAppContextSetup(webApplicationContext).build();
    mockMvc.perform(multipart("/upload").file(file))
      .andExpect(status().isOk());
}
```

这里，**我们使用`[MockMultipartFile](https://web.archive.org/web/20220626193709/https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/mock/web/MockMultipartFile.html)`** 构造函数`,` 定义一个`hello.txt`文件，然后**我们使用之前定义的`webApplicationContext`** 对象构建`mockMvc`对象。

**我们将使用 [`MockMvc#perform`](https://web.archive.org/web/20220626193709/https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/test/web/servlet/MockMvc.html#perform-org.springframework.test.web.servlet.RequestBuilder-) 方法调用 REST 端点**并向其传递 file 对象。最后，我们将检查状态代码 200 来验证我们的测试用例。

## 4.结论

在本文中，我们通过一个例子看到了如何使用`MockMvc`测试 Spring 多部分 POST 请求。

和往常一样，这个项目可以在 GitHub 上获得[。](https://web.archive.org/web/20220626193709/https://github.com/eugenp/tutorials/tree/master/spring-web-modules/spring-mvc-java-2)