# 使用 Open Feign 上传文件

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-feign-file-upload>

## 1.概观

在本教程中，我们将演示如何使用 Open Feign 上传文件。 [Feign](/web/20220626075944/https://www.baeldung.com/intro-to-feign) 是微服务开发者通过 REST API 以声明方式与其他微服务进行通信的强大工具。

## 2.先决条件

让我们**假设一个 RESTful web 服务被暴露**用于文件上传，下面给出细节:

```java
POST http://localhost:8081/upload-file
```

因此，为了解释通过`Feign`客户端上传文件，我们将调用公开的 web 服务 API，如下所示:

```java
@PostMapping(value = "/upload-file")
public String handleFileUpload(@RequestPart(value = "file") MultipartFile file) {
    // File upload logic
}
```

## 3.属国

为了**支持文件上传的`application/x-www-form-urlencoded`和`multipart/form-data`编码类型**，我们需要`[feign-core](https://web.archive.org/web/20220626075944/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22io.github.openfeign%22%20AND%20a%3A%22feign-core%22)`、 `[feign-form](https://web.archive.org/web/20220626075944/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22io.github.openfeign.form%22%20AND%20a%3A%22feign-form%22),`和 `[feign-form-spring](https://web.archive.org/web/20220626075944/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22io.github.openfeign.form%22%20AND%20a%3A%22feign-form-spring%22)`模块。

因此，我们将向 Maven 添加以下依赖项:

```java
<dependency>
    <groupId>io.github.openfeign</groupId>
    <artifactId>feign-core</artifactId>
    <version>10.12</version>
</dependency>
<dependency>
    <groupId>io.github.openfeign.form</groupId>
    <artifactId>feign-form</artifactId>
    <version>3.8.0</version>
</dependency>
<dependency>
    <groupId>io.github.openfeign.form</groupId>
    <artifactId>feign-form-spring</artifactId>
    <version>3.8.0</version>
</dependency>
```

我们也可以使用内部有`feign-core`的`[spring-cloud-starter-openfeign](https://web.archive.org/web/20220626075944/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22io.github.openfeign%22%20AND%20a%3A%22feign-core%22)`:

```java
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-openfeign</artifactId>
    <version>3.1.0</version>
</dependency>
```

## 4.配置

让我们将`@EnableFeignClients` 添加到我们的主类中。更多详情可以访问`[spring cloud open feign](/web/20220626075944/https://www.baeldung.com/spring-cloud-openfeign)` 教程:

```java
@SpringBootApplication
@EnableFeignClients
public class ExampleApplication {
    public static void main(String[] args) {
        SpringApplication.run(ExampleApplication.class, args);
    }
}
```

`@EnableFeignClients`注释允许组件扫描被声明为假客户端的接口。

## 5.通过虚拟客户端上传文件

### 5.1.通过带注释的客户端

让我们为带注释的`@FeignClient`类创建所需的编码器:

```java
public class FeignSupportConfig {
    @Bean
    public Encoder multipartFormEncoder() {
        return new SpringFormEncoder(new SpringEncoder(new ObjectFactory<HttpMessageConverters>() {
            @Override
            public HttpMessageConverters getObject() throws BeansException {
                return new HttpMessageConverters(new RestTemplate().getMessageConverters());
            }
        }));
    }
}
```

注意`FeignSupportConfig`不需要用`@Configuration.`标注

现在，让我们创建一个接口并用`@FeignClient`对其进行注释。我们还将添加 **`name`和`configuration`属性**及其相应的值:

```java
@FeignClient(name = "file", url = "http://localhost:8081", configuration = FeignSupportConfig.class)
public interface UploadClient {
    @PostMapping(value = "/upload-file", consumes = MediaType.MULTIPART_FORM_DATA_VALUE)
    String fileUpload(@RequestPart(value = "file") MultipartFile file);
} 
```

`UploadClient`指向中提到的 API，前提是。

在使用`Hystrix`时，我们将使用`fallback` 属性来添加替代项。这是在上传 API 失败时完成的`.`

现在我们的@FeignClient 看起来像这样:

```java
@FeignClient(name = "file", url = "http://localhost:8081", fallback = UploadFallback.class, configuration = FeignSupportConfig.class)
```

最后，我们可以直接从服务层调用`UploadClient`:

```java
public String uploadFile(MultipartFile file) {
    return client.fileUpload(file);
}
```

### 5.2.通过`Feign.builder`

在某些情况下，我们的 Feign 客户端需要定制，这在上面描述的注释方式中是不可能的。在这种情况下，我们使用`Feign.builder()` API 创建客户端。

让我们构建一个代理接口，它包含一个针对 REST API 的文件上传方法，用于文件上传:

```java
public interface UploadResource {
    @RequestLine("POST /upload-file")
    @Headers("Content-Type: multipart/form-data")
    Response uploadFile(@Param("file") MultipartFile file);
}
```

注解 **`@RequestLine`** 定义了 API 的 HTTP 方法和相对资源路径， **`@Headers`** 指定了 Content-Type 等头。

现在，让我们在代理接口中调用指定的方法。我们将在我们的服务类中实现这一点:

```java
public boolean uploadFileWithManualClient(MultipartFile file) {
    UploadResource fileUploadResource = Feign.builder().encoder(new SpringFormEncoder())
      .target(UploadResource.class, HTTP_FILE_UPLOAD_URL);
    Response response = fileUploadResource.uploadFile(file);
    return response.status() == 200;
}
```

这里，我们使用了`Feign.builder()`实用程序来构建一个`UploadResource`代理接口的实例。我们还使用了`SpringFormEncoder`和基于 RESTful Web 服务的 URL。

## 6.确认

让我们创建一个测试来验证带注释的客户端的文件上传:

```java
@SpringBootTest
public class OpenFeignFileUploadLiveTest {

    @Autowired
    private UploadService uploadService;

    private static String FILE_NAME = "fileupload.txt";

    @Test
    public void whenAnnotatedFeignClient_thenFileUploadSuccess() {
        ClassLoader classloader = Thread.currentThread().getContextClassLoader();
        File file = new File(classloader.getResource(FILE_NAME).getFile());
        Assert.assertTrue(file.exists());
        FileInputStream input = new FileInputStream(file);
        MultipartFile multipartFile = new MockMultipartFile("file", file.getName(), "text/plain",
          IOUtils.toByteArray(input));
        String uploadFile = uploadService.uploadFile(multipartFile);

        Assert.assertNotNull(uploadFile);
    }
}
```

现在，让我们创建另一个测试来验证使用`Feign.Builder()`上传的文件:

```java
@Test
public void whenFeignBuilder_thenFileUploadSuccess() throws IOException {
    // same as above
    Assert.assertTrue(uploadService.uploadFileWithManualClient(multipartFile));
}
```

## 7.结论

在本文中，我们展示了如何使用 OpenFeign 实现多部分文件上传，以及将它包含在一个简单应用程序中的各种方法。

我们还看到了如何配置一个虚拟客户端或者使用`Feign.Builder()`来执行相同的`.`

像往常一样，本教程中使用的所有代码示例都可以在 GitHub 上获得。