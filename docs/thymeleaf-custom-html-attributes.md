# 在百里香叶中使用自定义 HTML 属性

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/thymeleaf-custom-html-attributes>

## 1.概观

在本教程中，我们看看如何使用百里香叶在 HTML5 标签中定义自定义属性。它是一个模板引擎框架，允许使用普通的 HTML 来显示动态数据。

关于百里香或其与 Spring 集成的介绍性文章，请参考此[链接。](/web/20220728105348/https://www.baeldung.com/thymeleaf-in-spring-mvc)

## 2.HTML5 中的自定义属性

有时我们可能需要 HTML 页面中的额外信息来在客户端进行一些处理。例如，我们可能希望在`form` `input`标签中保存额外的数据，以便在使用 AJAX 提交表单时使用它们。

同样，我们可能希望向填写表单的用户显示自定义验证错误消息。

在任何情况下，**我们都可以通过使用 HTML 5 的自定义属性来提供这些额外的数据。可以在 HTML 标签中使用`data-`前缀定义定制属性。**

现在，让我们看看如何使用百里香中的默认方言来定义这些属性。

## 3.百里香中的自定义 HTML 属性

**我们可以使用语法:**在 HTML 标签中指定一个自定义属性

```java
th:data-<attribute_name>=""
```

让我们创建一个简单的表单，允许学生注册一门课程来查看实际情况:

```java
<form action="#" th:action="@{/registerCourse}" th:object="${course}"
    method="post" onsubmit="return validateForm();">
    <span id="errormesg" style="color: red"></span> <span
      style="font-weight: bold" th:text="${successMessage}"></span>
    <table>
        <tr>
            <td>
                <label th:text="#{msg.courseName}+': '"></label>
            </td>
            <td>
                <select id="course" th:field="*{name}">
                    <option th:value="''" th:text="'Select'"></option>
                    <option th:value="'Mathematics'" th:text="'Mathematics'"></option>
                    <option th:value="'History'" th:text="'History'"></option>
                </select></td>
        </tr>
        <tr>
            <td><input type="submit" value="Submit" /></td>
        </tr>
    </table>
</form>
```

该表单包含一个下拉列表，其中有可用的课程和一个提交按钮。

假设我们想向用户显示一个定制的错误消息，如果他们没有选择课程就点击提交。

我们可以将错误消息作为自定义属性保存在`select`标签中，并创建一个 JavaScript 函数在提交表单时验证其值。

更新后的`select`标签如下所示:

```java
<select id="course" th:field="*{name}" th:data-validation-message="#{msg.courseName.mandatory}">
```

这里，我们从资源包中获取错误消息。

现在，当用户在没有选择有效选项的情况下提交表单时，这个 JavaScript 函数将向用户显示一条错误消息:

```java
function validateForm() {
    var e = document.getElementById("course");
    var value = e.options[e.selectedIndex].value;
    if (value == '') {
        var error = document.getElementById("errormesg");
        error.textContent = e.getAttribute('data-validation-message');
        return false;
    }
    return true;
}
```

类似地，我们可以为 HTML 标签添加多个自定义属性，方法是为每个属性定义`th:data-*`属性。

## 3.结论

在这篇简短的文章中，我们探讨了如何利用百里香的支持在 HTML5 模板中添加自定义属性。

与往常一样，Github 上有这段代码的工作版本[。](https://web.archive.org/web/20220728105348/https://github.com/eugenp/tutorials/tree/master/spring-web-modules/spring-thymeleaf-2)