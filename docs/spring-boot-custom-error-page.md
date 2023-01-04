# Spring Boot:自定义白标错误页面

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-boot-custom-error-page>

## 1。概述

在本文中，我们将看看如何**禁用和定制 Spring Boot 应用程序的默认错误页面**，因为正确的错误处理体现了专业精神和高质量的工作。

## 2。禁用白标错误页面

首先，让我们看看如何通过将`server.error.whitelabel.enabled`属性设置为`false:`来完全禁用白色标签错误页面

```java
server.error.whitelabel.enabled=false
```

将此条目添加到 application.properties 文件将禁用错误页面，并显示源自底层应用程序容器(例如 Tomcat)的简明页面。

**我们可以通过排除`ErrorMvcAutoConfiguration` bean 获得相同的结果。**我们可以通过将这个条目添加到属性文件中:

```java
spring.autoconfigure.exclude=org.springframework.boot.autoconfigure.web.ErrorMvcAutoConfiguration

#for Spring Boot 2.0
#spring.autoconfigure.exclude=org.springframework.boot.autoconfigure.web.servlet.error.ErrorMvcAutoConfiguration
```

或者将这个注释添加到主类中:

```java
@EnableAutoConfiguration(exclude = {ErrorMvcAutoConfiguration.class})
```

上面提到的所有方法都会禁用白色标签错误页面。这就给我们留下了一个问题，谁来真正处理这个错误呢？

如上所述，它通常是底层应用程序容器。好的一面是，我们可以通过显示自定义错误页面而不是所有的默认页面来进一步自定义，这是下一节的重点。

## 3。显示自定义错误页面

我们首先需要创建一个定制的 HTML 错误页面。

**我们将文件保存为`error.html`** ，因为我们正在使用`Thymeleaf` 模板引擎:

```java
<!DOCTYPE html>
<html>
<body>
<h1>Something went wrong! </h1>
<h2>Our Engineers are on it</h2>
<a href="/">Go Home</a>
</body>
</html>
```

**如果我们把这个文件保存在`resources/templates`目录下，它会被默认的 Spring Boot 的`BasicErrorController`自动提取**。

这就是我们显示自定义错误页面所需的全部内容。通过一些样式，我们现在将为用户提供一个更好看的错误页面:

[![Spring Boot Custom Error Page](img/a2654c55a79da3c96d271d82a73a1e50.png)](/web/20221218220601/https://www.baeldung.com/wp-content/uploads/2018/04/error1.png)

我们可以更具体地用我们希望使用的 HTTP 状态代码来命名文件，例如，将文件保存为`resources/templates/error`中的`404.html`意味着它将明确地用于 404 错误。

### 3.1.一个习俗`ErrorController`

到目前为止的限制是，当错误发生时，我们不能运行自定义逻辑。为了实现这一点，我们必须创建一个错误控制器 bean 来替换默认的错误控制器 bean。

为此，**我们必须创建一个实现`ErrorController`接口的类。**此外，我们需要设置`server.error.path`属性来返回一个自定义路径，以便在发生错误时调用

```java
@Controller
public class MyErrorController implements ErrorController  {

    @RequestMapping("/error")
    public String handleError() {
        //do something like logging
        return "error";
    }
}
```

在上面的代码片段中，我们还用`@Controller`注释了这个类，并为被指定为属性`server.error.path:`的路径创建了一个映射

```java
server.error.path=/error
```

这样，控制器可以处理对`/error`路径的调用。

在`handleError()`中，我们返回之前创建的自定义错误页面。如果我们现在触发 404 错误，将显示我们的自定义页面。

**让我们进一步增强`handleError()`来显示不同错误类型的特定错误页面。**

例如，我们可以专门为 404 和 500 错误类型设计精美的页面。然后，我们可以使用错误的 HTTP 状态代码来确定要显示的合适的错误页面:

```java
@RequestMapping("/error")
public String handleError(HttpServletRequest request) {
    Object status = request.getAttribute(RequestDispatcher.ERROR_STATUS_CODE);

    if (status != null) {
        Integer statusCode = Integer.valueOf(status.toString());

        if(statusCode == HttpStatus.NOT_FOUND.value()) {
            return "error-404";
        }
        else if(statusCode == HttpStatus.INTERNAL_SERVER_ERROR.value()) {
            return "error-500";
        }
    }
    return "error";
}
```

然后，例如，对于 404 错误，我们将看到`error-404.html`页面:

[![error404](img/2bac17664f228c6e30a828e5090329fd.png)](/web/20221218220601/https://www.baeldung.com/wp-content/uploads/2018/04/error404.png)

## 4。结论

有了这些信息，我们现在可以更优雅地处理错误，并向用户展示一个美观的页面。

和往常一样，完整的源代码可以在 Github 上找到[。](https://web.archive.org/web/20221218220601/https://github.com/eugenp/tutorials/tree/master/spring-boot-modules/spring-boot-basic-customization)