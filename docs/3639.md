# 用 Spring MVC 定制错误页面

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/custom-error-page-spring-mvc>

## 1。概述

任何 web 应用程序中的一个常见需求是定制错误页面。

例如，假设您在 Tomcat 上运行一个普通的 Spring MVC 应用程序。用户在他的浏览器中输入了一个无效的 URL，并显示一个不太用户友好的蓝色和白色堆栈跟踪——不理想。

在本教程中，我们将为一些 HTTP 错误代码设置定制的错误页面。

工作假设是读者对使用 Spring MVC 相当满意；如果没有，[这是开始](/web/20220118043202/https://www.baeldung.com/spring-mvc-tutorial)的好方法。

本文主要关注 Spring MVC。我们的文章[Customize white label Error Page](/web/20220118043202/https://www.baeldung.com/spring-boot-custom-error-page)描述了如何在 Spring Boot 创建一个定制的错误页面。

## 2。简单的步骤

让我们从这里要遵循的简单步骤开始:

1.  在`web.xml`中指定一个 URL `/errors`,该 URL 映射到一个每当产生错误时都会处理错误的方法
2.  使用映射`/errors`创建一个名为`ErrorController`的控制器
3.  在运行时找出 HTTP 错误代码，并根据 HTTP 错误代码`.` 显示一条消息。例如，如果生成了 404 错误，那么用户应该会看到类似于`‘Resource not found' ,` 的消息，而对于 500 错误，用户应该会在 `‘Sorry! An Internal Server Error was generated at our end'`行看到一些内容

## 3。`web.xml`

我们首先将下面几行添加到我们的`web.xml**:**`

```
<error-page>
    <location>/errors</location>
</error-page>
```

请注意，该特性仅在高于 3.0 的 Servlet 版本中可用。

应用程序中生成的任何错误都与 HTTP 错误代码相关联。例如，假设用户在浏览器中输入一个 URL `/invalidUrl` ，但是 Spring 中没有定义这样的`RequestMapping`。然后，底层 web 服务器生成一个 HTTP 代码 404。我们刚刚添加到 `web.xml` 中的代码行告诉 Spring 执行映射到 URL `/errors.` 的方法中编写的逻辑

这里有一个小提示——不幸的是，相应的 Java Servlet 配置没有设置错误页面的 API 所以在这种情况下，我们实际上需要`web.xml`。

## 4。控制器

继续，我们现在创建我们的`ErrorController`。我们创建一个单一的统一方法来拦截错误并显示错误页面:

```
@Controller
public class ErrorController {

    @RequestMapping(value = "errors", method = RequestMethod.GET)
    public ModelAndView renderErrorPage(HttpServletRequest httpRequest) {

        ModelAndView errorPage = new ModelAndView("errorPage");
        String errorMsg = "";
        int httpErrorCode = getErrorCode(httpRequest);

        switch (httpErrorCode) {
            case 400: {
                errorMsg = "Http Error Code: 400\. Bad Request";
                break;
            }
            case 401: {
                errorMsg = "Http Error Code: 401\. Unauthorized";
                break;
            }
            case 404: {
                errorMsg = "Http Error Code: 404\. Resource not found";
                break;
            }
            case 500: {
                errorMsg = "Http Error Code: 500\. Internal Server Error";
                break;
            }
        }
        errorPage.addObject("errorMsg", errorMsg);
        return errorPage;
    }

    private int getErrorCode(HttpServletRequest httpRequest) {
        return (Integer) httpRequest
          .getAttribute("javax.servlet.error.status_code");
    }
} 
```

## 5。前端

出于演示的目的，我们将保持我们的错误页面非常简单和紧凑。该页面将只包含一条显示在白色屏幕上的消息。创建一个名为`errorPage.jsp :`的`jsp` 文件

```
<%@ taglib uri="http://java.sun.com/jsp/jstl/core" prefix="c"%>
<%@ page session="false"%>
<html>
<head>
    <title>Home</title>
</head>
<body>
    <h1>${errorMsg}</h1>
</body>
</html>
```

## 6。测试

我们将模拟任何应用程序中最常见的两个错误:HTTP 404 错误和 HTTP 500 错误。

运行服务器并前往`localhost:8080/spring-mvc-xml/invalidUrl.`因为这个 URL 不存在，我们期望看到我们的错误页面，消息为'`Http Error Code : 404\. Resource not found'.`

让我们看看当一个处理程序方法抛出一个`NullPointerException.` 时会发生什么

```
@RequestMapping(value = "500Error", method = RequestMethod.GET)
public void throwRuntimeException() {
    throw new NullPointerException("Throwing a null pointer exception");
}
```

转到`localhost:8080/spring-mvc-xml/500Error.` ,你应该会看到一个白色的屏幕，上面写着“Http 错误代码:500”。“内部服务器错误”。

## 7。结论

我们看到了如何使用 Spring MVC `.` 为不同的 HTTP 代码设置错误页面，我们创建了一个单一的错误页面，其中根据 HTTP 错误代码动态显示一条错误消息。