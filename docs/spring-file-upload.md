# 用 Spring MVC 上传文件

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-file-upload>

## 1。概述

在之前的教程中，我们[介绍了表单处理的基础](/web/20220617075809/https://www.baeldung.com/spring-mvc-form-tutorial)，并探索了 Spring MVC 中的[表单标签库](/web/20220617075809/https://www.baeldung.com/spring-mvc-form-tags)。

在本教程中，我们关注 Spring 在 web 应用程序中为**多部分(文件上传)支持**提供了什么。

Spring 允许我们用可插入的`MultipartResolver`对象来启用这种多部分支持。该框架提供了一个用于 **Commons FileUpload** 的`MultipartResolver`实现，以及另一个用于 **Servlet 3.0** 多部分请求解析的实现。

配置完`MultipartResolver`之后，我们将看到如何上传单个文件和多个文件。

我们还会谈到 Spring Boot。

## 2。 **公地文件上传**

要使用`CommonsMultipartResolver`来处理文件上传，我们需要添加以下依赖项:

```java
<dependency>
    <groupId>commons-fileupload</groupId>
    <artifactId>commons-fileupload</artifactId>
    <version>1.4</version>
</dependency>
```

现在我们可以在 Spring 配置中定义`CommonsMultipartResolver` bean 了。

这个`MultipartResolver`带有一系列`set`方法来定义属性，比如上传的最大大小:

```java
@Bean(name = "multipartResolver")
public CommonsMultipartResolver multipartResolver() {
    CommonsMultipartResolver multipartResolver = new CommonsMultipartResolver();
    multipartResolver.setMaxUploadSize(100000);
    return multipartResolver;
}
```

这里我们需要在 Bean 定义本身中控制`CommonsMultipartResolver`的不同属性。

## 3。用 **Servlet 3.0**

为了使用`Servlet 3.0`多部分解析，我们需要配置应用程序的几个部分。

**首先，我们需要在我们的`DispatcherServlet` `registration`** 中设置一个`MultipartConfigElement`:

```java
public class MainWebAppInitializer implements WebApplicationInitializer {

    private String TMP_FOLDER = "/tmp"; 
    private int MAX_UPLOAD_SIZE = 5 * 1024 * 1024; 

    @Override
    public void onStartup(ServletContext sc) throws ServletException {

        ServletRegistration.Dynamic appServlet = sc.addServlet("mvc", new DispatcherServlet(
          new GenericWebApplicationContext()));

        appServlet.setLoadOnStartup(1);

        MultipartConfigElement multipartConfigElement = new MultipartConfigElement(TMP_FOLDER, 
          MAX_UPLOAD_SIZE, MAX_UPLOAD_SIZE * 2, MAX_UPLOAD_SIZE / 2);

        appServlet.setMultipartConfig(multipartConfigElement);
    }
}
```

在`MultipartConfigElement`对象中，我们已经配置了存储位置、最大单个文件大小、最大请求大小(在单个请求中有多个文件的情况下)以及文件上传进度刷新到存储位置的大小。

这些设置必须在 servlet 注册级别应用，因为`Servlet 3.0`不允许它们像`CommonsMultipartResolver`一样在`MultipartResolver` 中注册。

完成后，**我们可以将`StandardServletMultipartResolver`添加到我们的弹簧配置**:

```java
@Bean
public StandardServletMultipartResolver multipartResolver() {
    return new StandardServletMultipartResolver();
}
```

## 4。上传文件

为了上传我们的文件，我们可以构建一个简单的表单，在这个表单中我们使用一个带有`type='file'`的 HTML `input`标签。

不管我们选择了什么样的上传处理配置，我们都需要将表单的编码属性设置为`multipart/form-data`。

这让浏览器知道如何对表单进行编码:

```java
<form:form method="POST" action="/spring-mvc-xml/uploadFile" enctype="multipart/form-data">
    <table>
        <tr>
            <td><form:label path="file">Select a file to upload</form:label></td>
            <td><input type="file" name="file" /></td>
        </tr>
        <tr>
            <td><input type="submit" value="Submit" /></td>
        </tr>
    </table>
</form>
```

**为了存储上传的文件，我们可以使用一个`MultipartFile`变量。**

我们可以从控制器方法内部的请求参数中检索这个变量:

```java
@RequestMapping(value = "/uploadFile", method = RequestMethod.POST)
public String submit(@RequestParam("file") MultipartFile file, ModelMap modelMap) {
    modelMap.addAttribute("file", file);
    return "fileUploadView";
} 
```

**`MultipartFile`类提供对上传文件**的详细信息的访问，包括文件名、文件类型等等。

我们可以使用一个简单的 HTML 页面来显示这些信息:

```java
<h2>Submitted File</h2>
<table>
    <tr>
        <td>OriginalFileName:</td>
        <td>${file.originalFilename}</td>
    </tr>
    <tr>
        <td>Type:</td>
        <td>${file.contentType}</td>
    </tr>
</table>
```

## 5。上传 **多个文件**

要在单个请求中上传多个文件，我们只需在表单中放置多个输入文件字段:

```java
<form:form method="POST" action="/spring-mvc-java/uploadMultiFile" enctype="multipart/form-data">
    <table>
        <tr>
            <td>Select a file to upload</td>
            <td><input type="file" name="files" /></td>
        </tr>
        <tr>
            <td>Select a file to upload</td>
            <td><input type="file" name="files" /></td>
        </tr>
        <tr>
            <td>Select a file to upload</td>
            <td><input type="file" name="files" /></td>
        </tr>
        <tr>
            <td><input type="submit" value="Submit" /></td>
        </tr>
    </table>
</form:form> 
```

我们需要注意每个输入字段都有相同的名称，这样它就可以作为一个数组`MultipartFile`来访问:

```java
@RequestMapping(value = "/uploadMultiFile", method = RequestMethod.POST)
public String submit(@RequestParam("files") MultipartFile[] files, ModelMap modelMap) {
    modelMap.addAttribute("files", files);
    return "fileUploadView";
} 
```

现在，我们可以简单地遍历该数组来显示文件信息:

```java
<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>
<html>
    <head>
        <title>Spring MVC File Upload</title>
    </head>
    <body>
        <h2>Submitted Files</h2>
        <table>
            <c:forEach items="${files}" var="file">    
                <tr>
                    <td>OriginalFileName:</td>
                    <td>${file.originalFilename}</td>
                </tr>
                <tr>
                    <td>Type:</td>
                    <td>${file.contentType}</td>
                </tr>
            </c:forEach>
        </table>
    </body>
</html>
```

## 6。上传带有附加表单数据的文件

我们还可以将附加信息与上传的文件一起发送到服务器。

我们只需在表单中包含必填字段:

```java
<form:form method="POST" 
  action="/spring-mvc-java/uploadFileWithAddtionalData"
  enctype="multipart/form-data">
    <table>
        <tr>
            <td>Name</td>
            <td><input type="text" name="name" /></td>
        </tr>
        <tr>
            <td>Email</td>
            <td><input type="text" name="email" /></td>
        </tr>
        <tr>
            <td>Select a file to upload</td>
            <td><input type="file" name="file" /></td>
        </tr>
        <tr>
            <td><input type="submit" value="Submit" /></td>
        </tr>
    </table>
</form:form>
```

在控制器中，我们可以使用`@RequestParam`注释获得所有表单数据:

```java
@PostMapping("/uploadFileWithAddtionalData")
public String submit(
  @RequestParam MultipartFile file, @RequestParam String name,
  @RequestParam String email, ModelMap modelMap) {

    modelMap.addAttribute("name", name);
    modelMap.addAttribute("email", email);
    modelMap.addAttribute("file", file);
    return "fileUploadView";
}
```

与前面的部分类似，我们可以使用带有`JSTL`标签的 HTML 页面来显示信息。

我们还可以将所有表单字段封装在一个模型类中，并在控制器中使用`@ModelAttribute`注释。当文件中有许多附加字段时，这将很有帮助。

让我们看看代码:

```java
public class FormDataWithFile {

    private String name;
    private String email;
    private MultipartFile file;

    // standard getters and setters
}
```

```java
@PostMapping("/uploadFileModelAttribute")
public String submit(@ModelAttribute FormDataWithFile formDataWithFile, ModelMap modelMap) {

    modelMap.addAttribute("formDataWithFile", formDataWithFile);
    return "fileUploadView";
}
```

## 7。Spring Boot 文件上传

如果我们使用 Spring Boot，那么我们到目前为止看到的一切仍然适用。 

然而，Spring Boot 使配置和启动一切变得更加容易，几乎没有任何麻烦。

特别是，**不需要配置任何 servlet** ，因为 Boot 会为我们注册和配置它，只要我们在依赖关系中包含 web 模块:

```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
    <version>2.6.1</version>
</dependency>
```

我们可以在 Maven Central 上找到最新版本的`[spring-boot-starter-web](https://web.archive.org/web/20220617075809/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.springframework.boot%22%20AND%20a%3A%22spring-boot-starter-web%22)`。

**如果我们想控制最大文件上传大小，我们可以编辑我们的`application.properties`** :

```java
spring.servlet.multipart.max-file-size=128KB
spring.servlet.multipart.max-request-size=128KB
```

我们还可以控制是否启用文件上传以及文件上传的位置:

```java
spring.servlet.multipart.enabled=true
spring.servlet.multipart.location=${java.io.tmpdir}
```

**注意，我们使用了`${java.io.tmpdir}`来定义上传位置，这样我们就可以为不同的操作系统使用这个临时位置。**

## 8。结论

在本文中，我们研究了在 Spring 中配置多部分支持的不同方法。使用这些，我们可以在 web 应用程序中支持文件上传。 

本教程的实现可以在 [GitHub 项目](https://web.archive.org/web/20220617075809/https://github.com/eugenp/tutorials/tree/master/spring-web-modules/spring-mvc-java)中找到。当项目在本地运行时，可以在 http://localhost:8080/spring-MVC-Java/file upload 访问表单示例