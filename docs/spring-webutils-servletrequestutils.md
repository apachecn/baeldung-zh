# Spring WebUtils 和 ServletRequestUtils 指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-webutils-servletrequestutils>

## 1。概述

在这篇简短的文章中，我们将探索 Spring MVC 中的内置 web 请求实用程序—`WebUtils`、`ServletRequestUtils`。

## 2。`WebUtils`和`ServletRequestUtils`

在几乎所有的应用程序中，我们都面临需要从传入的 HTTP 请求中获取一些参数的情况。

为此，我们必须创建一些非常繁忙的代码段，如:

```
HttpSession session = request.getSession(false);
if (session != null) {
    String foo = session.getAttribute("parameter");
}

String name = request.getParameter("parameter");
if (name == null) {
    name = "DEFAULT";
}
```

使用`WebUtils`和`ServletRequestUtils`，我们只用一行代码就可以做到。

为了了解这些实用程序是如何工作的，让我们创建一个简单的 web 应用程序。

## 3。样本页面

我们需要创建样本页面，以便能够链接的网址。我们将使用 [`Spring Boot`](https://web.archive.org/web/20220815030956/https://search.maven.org/classic/#search%7Cga%7C1%7Ca%3A%22spring-boot%22) 和 [`Thymeleaf`](https://web.archive.org/web/20220815030956/https://search.maven.org/classic/#search%7Cga%7C1%7Ca%3A%22thymeleaf%22) 作为我们的模板引擎。我们需要为它们添加所需的依赖项。

让我们创建一个具有简单表单的页面:

```
<form action="setParam" method="POST">
    <h3>Set Parameter:  </h3>
    <p th:text="${parameter}" class="param"/>
    <input type="text" name="param" id="param"/>
    <input type="submit" value="SET"/>
</form>
<br/>
<a href="other">Another Page</a>
```

正如我们所看到的，我们正在创建一个发起`POST`请求的表单。

还有一个链接，它将用户转到下一页，在这里我们将显示 session 属性中提交的参数。

让我们创建第二个页面:

```
Parameter set by you: <p th:text="${parameter}" class="param"/>
```

## 4。用途

现在我们已经完成了视图的构建，让我们创建控制器并使用`ServletRequestUtils`获取请求参数:

```
@PostMapping("/setParam")
public String post(HttpServletRequest request, Model model) {
    String param 
      = ServletRequestUtils.getStringParameter(
        request, "param", "DEFAULT");

    WebUtils.setSessionAttribute(request, "parameter", param);

    model.addAttribute("parameter", "You set: " + (String) WebUtils
      .getSessionAttribute(request, "parameter"));

    return "utils";
}
```

注意我们如何使用`ServletRequestUtils`中的`getStringParameter` API 来获取请求参数名`param`；如果没有值进入控制器，将为请求参数分配一个默认值。

当然，请注意`WebUtils`中的`setSessionAttribute` API 用于设置会话属性中的值。我们不需要明确检查会话是否已经存在，也不需要检查普通 servlet 中的链接。Spring 会动态配置它。

同样，让我们创建另一个处理程序，它将显示以下会话属性:

```
@GetMapping("/other")
public String other(HttpServletRequest request, Model model) {

    String param = (String) WebUtils.getSessionAttribute(
      request, "parameter");

    model.addAttribute("parameter", param);

    return "other";
}
```

这就是我们创建应用程序所需的全部内容。

这里需要注意的一点是,`ServletRequestUtils`有一些很棒的内置特性，可以根据我们的需要自动对请求参数进行类型转换。

下面是我们如何将请求参数转换为`Long`:

```
Long param = ServletRequestUtils.getLongParameter(request, "param", 1L);
```

类似地，我们可以将请求参数转换为其他类型:

```
boolean param = ServletRequestUtils.getBooleanParameter(
  request, "param", true);

double param = ServletRequestUtils.getDoubleParameter(
  request, "param", 1000);

float param = ServletRequestUtils.getFloatParameter(
  request, "param", (float) 1.00);

int param = ServletRequestUtils.getIntParameter(
  request, "param", 100);
```

另一点需要注意的是，`ServletRequestUtils`有另一个方法`getRequiredStringParameter(ServletRequest request, String name)`来获取请求参数。不同之处在于，如果在传入的请求中没有找到该参数，它将抛出`ServletRequestBindingException`。当我们需要处理关键数据时，这可能很有用。

下面是一个示例代码片段:

```
try {
    ServletRequestUtils.getRequiredStringParameter(request, "param");
} catch (ServletRequestBindingException e) {
    e.printStackTrace();
}
```

我们还可以创建一个简单的 JUnit 测试用例来测试应用程序:

```
@Test
public void givenParameter_setRequestParam_andSetSessionAttribute() 
  throws Exception {
      String param = "testparam";

      this.mockMvc.perform(
        post("/setParam")
          .param("param", param)
          .sessionAttr("parameter", param))
          .andExpect(status().isOk());
  }
```

## 5。结论

在本文中，我们看到使用`WebUtils`和`ServletRequestUtils`可以大大减少大量样板代码的开销。然而，另一方面，它无疑增加了对 Spring 框架的依赖——如果这是一个问题的话，这是需要记住的。

和往常一样，源代码可以在 GitHub 上的[处获得。](https://web.archive.org/web/20220815030956/https://github.com/eugenp/tutorials/tree/master/spring-boot-modules/spring-boot-mvc-4)