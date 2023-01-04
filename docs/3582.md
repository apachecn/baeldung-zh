# 百里香叶变量

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/thymeleaf-variables>

## 1.介绍

在本教程中，我们将看看[百里香叶](/web/20221126214451/https://www.baeldung.com/thymeleaf-in-spring-mvc)中的变量。我们将创建一个 Spring Boot 示例，该示例将获取 Baeldung 文章列表，并在百里香 HTML 模板中显示它们。

## 2.Maven 依赖性

要使用百里香叶，我们需要添加`[spring-boot-starter-thymeleaf](https://web.archive.org/web/20221126214451/https://search.maven.org/search?q=a:spring-boot-starter-thymeleaf%20g:org.springframework.boot)`和`[spring-boot-starter-web](https://web.archive.org/web/20221126214451/https://search.maven.org/search?q=a:spring-boot-starter-web%20g:org.springframework.boot)`依赖项:

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-thymeleaf</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

## 3.网络控制器

首先，我们将创建一个带有 GET 端点的 web 控制器，它返回一个包含 Baeldung 文章列表的页面。

用`@GetMapping`标注的方法将接受一个参数——T1。它保存了所有可以在百里香模板中进一步使用的全局变量。在我们的例子中，模型只有一个参数——文章列表。

`Article`类将由两个`String`字段`name`和`url`组成:

```
public class Article {
    private String name;
    private String url;

    // constructor, getters and setters
}
```

我们的控制器方法的返回值应该是所需的百里香模板的名称。这个名称应该对应于位于`src/resource/template` 目录中的 HTML 文件。在我们的例子中，它将是`src/resource/template/articles-list.html`。

让我们快速看一下我们的弹簧控制器:

```
@Controller
@RequestMapping("/api/articles")
public class ArticlesController {

    @GetMapping
    public String allArticles(Model model) {
        model.addAttribute("articles", fetchArticles());
        return "articles-list";
    }

    private List<Article> fetchArticles() {
        return Arrays.asList(
          new Article(
            "Introduction to Using Thymeleaf in Spring",
            "https://www.baeldung.com/thymeleaf-in-spring-mvc"
          ),
          // a few other articles
        );
    }
}
```

运行应用程序后，文章页面将在`http://localhost:8080/articles`可用。

## 4.百里香叶模板

现在，让我们进入百里香 HTML 模板。它应该具有标准的 HTML 文档结构，只带有额外的百里香命名空间定义:

```
<html xmlns:th="http://www.thymeleaf.org">
```

我们将在接下来的例子中使用它作为模板，我们将只替换`<main>`标签的内容:

```
<!DOCTYPE HTML>
<html xmlns:th="http://www.thymeleaf.org">
<head>
    <title>Thymeleaf Variables</title>
    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8" />
</head>
<body>
    <main>
        ...
    </main>
</body>
</html>
```

## 5.定义变量

有两种方法可以在百里香模板中定义变量。第一种选择是在迭代数组时采用单个元素:

```
<div th:each="article : ${articles}">
    <a th:text="${article.name}" th:href="${article.url}"></a>
</div>
```

因此，我们将得到一个包含几个与`articles`变量中文章数量相对应的`<a>`元素的`<div>`。

另一种方法是基于另一个变量定义一个新变量。例如，我们可以获取 articles 数组的第一个元素:

```
<div th:with="firstArticle=${articles[0]}">
    <a th:text="${firstArticle.name}" th:href="${firstArticle.url}"></a>
</div>
```

或者我们可以创建一个新变量，只保存文章的名称:

```
<div th:each="article : ${articles}", th:with="articleName=${article.name}">
    <a th:text="${articleName}" th:href="${article.url}"></a>
</div>
```

在上面的例子中，`${article.name}`和`${articleName}`片段是可替换的。

也可以定义多个变量。例如，我们可以创建两个单独的变量来保存文章名称和 URL:

```
<div th:each="article : ${articles}" th:with="articleName=${article.name}, articleUrl=${article.url}">
    <a th:text="${articleName}" th:href="${articleUrl}"></a>
</div>
```

## 6.变量范围

传递给控制器中的`Model`的变量具有全局范围。这意味着它们可以用在我们 HTML 模板的任何地方。

另一方面，HTML 模板中定义的变量有一个局部范围。它们只能在定义它们的元素范围内使用。

例如，下面的代码是正确的，因为`<a>`元素在`firstDiv`中:

```
<div id="firstDiv" th:with="firstArticle=${articles[0]}">
    <a th:text="${firstArticle.name}" th:href="${firstArticle.url}"></a>
</div>
```

另一方面，当我们试图在另一个`div`中使用`firstArticle`时:

```
<div id="firstDiv" th:with="firstArticle=${articles[0]}">
    <a th:text="${firstArticle.name}" th:href="${firstArticle.url}"></a>
</div>
<div id="secondDiv">
    <h2 th:text="${firstArticle.name}"></h2>
</div>
```

我们会在编译时得到一个异常，说`firstArticle`是`null`:

```
org.springframework.expression.spel.SpelEvaluationException: EL1007E: Property or field 'name' cannot be found on null
```

这是因为`<h2>`元素试图使用在`firstDiv,`中定义的超出范围的变量。

如果我们仍然需要在`secondDiv`中使用`firstArticle`变量，我们将需要在`secondDiv`中再次定义它，或者将这两个`div`标签包装在一个公共元素中，并在其中定义`firstArticle`。

## 7.更改变量值

也可以在给定范围内覆盖变量值:

```
<div id="mainDiv" th:with="articles = ${ { articles[0], articles[1] } }">
    <div th:each="article : ${articles}">
        <a th:text="${article.name}" th:href="${article.url}"></a>
    </div>
</div>
```

在上面的例子中，我们重新定义了`articles`变量，只有两个前元素。

请注意，在`mainDiv,`之外，`articles`变量仍会将其原始值传递给控制器。

## 8.结论

在本教程中，我们学习了如何在百里香中定义和使用变量。和往常一样，GitHub 上的所有源代码[都是可用的。](https://web.archive.org/web/20221126214451/https://github.com/eugenp/tutorials/tree/master/spring-web-modules/spring-thymeleaf-3)