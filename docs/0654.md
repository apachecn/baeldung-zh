# 在百里香叶中获取 URL 属性值

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/thymeleaf-url-attribute-value>

## 1.概观

在这个简短的教程中，我们将阐明**如何在百里香视图**中获得 URL 属性。

## 2.如何获取 URL 参数属性

访问 URL 属性，或者我们称之为请求参数，可以在 Thymleaf 视图中使用两个特殊的 Thymleaf 对象之一轻松完成。第一种方式是使用`param`对象，第二种方式是使用# `request`对象。

出于演示的目的，让我们考虑一个保存一个参数的 URL，`query`:

```
https://baeldung.com/search?query=Baeldung
```

### 2.1.使用`param`对象

首先，让我们看看如何使用`param`对象来访问 URL 属性“query”:

```
<div th:if="${param.query != null}">
    <p th:text="${param.query }"></p>
</div>
```

在上面的例子中，如果参数“query”不为空，将显示“query”的值。另外，**我们应该注意 URL 属性可以是多值的**。让我们看一个具有多值属性的示例 URL:

```
https://baeldung.com/search?query=Bealdung&query;=Thymleaf
```

在这种情况下，我们可以使用括号语法分别访问这些值:

```
<div th:if="${param.query != null}">
    <p th:text="${param.query[0] + ' ' + param.query[1]}" th:unless="${param.query == null}"></p>
</div>
```

### 2.2.使用`request`对象

接下来，让我们看看访问 URL 属性的第二种方法。我们可以使用特殊的# `request`对象，让您可以直接访问`javax.servlet.http.HttpServletRequest`对象，它将请求分解成经过解析的元素，比如查询属性和头。

让我们看看如何在胸腺视图中使用`#request`对象:

```
<div th:if="${#request.getParameter('query') != null}">
    <p th:text="${#request.getParameter('query')}" th:unless="${#request.getParameter('query') == null}"></p>
</div>
```

在上面的例子中，我们使用了由`#request`对象提供的特殊函数`getParameter(‘query')`。该方法将请求参数的值作为`String`返回，如果参数不存在，则返回`null`。

## 3.结论

在这篇简短的文章中，我们解释了**如何使用`param`和`#request`对象在百里香叶视图**中获得 URL 属性。和往常一样，代码片段可以在 GitHub 上找到[。](https://web.archive.org/web/20221129003557/https://github.com/eugenp/tutorials/tree/master/spring-web-modules/spring-thymeleaf-5)