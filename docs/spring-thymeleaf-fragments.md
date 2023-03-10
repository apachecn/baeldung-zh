# 在百里香叶中处理片段

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-thymeleaf-fragments>

## 1。概述

在本教程中，我们将展示如何**利用百里香叶碎片来重用某个站点**的一些公共部分。在建立了一个非常简单的 Spring MVC 项目之后，我们将把重点放在视图上。

如果你是百里香的新手，你可以看看这个网站上的其他文章，如[这篇介绍](/web/20221122041149/https://www.baeldung.com/thymeleaf-in-spring-mvc)，以及[这篇关于 3.0 版本](/web/20221122041149/https://www.baeldung.com/spring-thymeleaf-3)引擎的文章。

## 2。Maven 依赖关系

我们需要几个依赖项来启用百里香:

```java
<dependency>
    <groupId>org.thymeleaf</groupId>
    <artifactId>thymeleaf</artifactId>
    <version>3.0.11.RELEASE</version>
</dependency>
<dependency>
    <groupId>org.thymeleaf</groupId>
    <artifactId>thymeleaf-spring5</artifactId>
    <version>3.0.11.RELEASE</version>
</dependency>
```

百里香叶和 T2 的最新版本可以在 Maven Central 上找到。

## 3。春季项目

### 3.1。Spring MVC 配置

要启用百里香并设置模板后缀，我们需要**用视图解析器和模板解析器**配置 MVC。

我们还将为一些静态资源设置目录:

```java
@Bean
public ViewResolver htmlViewResolver() {
    ThymeleafViewResolver resolver = new ThymeleafViewResolver();
    resolver.setTemplateEngine(templateEngine(htmlTemplateResolver()));
    resolver.setContentType("text/html");
    resolver.setCharacterEncoding("UTF-8");
    resolver.setViewNames(ArrayUtil.array("*.html"));
    return resolver;
}

private ITemplateResolver htmlTemplateResolver() {
    SpringResourceTemplateResolver resolver
      = new SpringResourceTemplateResolver();
    resolver.setApplicationContext(applicationContext);
    resolver.setPrefix("/WEB-INF/views/");
    resolver.setCacheable(false);
    resolver.setTemplateMode(TemplateMode.HTML);
    return resolver;
}

@Override
public void addResourceHandlers(ResourceHandlerRegistry registry) {
    registry.addResourceHandler("/resources/**", "/css/**")
      .addResourceLocations("/WEB-INF/resources/", "/WEB-INF/css/");
}
```

注意，如果我们使用 Spring Boot，这个配置可能不是必需的，除非我们需要应用我们自己的定制。

### 3.2。控制器

在这种情况下，控制器只是视图的载体。每个视图显示了不同的片段使用场景。

最后一个加载了一些数据，这些数据通过模型显示在视图上:

```java
@Controller
public class FragmentsController {

    @GetMapping("/fragments")
    public String getHome() {
        return "fragments.html";
    }

    @GetMapping("/markup")
    public String markupPage() {
        return "markup.html";
    }

    @GetMapping("/params")
    public String paramsPage() {
        return "params.html";
    }

    @GetMapping("/other")
    public String otherPage(Model model) {
        model.addAttribute("data", StudentUtils.buildStudents());
        return "other.html";
    }
}
```

**注意，由于我们配置解析器的方式，视图名称必须包含`“.html”`后缀。**当我们提到片段名时，我们也会指定后缀。

## 4。观点

### 4.1。简单片段包含

首先，我们将在页面中重用公共部分。

我们可以将这些部分定义为片段，或者在独立的文件中，或者在一个公共页面中。在这个项目中，这些可重用部件被定义在一个名为`fragments`的文件夹中。

**包含片段内容有三种基本方式:**

*   `**insert –**`在标签内插入内容
*   `**replace –**`用定义片段的标签替换当前标签
*   这已被否决，但它仍可能出现在遗留代码中

下一个例子，`fragments.html,`展示了所有三种方式的使用。这个百里香模板在文档的头和正文中添加了片段:

```java
<!DOCTYPE HTML>
<html xmlns:th="http://www.thymeleaf.org">
<head>
<title>Thymeleaf Fragments: home</title>
<!--/*/ <th:block th:include="fragments/general.html :: headerfiles">
        </th:block> /*/-->
</head>
<body>
    <header th:insert="fragments/general.html :: header"> </header>
    <p>Go to the next page to see fragments in action</p>
    <div th:replace="fragments/general.html :: footer"></div>
</body>
</html>
```

**现在，让我们来看一个包含一些片段的页面。**它被称为`general.html`，它就像一整页，其中一些部分被定义为随时可用的片段:

```java
<!DOCTYPE HTML>
<html xmlns:th="http://www.thymeleaf.org">
<head th:fragment="headerfiles">
<meta charset="UTF-8" />
<link th:href="@{/css/styles.css}" rel="stylesheet">
</head>
<body>
    <div th:fragment="header">
        <h1>Thymeleaf Fragments sample</h1>
    </div>
    <p>Go to the next page to see fragments in action</p>
    <aside>
        <div>This is a sidebar</div>
    </aside>
    <div class="another">This is another sidebar</div>
    <footer th:fragment="footer">
        <a th:href="@{/fragments}">Fragments Index</a> | 
        <a th:href="@{/markup}">Markup inclussion</a> | 
        <a th:href="@{/params}">Fragment params</a> | 
        <a th:href="@{/other}">Other</a>
    </footer>
</body>
</html>
```

`<head>`部分只包含一个样式表，但是我们可以直接或使用 Webjars 应用其他工具，比如 Bootstrap、jQuery 或 Foundation。

注意，这个模板的所有可重用标签都有属性`th:fragment`，但是接下来我们将看到如何包含页面的任何其他部分。

在渲染和片段包含之后，返回的内容是:

```java
<!DOCTYPE HTML>
<html>
<head>
<title>Thymeleaf Fragments: home</title>
<meta charset="UTF-8" />
<link href="/spring-thymeleaf/css/styles.css" rel="stylesheet">
</head>
<body>
    <header>
        <div>
            <h1>Thymeleaf Fragments sample</h1>
        </div>
    </header>
    <p>Go to the next page to see fragments in action</p>
    <footer>
        <a href="/spring-thymeleaf/fragments">Fragments Index</a> | 
        <a href="/spring-thymeleaf/markup">Markup inclussion</a> | 
        <a href="/spring-thymeleaf/params">Fragment params</a> | 
        <a href="/spring-thymeleaf/other">Other</a>
    </footer>
</body>
</html>
```

### 4.2。片段的标记选择器

百里香片段的一个伟大之处在于**我们也可以通过简单的选择器**，通过类、id 或者简单地通过标签来获取模板的任何部分。

例如，该页面包括来自`general.html`文件的一些组件:一个`aside`块和一个`div.another`块:

```java
<body>
    <header th:insert="fragments/general.html :: header"> </header>
    <div th:replace="fragments/general.html :: aside"></div>
    <div th:replace="fragments/general.html :: div.another"></div>
    <div th:replace="fragments/general.html :: footer"></div>
</body> 
```

### 4.3。参数化片段

我们可以向一个片段传递参数，以便改变它的某个特定部分。为此，片段必须被定义为一个函数调用，其中我们必须声明一个参数列表。

在本例中，我们为通用表单字段定义了一个片段:

```java
<div th:fragment="formField (field, value, size)">
    <div>
        <label th:for="${#strings.toLowerCase(field)}"> <span
            th:text="${field}">Field</span>
        </label>
    </div>
    <div>
        <input type="text" th:id="${#strings.toLowerCase(field)}"
            th:name="${#strings.toLowerCase(field)}" th:value="${value}"
            th:size="${size}">
    </div>
</div>
```

下面是这个片段的一个简单用法，我们向它传递参数:

```java
<body>
    <header th:insert="fragments/general.html :: header"> </header>
    <div th:replace="fragments/forms.html
      :: formField(field='Name', value='John Doe',size='40')">
    </div>
    <div th:replace="fragments/general.html :: footer"></div>
</body> 
```

返回的字段将是这样的:

```java
<div>
    <div>
        <label for="name"> <span>Name</span>
        </label>
    </div>
    <div>
        <input type="text" id="name"
        name="name" value="John Doe"
        size="40">
    </div>
</div> 
```

### 4.4。片段包含表达式

百里香叶片段提供了其他有趣的选项，比如支持**条件表达式来决定是否包含片段**。

将`Elvis`操作符与 Thymeleaf 提供的任何表达式(例如安全性、字符串和集合)一起使用，我们能够加载不同的片段。

例如，我们可以用一些内容来定义这个片段，我们将根据给定的条件来显示这些内容。这可能是一个包含不同类型块的文件:

```java
<div th:fragment="dataPresent">Data received</div>
<div th:fragment="noData">No data</div>
```

这就是我们如何用一个表达式来装载它们:

```java
<div
    th:replace="${#lists.size(data) > 0} ? 
        ~{fragments/menus.html :: dataPresent} : 
        ~{fragments/menus.html :: noData}">
</div>
```

要了解更多关于百里香叶的表达，请点击这里查看我们的文章。

### 4.5。灵活的布局

下一个例子还展示了片段的另外两个有趣的用法，即**用数据**呈现一个表格。这是可重用的表片段，有两个重要部分:可以更改的表头和呈现数据的表体:

```java
<table>
    <thead th:fragment="fields(theadFields)">
        <tr th:replace="${theadFields}">
        </tr>
    </thead>
    <tbody th:fragment="tableBody(tableData)">
        <tr th:each="row: ${tableData}">
            <td th:text="${row.id}">0</td>
            <td th:text="${row.name}">Name</td>
        </tr>
    </tbody>
    <tfoot>
    </tfoot>
</table>
```

当我们想要使用这个表时，我们可以使用`fields`函数传递我们自己的表头。头文件被类`myFields`引用。通过将数据作为参数传递给`tableBody`函数来加载表体:

```java
<body>
    <header th:replace="fragments/general.html :: header"> </header>
    <table>
        <thead th:replace="fragments/tables.html
              :: fields(~{ :: .myFields})">
            <tr class="myFields">

                <th>Id</th>
                <th>Name</th>
            </tr>
        </thead>
        <div th:replace="fragments/tables.html
          :: tableBody(tableData=${data})">
        </div>
    </table>
    <div th:replace="fragments/general.html :: footer"></div>
</body> 
```

这是最终页面的样子:

```java
<body>
    <div>
        <h1>Thymeleaf Fragments sample</h1>
    </div>
    <div>Data received</div>
    <table>
        <thead>
            <tr class="myFields">

                <th>Id</th>
                <th>Name</th>
            </tr>
        </thead>
        <tbody>
            <tr>
                <td>1001</td>
                <td>John Smith</td>
            </tr>
            <tr>
                <td>1002</td>
                <td>Jane Williams</td>
            </tr>
        </tbody>
    </table>
    <footer>
        <a href="/spring-thymeleaf/fragments">Fragments Index</a> |
        <a href="/spring-thymeleaf/markup">Markup inclussion</a> |
        <a href="/spring-thymeleaf/params">Fragment params</a> |
        <a href="/spring-thymeleaf/other">Other</a>
    </footer>
</body> 
```

## 5。结论

在本文中，我们展示了如何通过使用百里香叶片段来重用视图组件，这是一个强大的工具，可以使模板管理更加容易。

我们还展示了一些超出基础的其他有趣特性。当选择百里香叶作为我们的视图渲染引擎时，我们应该考虑这些因素。

如果你想了解百里香的其他特性，你一定要看看我们关于[布局方言](/web/20221122041149/https://www.baeldung.com/thymeleaf-spring-layouts)的文章。

与往常一样，GitHub 上的[提供了该示例的完整实现代码。](https://web.archive.org/web/20221122041149/https://github.com/eugenp/tutorials/tree/master/spring-web-modules/spring-thymeleaf)