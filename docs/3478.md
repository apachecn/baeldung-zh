# 用 Spring MVC 返回图像/媒体数据

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-mvc-image-media-data>

## 1。概述

在本教程中，我们将演示如何使用 Spring MVC 框架返回图像和其他媒体。

我们将讨论几种方法，从直接操作`HttpServletResponse`开始，然后讨论受益于[消息转换](/web/20220625180224/https://www.baeldung.com/spring-httpmessageconverter-rest)、[内容协商](/web/20220625180224/https://www.baeldung.com/spring-mvc-content-negotiation-json-xml)和 Spring 的`Resource` 抽象的方法。我们将仔细研究它们，并讨论它们的优缺点。

## 2。使用`HttpServletResponse`

图像下载最基本的方法是直接操作一个`response`对象并模仿一个纯粹的`Servlet`实现，并使用下面的代码片段进行演示:

```
@RequestMapping(value = "/image-manual-response", method = RequestMethod.GET)
public void getImageAsByteArray(HttpServletResponse response) throws IOException {
    InputStream in = servletContext.getResourceAsStream("/WEB-Iimg/image-example.jpg");
    response.setContentType(MediaType.IMAGE_JPEG_VALUE);
    IOUtils.copy(in, response.getOutputStream());
}
```

发出以下请求将在浏览器中呈现图像:

```
http://localhost:8080/spring-mvc-xml/image-manual-response.jpg
```

由于`org.apache.commons.io`包中的`IOUtils`，实现相当简单明了。然而，这种方法的缺点是它对潜在的变化不够健壮。mime 类型是硬编码的，转换逻辑的更改或图像位置的具体化需要更改代码。

下一节讨论一种更灵活的方法。

## 3。使用`HttpMessageConverter`

前一节讨论了一种基本方法，它没有利用 Spring MVC 框架的[消息转换](/web/20220625180224/https://www.baeldung.com/spring-httpmessageconverter-rest)和[内容协商](/web/20220625180224/https://www.baeldung.com/spring-mvc-content-negotiation-json-xml)特性。为了引导这些功能，我们需要:

*   用`@ResponseBody`注释来注释控制器方法
*   根据控制器方法的返回类型注册一个适当的消息转换器(例如，需要将字节数组正确转换为图像文件)

### 3.1。配置

为了展示转换器的配置，我们将使用内置的`ByteArrayHttpMessageConverter`，每当方法返回`byte[]`类型时，它就转换一条消息。

默认情况下会注册`ByteArrayHttpMessageConverter`,但其他内置或定制转换器的配置类似。

应用消息转换器 bean 需要在 Spring MVC 上下文中注册一个适当的`MessageConverter` bean，并设置它应该处理的媒体类型。您可以使用`<mvc:message-converters>` 标签通过 XML 定义它。

这个标签应该在`<mvc:annotation-driven>`标签中定义，如下例所示:

```
<mvc:annotation-driven>
    <mvc:message-converters>
        <bean class="org.springframework.http.converter.ByteArrayHttpMessageConverter">
            <property name="supportedMediaTypes">
                <list>
                    <value>image/jpeg</value>
                    <value>image/png</value>
                </list>
            </property>
        </bean>
    </mvc:message-converters>
</mvc:annotation-driven>
```

前述配置部分将为`image/jpeg`和`image/png`响应内容类型注册`ByteArrayHttpMessageConverter` 。如果 mvc 配置中没有`<mvc:message-converters>`标签，那么将注册默认的一组转换器。

此外，您可以使用 Java 配置注册消息转换器**:**

```
@Override
public void configureMessageConverters(List<HttpMessageConverter<?>> converters) {
    converters.add(byteArrayHttpMessageConverter());
}

@Bean
public ByteArrayHttpMessageConverter byteArrayHttpMessageConverter() {
    ByteArrayHttpMessageConverter arrayHttpMessageConverter = new ByteArrayHttpMessageConverter();
    arrayHttpMessageConverter.setSupportedMediaTypes(getSupportedMediaTypes());
    return arrayHttpMessageConverter;
}

private List<MediaType> getSupportedMediaTypes() {
    List<MediaType> list = new ArrayList<MediaType>();
    list.add(MediaType.IMAGE_JPEG);
    list.add(MediaType.IMAGE_PNG);
    list.add(MediaType.APPLICATION_OCTET_STREAM);
    return list;
}
```

### 3.2。实施

现在我们可以实现我们的方法来处理对媒体的请求。如上所述，您需要用`@ResponseBody`注释标记您的控制器方法，并使用`byte[]`作为返回类型:

```
@RequestMapping(value = "/image-byte-array", method = RequestMethod.GET)
public @ResponseBody byte[] getImageAsByteArray() throws IOException {
    InputStream in = servletContext.getResourceAsStream("/WEB-Iimg/image-example.jpg");
    return IOUtils.toByteArray(in);
}
```

要测试该方法，请在浏览器中发出以下请求:

```
http://localhost:8080/spring-mvc-xml/image-byte-array.jpg
```

有利的一面是，该方法对`HttpServletResponse,` 一无所知。转换过程是高度可配置的，从使用可用的转换器到指定一个定制的转换器。响应的内容类型不必硬编码，而是基于请求路径后缀`.jpg`进行[协商](/web/20220625180224/https://www.baeldung.com/spring-mvc-content-negotiation-json-xml)。

这种方法的缺点是，您需要显式地实现从数据源(本地文件、外部存储等)检索图像的逻辑。)并且您无法控制响应的标头或状态代码。

## 4。使用`ResponseEntity`类

您可以返回一个包装在`Response Entity`中的图像`byte[]`。Spring MVC `ResponseEntity`不仅支持对 HTTP 响应主体的控制，还支持对头部和响应状态代码的控制。按照这种方法，您需要将方法的返回类型定义为`ResponseEntity<byte[]>`，并在方法体中创建返回的`ResponseEntity`对象。

```
@RequestMapping(value = "/image-response-entity", method = RequestMethod.GET)
public ResponseEntity<byte[]> getImageAsResponseEntity() {
    HttpHeaders headers = new HttpHeaders();
    InputStream in = servletContext.getResourceAsStream("/WEB-Iimg/image-example.jpg");
    byte[] media = IOUtils.toByteArray(in);
    headers.setCacheControl(CacheControl.noCache().getHeaderValue());

    ResponseEntity<byte[]> responseEntity = new ResponseEntity<>(media, headers, HttpStatus.OK);
    return responseEntity;
}
```

使用`ResponseEntity`允许您为给定的请求配置响应代码。

显式设置响应代码在遇到异常事件时特别有用，例如，如果图像未找到(`FileNotFoundException`)或损坏(`IOException)`)。在这些情况下，所需要的就是在一个适当的 catch 块中设置响应代码，例如`new ResponseEntity<>(null, headers, HttpStatus.NOT_FOUND),` 。

此外，如果您需要在您的响应中设置一些特定的头，这种方法比通过方法作为参数接受的`HttpServletResponse`对象来设置头更直接。它使方法签名清晰而集中。

## 5。使用`Resource` 类返回图像

最后，您可以以`Resource`对象的形式返回一个图像。

`Resource`接口是抽象访问低级资源的接口。它在春季推出，作为标准`java.net.URL`级的更强大的替代品。它允许轻松访问不同类型的资源(本地文件、远程文件、类路径资源)，而无需编写显式检索它们的代码。

要使用这种方法，应该将方法的返回类型设置为`Resource`，并且需要用`@ResponseBody`注释来注释该方法。

### 5.1。实施

```
@ResponseBody
@RequestMapping(value = "/image-resource", method = RequestMethod.GET)
public Resource getImageAsResource() {
   return new ServletContextResource(servletContext, "/WEB-Iimg/image-example.jpg");
}
```

或者，如果我们希望对响应头有更多的控制:

```
@RequestMapping(value = "/image-resource", method = RequestMethod.GET)
@ResponseBody
public ResponseEntity<Resource> getImageAsResource() {
    HttpHeaders headers = new HttpHeaders();
    Resource resource = 
      new ServletContextResource(servletContext, "/WEB-Iimg/image-example.jpg");
    return new ResponseEntity<>(resource, headers, HttpStatus.OK);
}
```

使用这种方法，您将图像视为可以使用`ResourceLoader`接口实现加载的资源。在这种情况下，你从你的图像的确切位置提取，然后`ResourceLoader`决定从哪里加载。

它提供了一种使用配置控制图像位置的通用方法，并消除了编写文件加载代码的需要。

## 6。结论

在上述方法中，我们从基本方法开始，然后使用受益于框架的消息转换特性的方法。我们还讨论了如何在不直接处理响应对象的情况下获得响应代码和响应头。

最后，我们从图像位置的角度增加了灵活性，因为从哪里检索图像是在配置中定义的，更容易随时更改。

[用 Spring 下载图像或文件](/web/20220625180224/https://www.baeldung.com/spring-controller-return-image-file)解释了如何使用 Spring Boot 实现同样的事情。

教程后面的示例代码可以从 [GitHub](https://web.archive.org/web/20220625180224/https://github.com/eugenp/tutorials/tree/master/spring-web-modules/spring-mvc-xml) 获得。