# 杰克逊的 Spring JSON-P

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-jackson-jsonp>

## 1。概述

如果你一直在开发 web 上的东西，你会知道**浏览器在处理 AJAX 请求时的同源策略约束**。该约束的简单概述是，不允许来自不同域、模式或端口的任何请求。

在处理 JSON 数据时，**放松这种浏览器限制的一种方法是使用带有填充的 JSON([JSON-P](https://web.archive.org/web/20220812055347/https://en.wikipedia.org/wiki/JSONP))。**

本文讨论了 Spring 对使用 JSON-P 数据的支持——在`AbstractJsonpResponseBodyAdvice`的帮助下。

## 2。JSON-P 在行动

同源策略不强加在`<script>`标签上，允许脚本跨不同的域加载。JSON-P 技术利用了这一点，将 JSON 响应作为 javascript 函数的参数传递。

### 2.1。准备工作

在我们的例子中，我们将使用这个简单的`Company` 类:

```java
public class Company {

    private long id;
    private String name;

    // standard setters and getters
} 
```

这个类将绑定请求参数，并将作为 JSON 表示从服务器返回。

控制器方法也是一个简单的实现——返回`Company`实例:

```java
@RestController
public class CompanyController {

    @RequestMapping(value = "/companyRest",
      produces = MediaType.APPLICATION_JSON_VALUE)
    public Company getCompanyRest() {
        Company company = new Company(1, "Xpto");
        return company;
    }
}
```

在客户端，我们可以使用`jQuery`库创建并发送一个 AJAX 请求:

```java
$.ajax({
    url: 'http://localhost:8080/spring-mvc-java/companyRest',
    data: {
        format: 'json'
    },
    type: 'GET',
    ...
});
```

考虑一个针对以下 URL 的 AJAX 请求:

```java
http://localhost:8080/spring-mvc-java/companyRest 
```

服务器的响应如下:

```java
{"id":1,"name":"Xpto"}
```

因为请求是针对相同的模式、域和端口发送的，所以响应不会被阻塞，浏览器将允许 JSON 数据。

### 2.2。跨来源请求

通过将请求 URL 更改为:

```java
http://127.0.0.1:8080/spring-mvc-java/companyRest 
```

响应将被浏览器阻止，因为请求从`localhost`发送到被认为是不同域的`127.0.0.1`，违反了同源策略。

使用 JSON-P，我们能够向请求添加一个回调参数:

```java
http://127.1.1.1:8080/spring-mvc-java/companyRest?callback=getCompanyData 
```

在客户端，向 AJAX 请求添加以下参数非常简单:

```java
$.ajax({
    ...
    jsonpCallback:'getCompanyData',
    dataType: 'jsonp',
    ...
});
```

`getCompanyData`将是收到响应时调用的函数。

如果服务器格式化响应，如下所示:

```java
getCompanyData({"id":1,"name":"Xpto"}); 
```

浏览器不会阻止它，因为它会将响应视为客户端和服务器之间协商并达成一致的脚本，因为请求和响应都匹配`getCompanyData`。

## 3。`@ControllerAdvice`注解

用`@ControllerAdvice`标注的 beans 能够帮助所有控制器或控制器的特定子集，并用于封装不同控制器之间共享的横切行为。典型的使用模式与[异常处理](/web/20220812055347/https://www.baeldung.com/exception-handling-for-rest-with-spring)、[向模型添加属性](/web/20220812055347/https://www.baeldung.com/spring-mvc-and-the-modelattribute-annotation)或注册绑定器相关。

**从 Spring 4.1** 开始，`@ControllerAdvice`能够注册`ResponseBodyAdvice`接口的实现，该接口允许在响应被控制器方法返回之后，但在被合适的转换器写入之前改变响应。

## 4。使用`AbstractJsonpResponseBodyAdvice` 改变响应

**同样从 Spring 4.1** 开始，我们现在可以访问`AbstractJsonpResponseBodyAdvice`类——它根据 JSON-P 标准格式化响应。

这一节解释了如何在不改变现有控制器的情况下使用基类并改变响应。

为了启用对 JSON-P 的 Spring 支持，让我们从配置开始:

```java
@ControllerAdvice
public class JsonpControllerAdvice 
  extends AbstractJsonpResponseBodyAdvice {

    public JsonpControllerAdvice() {
        super("callback");
    }
} 
```

使用`AbstractJsonpResponseBodyAdvice` 类提供支持。在 super 方法上传递的键是将在请求 JSON-P 数据的 URL 中使用的键。

有了这个控制器建议，我们自动将响应转换成 JSON-P。

## 5。实践中的 JSON-P 与 Spring

有了前面讨论的配置，我们就能够让 REST 应用程序用 JSON-P 进行响应了。在下面的例子中，我们将返回公司的数据，所以我们的 AJAX 请求 URL 应该是这样的:

```java
http://127.0.0.1:8080/spring-mvc-java/companyRest?callback=getCompanyData 
```

由于前面的配置，响应将如下所示:

```java
getCompanyData({"id":1,"name":"Xpto"});
```

如前所述，尽管来自不同的域，这种格式的响应不会被阻止。

`JsonpControllerAdvice`可以很容易地应用于任何返回用`@ResponseBody`和`ResponseEntity`标注的响应的方法。

回调中应该传递了一个同名的函数`getCompanyData`，用于处理所有的响应。

## 6。结论

这篇简短的文章展示了如何使用 Spring 4.1 中的新功能简化利用 JSON-P 格式化响应的繁琐工作。

示例和代码片段的实现可以在这个 [GitHub 项目](https://web.archive.org/web/20220812055347/https://github.com/eugenp/tutorials/tree/master/spring-4)中找到。