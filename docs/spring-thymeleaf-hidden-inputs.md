# 使用弹簧和百里香叶的隐藏输入

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-thymeleaf-hidden-inputs>

## 1.介绍

百里香叶是 Java 生态系统中最流行的模板引擎之一。它允许我们轻松地使用 Java 应用程序中的数据来创建动态 HTML 页面。

在本教程中，我们将看看几种方法来使用弹簧和百里香叶隐藏输入。

## 2.HTML 格式的百里香叶

在我们研究如何使用隐藏字段之前，让我们后退一步，看看百里香如何处理 HTML 表单。

最常见的用例是在我们的应用程序中使用直接映射到 DTO 的 HTML 表单。

例如，假设我们正在编写一个博客应用程序，并且有一个代表单个博客帖子的 DTO:

```
class BlogDTO {
    long id;
    String title;
    String body;
    String category;
    String author;
    Date publishedDate;  
}
```

我们可以使用 HTML 表单通过百里香叶和 Java 创建这个 DTO 的新实例:

```
<form action="#" method="post" th:action="@{/blog}" th:object="${blog}">
    <input type="text" th:field="*{title}">
    <input type="text" th:field="*{category}">
    <textarea th:field="*{body}"></textarea>
</form>
```

注意，我们的博客文章 DTO 中的字段映射到 HTML 表单中的一个输入。这在大多数情况下工作良好，但是哪些字段不应该是可编辑的呢？这就是隐藏输入可以发挥作用的地方。

例如，每篇博客文章都有一个唯一的 ID 字段，不允许用户编辑。使用隐藏输入，我们可以将 ID 字段传递到 HTML 表单中，而不允许它被显示或编辑。

## 3.使用`th:field`属性

为隐藏输入赋值的最快方法是使用`th:field`属性:

```
<input type="hidden" th:field="*{blogId}" id="blogId">
```

这是最简单的方法，因为我们不需要指定 value 属性，**，但是在旧版本的百里叶**中可能不支持。

## 4.使用`th:attr`属性

我们可以在百里香叶中使用隐藏输入的下一种方法是使用内置的`th:attr`属性:

```
<input type="hidden" th:value="${blog.id}" th:attr="name='blogId'"/>
```

在这种情况下，我们必须使用`blog`对象来引用`id`字段。

## 5.使用`name`属性

另一个不太冗长的方法是使用标准的 HTML `name`属性:

```
<input type="hidden" th:value="${blog.id}" name="blogId" />
```

它仅仅依赖于标准的 HTML 属性。在这种情况下，我们还必须使用`blog`对象引用`id`字段。

## 6.结论

在本教程中，我们看了几种使用百里香叶隐藏输入的方法。这是一种将只读字段从 dto 传递到 HTML 表单的有用技术。

一如既往，本教程中使用的所有代码示例都可以在 Github 上找到[。](https://web.archive.org/web/20220727020730/https://github.com/eugenp/tutorials/tree/master/spring-web-modules/spring-thymeleaf-3)