# 从 Spring MVC 控制器返回普通的 HTML

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-mvc-return-html>

## 1.概观

在本教程中，我们想看看如何从 Spring MVC 控制器返回 HTML。

让我们来看看需要做些什么。

## 2.Maven 依赖性

首先，我们必须为我们的 MVC 控制器添加 [`spring-boot-starter-web`](https://web.archive.org/web/20221128035829/https://search.maven.org/artifact/cn.org.faster/spring-boot-starter-web) Maven 依赖项:

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
    <versionId>1.3.7.RELEASE</versionId>
</dependency>
```

## 3.控制器

接下来，让我们创建控制器:

```
@Controller
public class HtmlController {
    @GetMapping(value = "/welcome", produces = MediaType.TEXT_HTML_VALUE)
    @ResponseBody
    public String welcomeAsHTML() {
        return "<html>\n" + "<header><title>Welcome</title></header>\n" +
          "<body>\n" + "Hello world\n" + "</body>\n" + "</html>";
    }
}
```

我们使用`[@Controller](/web/20221128035829/https://www.baeldung.com/spring-controllers) `注释告诉 [`DispatcherServlet`](/web/20221128035829/https://www.baeldung.com/spring-dispatcherservlet) 这个类处理 HTTP 请求。

接下来，我们配置我们的 [`@GetMapping`](/web/20221128035829/https://www.baeldung.com/spring-new-requestmapping-shortcuts) 注释来产生`MediaType.TEXT_HTML_VALUE`输出。

最后，`@ResponseBody`注释告诉控制器，**返回的对象应该自动序列化为配置的媒体类型，**即`TEXT_HTML_VALUE,` 或`text/html`。

如果没有最后一个注释，我们会收到 404 错误，因为默认情况下,`String` 返回值引用了一个视图名。

有了控制器，我们就可以进行测试了:

```
curl -v localhost:8081/welcome
```

输出将类似于:

```
> ... request ...
>
< HTTP/1.1 200
< Content-Type: text/html;charset=UTF-8
< ... other response headers ...
<

<html>
<header><title>Welcome</title></header>
<body>
Hello world
</body>
</html>
```

正如所料，我们看到响应的`Content-Type`是`text/html`。此外，我们看到响应也有正确的 HTML 内容。

## 4.结论

在本文中，我们研究了如何从 Spring MVC 控制器返回 HTML。

和往常一样，代码样本可以在 GitHub 上的[处获得。](https://web.archive.org/web/20221128035829/https://github.com/eugenp/tutorials/tree/master/spring-web-modules/spring-5-mvc)