# 春季 MVC 数据和百里香叶

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-mvc-thymeleaf-data>

## 1.介绍

在本教程中，我们将讨论使用百里香叶访问 Spring MVC 数据的不同方法。

我们将从使用百里香创建一个电子邮件模板开始，我们将使用来自 Spring 应用程序的数据来增强它。

## 2.项目设置

首先，我们需要添加我们的[百里香叶依赖关系](https://web.archive.org/web/20220728105348/https://search.maven.org/artifact/org.springframework.boot/spring-boot-starter-thymeleaf):

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-thymeleaf</artifactId>
    <version>2.6.1</version>
</dependency>
```

第二，让我们把 Spring Boot 的[`web starter`](https://web.archive.org/web/20220728105348/https://search.maven.org/artifact/org.springframework.boot/spring-boot-starter-web)包括进来:

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
    <version>2.6.1</version>
</dependency> 
```

这种依赖性为我们提供了 REST 支持，我们稍后将使用它来创建一些端点。

我们将创建几个百里香模板来覆盖我们的示例案例，并将它们存储在`resources/mvcdata`中。教程的每个部分将实现不同的模板:

```
<!DOCTYPE html>
<html  xmlns:th="http://www.thymeleaf.org">
    <!-- data -->
</html>
```

最后，我们需要实现一个控制器类来存储我们的业务逻辑:

```
@Controller
public class EmailController {
    private ServletContext servletContext;

    public EmailController(ServletContext servletContext) {
        this.servletContext = servletContext;
    }
} 
```

我们的控制器类并不总是依赖于 servlet 上下文，但是我们在这里添加了它，以便我们稍后可以演示一个特定的百里香特性。

## 3.模型属性

模型属性在**控制器类中使用，这些控制器类为视图**中的渲染准备数据。

我们可以向模型中添加属性的一种方式是要求一个`Model`实例作为控制器方法中的参数。

让我们将我们的`emailData`作为一个属性来传递:

```
@GetMapping(value = "/email/modelattributes")
public String emailModel(Model model) {
    model.addAttribute("emailData", emailData);
    return "mvcdata/email-model-attributes";
} 
```

当请求`/email/modelattributes` 时，Spring 将为我们注入一个`Model`的实例。

然后，我们可以在百里香表达式中引用我们的`emailData`模型属性:

```
<p th:text="${emailData.emailSubject}">Subject</p>
```

另一种方法是通过使用`@ModelAttribute`告诉我们的 Spring 容器在我们的视图中需要什么属性:

```
@ModelAttribute("emailModelAttribute")
EmailData emailModelAttribute() {
    return emailData;
}
```

然后我们可以将视图中的数据表示为:

```
<p th:each="emailAddress : ${emailModelAttribute.getEmailAddresses()}">
    <span th:text="${emailAddress}"></span>
</p>
```

有关模型数据的更多示例，请查看 Spring MVC 教程中的[模型、模型图和模型视图。](/web/20220728105348/https://www.baeldung.com/spring-mvc-model-model-map-model-view)

## 4.请求参数

另一种访问数据的方式是通过请求参数:

```
@GetMapping(value = "/email/requestparameters")
public String emailRequestParameters(
    @RequestParam(value = "emailsubject") String emailSubject) {
    return "mvcdata/email-request-parameters";
}
```

同时，在我们的模板中，我们需要使用关键字`param`指定**哪个参数包含数据**:

```
<p th:text="${param.emailsubject}"></p>
```

我们也可以有多个同名的请求参数:

```
@GetMapping(value = "/email/requestparameters")
public String emailRequestParameters(
    @RequestParam(value = "emailsubject") String emailSubject,
    @RequestParam(value = "emailaddress") String emailAddress1,
    @RequestParam(value = "emailaddress") String emailAddress2) {
    return "mvcdata/email-request-parameters";
} 
```

然后，我们将有两个选项来显示数据。

首先，我们可以使用`th:each `遍历每个同名的参数:

```
<p th:each="emailaddress : ${param.emailaddress}">
    <span th:text="${emailaddress}"></span>
</p>
```

其次，我们可以使用参数数组的索引:

```
<p th:text="${param.emailaddress[0]}"></p>
<p th:text="${param.emailaddress[1]}"></p>
```

## 5.会话属性

或者，我们可以将数据放在一个`HttpSession` 属性中:

```
@GetMapping("/email/sessionattributes")
public String emailSessionAttributes(HttpSession httpSession) {
    httpSession.setAttribute("emaildata", emailData);
    return "mvcdata/email-session-attributes";
}
```

然后，类似于请求参数，我们可以使用`session`关键字:

```
<p th:text="${session.emaildata.emailSubject}"></p>
```

## 6.`ServletContext`属性

对于`ServletContext`，我们将无法使用表达式来访问`emailData`的属性。

为了解决这个问题，我们将每个值作为一个单独的属性传递:

```
@GetMapping("/email/servletcontext")
public String emailServletContext() {
    servletContext.setAttribute("emailsubject", emailData.getEmailSubject());
    servletContext.setAttribute("emailcontent", emailData.getEmailBody());
    servletContext.setAttribute("emailaddress", emailData.getEmailAddress1());
    servletContext.setAttribute("emaillocale", emailData.getEmailLocale());
    return "mvcdata/email-servlet-context";
} 
```

**然后，我们可以通过`servletContext `变量:**检索每一个

```
<p th:text="${#servletContext.getAttribute('emailsubject')}"></p>
```

## 7.豆子

最后，我们还可以使用上下文 beans 提供数据:

```
@Bean
public EmailData emailData() {
    return new EmailData();
} 
```

百里香允许使用`@beanName`语法进行 bean 访问:

```
<p th:text="${@emailData.emailSubject}"></p>
```

## 8.结论

在这个小教程中，我们学习了如何通过百里香访问数据。

首先，我们添加了适当的依赖项。其次，我们实现了一些 REST 方法，这样我们就可以将数据传递给模板。

和往常一样，代码可以在 GitHub 的[上获得。](https://web.archive.org/web/20220728105348/https://github.com/eugenp/tutorials/tree/master/spring-web-modules/spring-thymeleaf-5)