# 假装客户端异常处理

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-feign-client-exception-handling>

## 1。概述

在本教程中，我们将演示如何在[佯装](/web/20220906000326/https://www.baeldung.com/intro-to-feign)中处理异常。Feign 是微服务开发者的强大工具，支持 **`[ErrorDecoder](https://web.archive.org/web/20220906000326/https://appdoc.app/artifact/com.netflix.feign/feign-core/8.11.0/feign/codec/ErrorDecoder.html)`和`[FallbackFactory](https://web.archive.org/web/20220906000326/https://github.com/OpenFeign/feign/blob/master/hystrix/src/main/java/feign/hystrix/FallbackFactory.java)` 进行异常处理**。

## 2。Maven 依赖关系

首先，让我们通过包含 [`spring-cloud-starter-openfeign`](https://web.archive.org/web/20220906000326/https://mvnrepository.com/artifact/org.springframework.cloud/spring-cloud-starter-openfeign) 来创建一个 [Spring Boot](/web/20220906000326/https://www.baeldung.com/spring-boot) 项目。 **`spring-cloud-starter-openfeign`内包含`feign-core`依赖**:

```java
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-openfeign</artifactId>
    <version>3.1.3</version>
</dependency>
```

或者我们可以将 [`feign-core`](https://web.archive.org/web/20220906000326/https://search.maven.org/artifact/io.github.openfeign/feign-core) 依赖项添加到我们的`pom.xml`文件中:

```java
<dependency>
    <groupId>io.github.openfeign</groupId>
    <artifactId>feign-core</artifactId>
    <version>11.9.1</version>
</dependency>
```

## 3。 **异常处理同`ErrorDecoder`**

我们可以通过配置`ErrorDecoder,` 来**处理异常，这也允许我们在需要时定制消息。当错误发生时，Feign 客户端会隐藏原始消息。为了检索它，我们可以编写一个自定义的`ErrorDecoder`。让我们覆盖默认的`ErrorDecoder` 实现:**

```java
public class RetreiveMessageErrorDecoder implements ErrorDecoder {
    private final ErrorDecoder errorDecoder = new Default();
    @Override
    public Exception decode(String methodKey, Response response) {
        ExceptionMessage message = null;
        try (InputStream bodyIs = response.body().asInputStream()) {
            ObjectMapper mapper = new ObjectMapper();
            message = mapper.readValue(bodyIs, ExceptionMessage.class);
        } catch (IOException e) {
            return new Exception(e.getMessage());
        }
        switch (response.status()) {
            case 400:
                return new BadRequestException(message.getMessage() != null ? message.getMessage() : "Bad Request");
            case 404:
                return new NotFoundException(message.getMessage() != null ? message.getMessage() : "Not found");
            default:
                return errorDecoder.decode(methodKey, response);
        }
    }
} 
```

在上面的编码器中，我们覆盖了默认行为，以便更好地控制异常。

## 4。 **异常处理同`Fallback`**

我们还可以通过配置`fallback`来处理异常。让我们首先创建一个客户端并配置`fallback`:

```java
@FeignClient(name = "file", url = "http://localhost:8081", 
  configuration = FeignSupportConfig.class, fallback = FileUploadClientWithFallbackImpl.class)
public interface FileUploadClientWithFallBack {
    @PostMapping(value = "/upload-error", consumes = MediaType.MULTIPART_FORM_DATA_VALUE)
    String fileUpload(@RequestPart(value = "file") MultipartFile file);
} 
```

现在，让我们创建`FileUploadClientWithFallbackImpl`来根据我们的需求处理异常:

```java
@Component
public class FileUploadClientWithFallbackImpl implements FileUploadClientWithFallBack {
    @Override
    public String fileUpload(MultipartFile file) {
        try {
            throw new NotFoundException("hi, something wrong");
        } catch (Exception ex) {
            if (ex instanceof BadRequestException) {
                return "Bad Request!!!";
            }
            if (ex instanceof NotFoundException) {
                return "Not Found!!!";
            }
            if (ex instanceof Exception) {
                return "Exception!!!";
            }
            return "Successfully Uploaded file!!!";
        }
    }
} 
```

现在让我们创建一个简单的测试来验证`fallback`选项:

```java
@Test(expected = NotFoundException.class)
public void whenFileUploadClientFallback_thenFileUploadError() throws IOException {
    ClassLoader classloader = Thread.currentThread().getContextClassLoader();
    File file = new File(classloader.getResource(FILE_NAME).getFile());
    Assert.assertTrue(file.exists());
    FileInputStream input = new FileInputStream(file);
    MultipartFile multipartFile = new MockMultipartFile("file", file.getName(), "text/plain",
      IOUtils.toByteArray(input));
    uploadService.uploadFileWithFallback(multipartFile);
}
```

## 5。 **异常处理同`FallbackFactory`**

我们还可以通过配置`FallbackFactory`来处理异常。让我们首先创建一个客户端并配置`FallbackFactory`:

```java
@FeignClient(name = "file", url = "http://localhost:8081", 
  configuration = FeignSupportConfig.class, fallbackFactory = FileUploadClientFallbackFactory.class)
public interface FileUploadClient {
    @PostMapping(value = "/upload-file", consumes = MediaType.MULTIPART_FORM_DATA_VALUE)
    String fileUpload(@RequestPart(value = "file") MultipartFile file);
}
```

现在，让我们创建`FileUploadClientFallbackFactory`来根据我们的需求处理异常:

```java
@Component
public class FileUploadClientFallbackFactory implements FallbackFactory<FileUploadClient> {
    @Override
    public FileUploadClient create(Throwable cause) {
        return new FileUploadClient() {
            @Override
            public String fileUpload(MultipartFile file) {
                if (cause instanceof BadRequestException) {
                    return "Bad Request!!!";
                }
                if (cause instanceof NotFoundException) {
                    return "Not Found!!!";
                }
                if (cause instanceof Exception) {
                    return "Exception!!!";
                }
                return "Successfully Uploaded file!!!";
            }
        };
    }
}
```

现在让我们创建一个简单的测试来验证`FallbackFactory`选项:

```java
@Test(expected = NotFoundException.class)
public void whenFileUploadClientFallbackFactory_thenFileUploadError() throws IOException {
    ClassLoader classloader = Thread.currentThread().getContextClassLoader();
    File file = new File(classloader.getResource(FILE_NAME).getFile());
    Assert.assertTrue(file.exists());
    FileInputStream input = new FileInputStream(file);
    MultipartFile multipartFile = new MockMultipartFile("file", file.getName(), "text/plain",
      IOUtils.toByteArray(input));
    uploadService.uploadFileWithFallbackFactory(multipartFile);
} 
```

## 6。结论

在本文中，我们展示了如何在 feign 中处理异常。

和往常一样，本教程中使用的所有代码示例都可以从 GitHub 上的[处获得。](https://web.archive.org/web/20220906000326/https://github.com/eugenp/tutorials/tree/master/spring-cloud-modules/spring-cloud-openfeign-2)