# 上传 Spring Boot 和百里香叶的图片

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-boot-thymeleaf-image-upload>

## 1。概述

在这个快速教程中，我们将看看**如何使用 Spring Boot 和百里香叶**在 Java web 应用程序中上传图像。

## 2。依赖性

我们只需要两个依赖项— Spring Boot 网和百里香叶:

```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-thymeleaf</artifactId>
</dependency>
```

## 3。Spring Boot 控制器

我们的第一步是创建一个 Spring Boot web 控制器来处理我们的请求:

```java
@Controller public class UploadController {

    public static String UPLOAD_DIRECTORY = System.getProperty("user.dir") + "/uploads";

    @GetMapping("/uploadimage") public String displayUploadForm() {
        return "imageupload/index";
    }

    @PostMapping("/upload") public String uploadImage(Model model, @RequestParam("image") MultipartFile file) throws IOException {
        StringBuilder fileNames = new StringBuilder();
        Path fileNameAndPath = Paths.get(UPLOAD_DIRECTORY, file.getOriginalFilename());
        fileNames.append(file.getOriginalFilename());
        Files.write(fileNameAndPath, file.getBytes());
        model.addAttribute("msg", "Uploaded images: " + fileNames.toString());
        return "imageupload/index";
    }
}
```

我们定义了两种方法来处理 HTTP GET 请求。`displayUploadForm()`方法处理 GET 请求并返回百里香模板的名称以显示给用户，以便让他导入图像。

`uploadImage()`方法处理图像上传。它接受一个关于映像上传的`multipart/form-data` POST 请求，并将映像保存在本地文件系统上。**`MultipartFile`接口是 Spring Boot 提供的一种特殊数据结构，用于在多部分请求**中表示上传的文件。

最后，我们创建了一个上传文件夹来存储所有上传的图片。我们还添加了一条消息，包含上传图像的名称，在用户提交表单后显示。

## 4。百里香叶模板

第二步是创建一个百里香模板，我们在路径`src/main/resources/templates`中称之为`index.html`。该模板显示一个 HTML 表单，允许用户选择和上传图像。此外，我们使用`accept=”image/*”` 属性来允许用户只导入图像，而不导入任何类型的文件。

让我们看看我们的`index.html`文件的结构:

```java
<body>
<section class="my-5">
    <div class="container">
        <div class="row">
            <div class="col-md-8 mx-auto">
                <h2>Upload Image Example</h2>
                <p th:text="${message}" th:if="${message ne null}" class="alert alert-primary"></p>
                <form method="post" th:action="@{/upload}" enctype="multipart/form-data">
                    <div class="form-group">
                        <input type="file" name="image" accept="image/*" class="form-control-file">
                    </div>
                    <button type="submit" class="btn btn-primary">Upload image</button>
                </form>
                <span th:if="${msg != null}" th:text="${msg}"></span>
            </div>
        </div>
    </div>
</section>
</body>
```

## 5T2。自定义文件大小

如果我们试图上传一个大文件，将会抛出一个`MaxUploadSizeExceededException`异常。然而，**我们可以通过在`application.properties`文件中定义的属性`spring.servlet.multipart.max-file-size`和`spring.servlet.multipart.max-request-size` 来调整文件上传限制:**

```java
spring.servlet.multipart.max-file-size = 5MB
spring.servlet.multipart.max-request-size = 5MB
```

## 6T2。结论

在这篇简短的文章中，我们介绍了如何在基于 Spring Boot 和百里香叶的 Java web 应用程序中上传图像。

和往常一样，本文的完整源代码可以在 GitHub 上找到[。](https://web.archive.org/web/20220922075308/https://github.com/eugenp/tutorials/tree/master/spring-web-modules/spring-thymeleaf-5)