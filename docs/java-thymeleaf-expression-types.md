# 百里香叶中的表达类型

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-thymeleaf-expression-types>

## 1.概观

[百里叶](https://web.archive.org/web/20221122042753/https://www.thymeleaf.org/) 是流行于 Java 生态系统中的模板引擎。它有助于将数据从控制器层绑定到视图层。**使用[表达式](/web/20221122042753/https://www.baeldung.com/spring-thymeleaf-3-expressions)** 设置百里叶属性。在本教程中，我们将通过例子来讨论表达式类型。

## 2.示例设置

我们将使用一个简单的 web 应用程序 Dino 作为例子。这是一个简单的创建恐龙档案的网络应用程序。

首先，让我们为恐龙创建一个模型类:

```java
public class Dino {
    private int id;
    private String name;
    // constructors   
    // getter and setter
} 
```

接下来，让我们创建一个控制器类:

```java
@Controller
public class DinoController {

    @RequestMapping("/")
    public String dinoList(Model model) {
        Dino dinos = new Dino(1, "alpha", "red", 50);
        model.addAttribute("dinos", dinos);
        return "index";
    }
}
```

使用我们的示例设置，我们将能够将一个`Dino`实例注入到模板文件中。

## 3.可变表达式

**变量表达式帮助将数据从控制器注入模板文件**。它向 web 视图公开模型属性。

变量表达式语法是美元符号和花括号的组合。我们的变量名位于花括号内:

```java
${...}
```

让我们将`Dino`数据注入模板文件:

```java
<span th:text="${dinos.id}"></span> 
<span th:text="${dinos.name}"></span>
```

[条件句](/web/20221122042753/https://www.baeldung.com/spring-thymeleaf-conditionals)和[迭代](/web/20221122042753/https://www.baeldung.com/thymeleaf-iteration)也可以使用变量表达式:

```java
<!-- for iterating -->
<div th:each="dinos : ${dinos}">

<!-- in conditionals -->
<div th:if="${dinos.id == 2}">
```

## 4.选择表达式

**Selection expression operates on a previously** **chosen object**. It helps us select the child of the chosen object.

选择表达式语法是星号和花括号的组合。我们的子对象位于花括号内:

```java
*{...}
```

让我们选择我们的`Dino`实例的`id`和`name`，并将其注入到我们的模板文件中:

```java
<div th:object="${dinos}">
    <p th:text="*{id}">
    <p th:text="*{name}">
</div>
```

此外， **选择表达式主要用于 HTML** 中的表单内。 帮助绑定表单输入和模型属性 。

与变量表达式不同，不需要单独对待每个输入元素。以我们的 Dino web 应用程序为例，让我们创建一个新的`Dino` 实例，并将其绑定到我们的模型属性:

```java
<form action="#" th:action="@{/dino}" th:object="${dinos}" method="post">
    <p>Id: <input type="text" th:field="*{id}" /></p>
    <p>Name: <input type="text" th:field="*{name}" /></p>
</form>
```

## 5.消息表达式

这个表达式有助于将外部化的文本放入我们的模板文件。也叫文本外化。

我们的文本所在的外部源可以是一个`.properties`文件。**此表达式在有占位符**时是动态的。

消息表达式语法是哈希和花括号的组合。我们的密钥在花括号里:

```java
#{...}
```

例如，假设我们想要在 Dino web 应用程序的所有页面中显示特定的消息。我们可以将消息放在一个`messages.properties`文件中:

```java
welcome.message=welcome to Dino world. 
```

为了将欢迎消息绑定到我们的视图模板，我们可以通过它的键来引用它:

```java
<h2 th:text="#{welcome.message}"></h2>
```

通过在外部文件中添加一个占位符，我们可以让消息表达式接受参数:

```java
dino.color=red is my favourite, mine is {0}
```

在我们的模板文件中，我们将引用消息并向占位符添加一个值:

```java
<h2 th:text="#{dino.color('blue')}"></h2>
```

此外，我们可以通过注入一个变量表达式作为占位符的值，使占位符成为动态的:

```java
<h2 th:text="#{dino.color(${dino.color})}"></h2>
```

This expression is also called internationalization. It can help us adapt our web application to accommodate different languages.

## 6.链接表达式

Link expressions are integral in URL building. **This expression binds to the specified URL**.The link expression syntax is a combination of the “at” sign and curly braces. Our link resides inside the curly braces:

```java
@{...}
```

URL 可以是绝对的，也可以是相对的。当使用带有绝对 URL 的链接表达式时，它会绑定到以“`http(s)`”开头的完整 URL:

```java
<a th:href="@{http://www.baeldung.com}"> Baeldung Home</a>
```

另一方面，相对链接绑定到我们的 web 服务器的上下文。我们可以轻松地浏览控制器中定义的模板文件:

```java
@RequestMapping("/create")
public String dinoCreate(Model model) {
    model.addAttribute("dinos", new Dino());
    return "form";
}
```

我们可以请求 `@RequestMapping` : 中指定的页面

```java
<a th:href="@{/create}">Submit Another Dino</a>
```

**它可以通过路径变量** 取参数。让我们假设我们想要提供一个链接来编辑一个现有的实体。我们可以通过它的 `id` 调用我们想要编辑的对象。链接表达式可以接受 `id` 作为参数:

```java
<a th:href="/@{'/edit/' + ${dino.id}}">Edit</a>
```

链接表达式可以设置协议相关的 URL。相对协议类似于绝对 URL。URL 将使用 HTTP 或 HTTPS 协议方案，具体取决于服务器的协议:

```java
<a th:href="@{//baeldung.com}">Baeldung</a>
```

## 7.片段表达式

片段表达式可以帮助我们在模板文件之间移动标记。 **这个表达式使我们能够生成一个可移动的标记片段。**

片段表达式语法是波浪号和花括号的组合。我们的片段位于花括号内:

```java
~{...}
```

对于我们的 Dino web 应用程序，让我们在 `index.html` 文件中创建一个带有`fragment`属性的页脚:

```java
<div th:fragment="footer">
    <p>Copyright 2022</p>
</div>
```

我们现在可以将`footer`注入其他模板文件:

```java
<div th:replace="~{index :: footer}"></div>
```

## 8.结论

在这篇文章中，我们看了各种百里香简单的表达和例子。

与往常一样，GitHub 上的[提供了示例的完整源代码。](https://web.archive.org/web/20221122042753/https://github.com/eugenp/tutorials/tree/master/spring-web-modules/spring-thymeleaf-5)