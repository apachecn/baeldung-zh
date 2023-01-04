# 在 JavaScript 中访问 Spring MVC 模型对象

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-mvc-model-objects-js>

## 1.概观

在本教程中，我们将展示如何在包含 JavaScript 代码的百里香视图中访问 Spring MVC 对象。在我们的例子中，我们将使用 Spring Boot 和百里香模板引擎，但是这个想法也适用于其他模板引擎。

我们将描述两种情况:当 JavaScript 代码嵌入或内置于引擎生成的网页时，以及当它在页面外部时——例如，在一个单独的 JavaScript 文件中。

## 2.设置

让我们假设我们已经配置了一个使用百里香模板引擎的 Spring Boot web 应用程序。否则，您可能会发现这些教程非常有用:

*   [Bootstrap 一个简单的应用程序](/web/20221208143917/https://www.baeldung.com/spring-boot-start)-讲述如何从头开始创建一个 Spring Boot 应用程序
*   Spring MVC +百里香叶 3.0:新特性-关于如何使用百里香叶语法

此外，让我们假设我们的应用程序有一个对应于端点`/index`的控制器，它从名为`index.html`的模板中呈现一个视图。这个模板可能包括一个嵌入的或者外部的 JavaScript 代码，比如说`script.js`。

我们的目标是能够从嵌入式或外部 JavaScript (JS)代码中访问 Spring MVC 参数。

## 3.访问参数

首先，我们需要从 JS 代码中创建我们想要使用的模型变量。

在 Spring MVC 中，有多种方法可以做到这一点。让我们使用`ModelAndView`方法:

```java
@RequestMapping("/index")
public ModelAndView thymeleafView(Map<String, Object> model) {
    model.put("number", 1234);
    model.put("message", "Hello from Spring MVC");
    return new ModelAndView("thymeleaf/index");
} 
```

我们可以在我们关于 Spring MVC 中[`Model``ModelMap``ModelView`的教程中找到其他的可能性。](/web/20221208143917/https://www.baeldung.com/spring-mvc-model-model-map-model-view)

## 4.嵌入式 JS 代码

嵌入式 JS 代码只是位于`<script>`元素内部的 `index.html`文件的一部分。我们可以非常直接地传递 Spring MVC 变量:

```java
<script>
    var number = [[${number}]];
    var message = "[[${message}]]";
</script>
```

百里香模板引擎用当前执行范围内可用的值替换每个表达式。这意味着模板引擎将上述代码转换为以下 HTML 代码:

```java
<script>
    var number = 1234;
    var message = "Hello from Spring MVC!";
</script>
```

## 5.外部 JS 代码

假设我们的外部 JS 代码包含在使用相同的`<script>`标签的 `index.html`文件中，其中我们指定了`src`属性:

```java
<script src="/js/script.js"></script>
```

现在，如果我们想使用来自`script.js`的 Spring MVC 参数，我们应该:

1.  像我们在上一节中所做的那样，在嵌入式 JS 代码中定义 JS 变量
2.  从`script.js `文件中访问这些变量

**注意，外部 JS 代码应该在嵌入式 JS 代码**的变量初始化之后调用。

这可以通过两种方式实现:指定 JS 代码执行的顺序，或者使用异步 JS 代码执行。

### 5.1.指定执行顺序

我们可以通过在`index.html`中嵌入的 JS 代码之后声明外部 JS 代码来做到这一点:

```java
<script>
    var number = [[${number}]];
    var message = "[[${message}]]";
</script>
<script src="/js/script.js"></script>
```

### 5.2.异步代码执行

在这种情况下，我们在`index.html`中声明外部和嵌入式 JS 代码的顺序并不重要，但是我们应该将来自`script.js`的代码放入一个典型的页面加载包装器中:

```java
window.onload = function() {
    // JS code
};
```

尽管这段代码很简单，但最常见的做法是使用`jQuery `来代替。我们将这个库作为第一个`<script>`元素包含在`index.html`文件中:

```java
<!DOCTYPE html>
<html>
    <head>
        <script src="/js/jquery.js"></script>
        ...
    </head>
 ...
</html>
```

现在，我们可以将 JS 代码放在下面的`jQuery`包装器中:

```java
$(function () {
    // JS code
});
```

有了这个包装器，我们可以保证 JS 代码只有在整个页面内容以及所有其他嵌入的 JS 代码都被完全加载时才会被执行。

## 6.一点解释

在 Spring MVC 中，百里香模板引擎只解析模板文件(在我们的例子中是`index.html`),并将其转换成 HTML 文件。反过来，该文件可能包含对模板引擎范围之外的其他资源的引用。是用户的浏览器解析这些资源并呈现 HTML 视图。

因此，模板引擎不会解析这些资源，我们可以只将控制器中定义的变量注入到嵌入的 JS 代码中，然后外部 JS 代码就可以使用这些变量了。

## 7.结论

在本教程中，我们学习了如何在 JavaScript 代码中访问 Spring MVC 参数。

和往常一样，完整的代码示例在我们的 GitHub 库中[。](https://web.archive.org/web/20221208143917/https://github.com/eugenp/tutorials/tree/master/spring-web-modules/spring-mvc-java)