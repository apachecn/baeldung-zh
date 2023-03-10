# 用 Spring MVC 下载图像或文件

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-controller-return-image-file>

## 1.概观

向客户机提供静态文件可以通过多种方式完成，使用 Spring 控制器不一定是最佳选择。

然而，有时控制器路线是必要的，这就是我们将在这个快速教程的重点。

## 2.Maven 依赖性

首先，我们需要向我们的`pom.xml`添加一个依赖项:

```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

这就是我们在这里需要做的。有关版本信息，请访问 [Maven Central](https://web.archive.org/web/20221225091828/https://search.maven.org/search?q=a:spring-boot-starter-web) 。

## 3.使用 *@ResponseBody*

第一个简单的解决方案是在控制器方法上使用 *@ResponseBody* 注释，以指示该方法返回的对象应该直接封送到 HTTP 响应体:

```java
@GetMapping("/get-text")
public @ResponseBody String getText() {
    return "Hello world";
} 
```

这个方法将只返回字符串 *Hello world，*，而不是返回名为 *Hello world* 的视图，就像一个更典型的 MVC 应用程序一样。

使用 *@ResponseBody，*我们可以返回几乎任何媒体类型，只要我们有一个相应的 HTTP 消息转换器可以处理它并将其封送到输出流。

## 4.使用*产生用于返回图像的*

返回字节数组允许我们返回几乎任何东西，比如图像或文件:

```java
@GetMapping(value = "/image")
public @ResponseBody byte[] getImage() throws IOException {
    InputStream in = getClass()
      .getResourceAsStream("/com/baeldung/produceimage/image.jpg");
    return IOUtils.toByteArray(in);
} 
```

因为我们没有定义返回的字节数组是一个图像，所以客户端不能把它作为一个图像来处理。事实上，浏览器很可能只显示图像的实际字节。

为了定义返回的字节数组对应于一个图像，我们可以将 *@GetMapping* 注释的 *produces* 属性精确地设置为返回对象的 MIME 类型:

```java
@GetMapping(
  value = "/get-image-with-media-type",
  produces = MediaType.IMAGE_JPEG_VALUE
)
public @ResponseBody byte[] getImageWithMediaType() throws IOException {
    InputStream in = getClass()
      .getResourceAsStream("/com/baeldung/produceimage/image.jpg");
    return IOUtils.toByteArray(in);
} 
```

在这里，*产生的*被设置为*的媒体类型。IMAGE_JPEG_VALUE* 表示返回的对象必须作为 JPEG 图像处理。

现在，浏览器将识别并正确地将响应正文显示为图像。

## 5.使用*产生用于返回原始数据的*

参数*产生*可以根据我们想要返回的对象的类型设置成很多不同的值(完整的列表可以在[这里](https://web.archive.org/web/20221225091828/https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/http/MediaType.html)找到)。

如果我们想返回一个原始文件，我们可以简单地使用*APPLICATION _ OCTET _ STREAM _ VALUE*:

```java
@GetMapping(
  value = "/get-file",
  produces = MediaType.APPLICATION_OCTET_STREAM_VALUE
)
public @ResponseBody byte[] getFile() throws IOException {
    InputStream in = getClass()
      .getResourceAsStream("/com/baeldung/produceimage/data.txt");
    return IOUtils.toByteArray(in);
} 
```

## 6.动态设置`contentType`

现在我们将说明如何动态设置响应的内容类型。在这种情况下，我们不能使用`produces` 参数，因为它需要一个常量。我们需要直接将 [`ResponseEntity`](/web/20221225091828/https://www.baeldung.com/spring-response-entity) 中的`contentType`改为:

```java
@GetMapping("/get-image-dynamic-type")
@ResponseBody
public ResponseEntity<InputStreamResource> getImageDynamicType(@RequestParam("jpg") boolean jpg) {
    MediaType contentType = jpg ? MediaType.IMAGE_JPEG : MediaType.IMAGE_PNG;
    InputStream in = jpg ?
      getClass().getResourceAsStream("/com/baeldung/produceimage/image.jpg") :
      getClass().getResourceAsStream("/com/baeldung/produceimage/image.png");
    return ResponseEntity.ok()
      .contentType(contentType)
      .body(new InputStreamResource(in));
}
```

我们将根据查询参数设置返回图像的内容类型。

## 7.结论

在这篇简短的文章中，我们讨论了一个简单的问题，从 Spring 控制器返回图像或文件。

和往常一样，示例代码可以在 Github 上找到[。](https://web.archive.org/web/20221225091828/https://github.com/eugenp/tutorials/tree/master/spring-boot-modules/spring-boot-mvc-3)