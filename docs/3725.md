# 用 Jsoup 解析 Java 中的 HTML

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-with-jsoup>

## 1。概述

Jsoup 是一个开源的 Java 库，主要用于从 HTML 中提取数据。它还允许您操作和输出 HTML。它有稳定的开发路线、优秀的文档和流畅灵活的 API。Jsoup 也可以用来解析和构建 XML。

在本教程中，我们将使用 [Spring Blog](https://web.archive.org/web/20221208143845/https://spring.io/blog) 来演示一个抓取练习，它展示了 jsoup 的几个特性:

*   加载:获取并解析 HTML 到一个`Document`
*   过滤:将需要的数据选择到`Elements`中并遍历
*   提取:获取节点的属性、文本和 HTML
*   修改:添加/编辑/删除节点并编辑其属性

## 2。Maven 依赖关系

为了在您的项目中使用 jsoup 库，将依赖项添加到您的`pom.xml`:

```
<dependency>
    <groupId>org.jsoup</groupId>
    <artifactId>jsoup</artifactId>
    <version>1.10.2</version>
</dependency>
```

您可以在 Maven 中央存储库中找到最新版本的 [jsoup](https://web.archive.org/web/20221208143845/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22org.jsoup%22%20AND%20a%3A%22jsoup%22) 。

## 3。j 群组一览

Jsoup 加载页面 HTML 并构建相应的 DOM 树。该树的工作方式与浏览器中的 DOM 相同，提供了类似于 jQuery 和普通 JavaScript 的方法来选择、遍历、操作文本/HTML/属性以及添加/删除元素。

如果您熟悉客户端选择器和 DOM 遍历/操作，您会发现 jsoup 非常熟悉。检查打印页面段落的难易程度:

```
Document doc = Jsoup.connect("http://example.com").get();
doc.select("p").forEach(System.out::println);
```

请记住，jsoup 只解释 HTML 它不解释 JavaScript。因此，在支持 JavaScript 的浏览器中，通常会在页面加载后发生的对 DOM 的更改在 jsoup 中看不到。

## 4。正在加载

加载阶段包括获取 HTML 并将其解析成一个`[Document](https://web.archive.org/web/20221208143845/https://jsoup.org/apidocs/org/jsoup/nodes/Document.html)`。Jsoup 保证解析任何 HTML，从最无效的到完全有效的，就像现代浏览器一样。可以通过加载一个`String`、`InputStream`、`File`或者一个 URL 来实现。

让我们从 Spring 博客的 URL 加载一个`Document`:

```
String blogUrl = "https://spring.io/blog";
Document doc = Jsoup.connect(blogUrl).get();
```

注意`get`方法，它代表一个 HTTP GET 调用。您也可以用`post`方法进行 HTTP POST(或者您可以使用一个`method`，它接收 HTTP 方法类型作为参数)。

如果需要检测异常状态代码(如 404)，应该捕捉`HttpStatusException`异常:

```
try {
   Document doc404 = Jsoup.connect("https://spring.io/will-not-be-found").get();
} catch (HttpStatusException ex) {
   //...
}
```

有时，连接需要更加个性化。`Jsoup.connect(…)`返回一个`Connection`,允许您设置用户代理、推荐人、连接超时、cookies、发布数据和标题等:

```
Connection connection = Jsoup.connect(blogUrl);
connection.userAgent("Mozilla");
connection.timeout(5000);
connection.cookie("cookiename", "val234");
connection.cookie("cookiename", "val234");
connection.referrer("http://google.com");
connection.header("headersecurity", "xyz123");
Document docCustomConn = connection.get();
```

由于连接遵循流畅的接口，因此您可以在调用所需的 HTTP 方法之前链接这些方法:

```
Document docCustomConn = Jsoup.connect(blogUrl)
  .userAgent("Mozilla")
  .timeout(5000)
  .cookie("cookiename", "val234")
  .cookie("anothercookie", "ilovejsoup")
  .referrer("http://google.com")
  .header("headersecurity", "xyz123")
  .get();
```

你可以通过[浏览相应的 Javadoc](https://web.archive.org/web/20221208143845/https://jsoup.org/apidocs/org/jsoup/Connection.html) 来了解更多关于`Connection`的设置。

## 5。过滤

现在我们已经将 HTML 转换成了一个`Document`，是时候导航它并找到我们想要的东西了。这就是与 jQuery/JavaScript 的相似之处，因为它的选择器和遍历方法是相似的。

### 5.1。选择

`Document` `select`方法接收代表选择器的`String`，使用与 CSS 或 JavaScript 中相同的选择器语法[，并检索匹配列表`Elements`。该列表可以为空，但不能为`null`。](https://web.archive.org/web/20221208143845/https://api.jquery.com/category/selectors/)

让我们来看看使用`select`方法的一些选择:

```
Elements links = doc.select("a");
Elements sections = doc.select("section");
Elements logo = doc.select(".spring-logo--container");
Elements pagination = doc.select("#pagination_control");
Elements divsDescendant = doc.select("header div");
Elements divsDirect = doc.select("header > div");
```

您还可以使用受浏览器 DOM 启发的更明确的方法，而不是通用的`select`:

```
Element pag = doc.getElementById("pagination_control");
Elements desktopOnly = doc.getElementsByClass("desktopOnly");
```

由于`Element`是`Document`的超类，您可以了解更多关于使用`[Document](https://web.archive.org/web/20221208143845/https://jsoup.org/apidocs/org/jsoup/nodes/Document.html)`和[`Element`](https://web.archive.org/web/20221208143845/https://jsoup.org/apidocs/org/jsoup/nodes/Element.html)javadoc 中的选择方法的信息。

### 5.2。穿越

**遍历意味着在 DOM 树中导航。** Jsoup 提供了在`Document`、一组`Elements,`或特定的`Element`上操作的方法，允许您导航到一个节点的父节点、兄弟节点或子节点。

此外，您可以跳转到一组`Elements`中的第一个、最后一个和第 n 个`Element`(使用基于 0 的索引):

```
Element firstSection = sections.first();
Element lastSection = sections.last();
Element secondSection = sections.get(2);
Elements allParents = firstSection.parents();
Element parent = firstSection.parent();
Elements children = firstSection.children();
Elements siblings = firstSection.siblingElements();
```

您还可以迭代选择。事实上，`Elements`类型的任何东西都可以被迭代:

```
sections.forEach(el -> System.out.println("section: " + el));
```

您可以将选择限制为前一个选择(子选择):

```
Elements sectionParagraphs = firstSection.select(".paragraph");
```

## 6。提取

我们现在知道了如何访问特定的元素，所以是时候获取它们的内容了——即它们的属性、HTML 或子文本。

看一下这个例子，它从博客中选择第一篇文章，并获取它的日期、它的第一部分文本，最后是它的内部和外部 HTML:

```
Element firstArticle = doc.select("article").first();
Element timeElement = firstArticle.select("time").first();
String dateTimeOfFirstArticle = timeElement.attr("datetime");
Element sectionDiv = firstArticle.select("section div").first();
String sectionDivText = sectionDiv.text();
String articleHtml = firstArticle.html();
String outerHtml = firstArticle.outerHtml();
```

以下是选择和使用选择器时需要记住的一些提示:

*   依赖浏览器的“查看源代码”功能，而不仅仅是页面 DOM，因为它可能已经发生了变化(在浏览器控制台选择可能会产生与 jsoup 不同的结果)
*   了解你的选择者，因为他们有很多，至少以前见过他们总是好的；掌握选择器需要时间
*   使用选择器的游戏场来试验它们(在那里粘贴一个样本 HTML)
*   减少对页面变化的依赖:以最小和最不妥协的选择器为目标(例如，首选 id。基于)

## 7。修改

修改包括设置元素的属性、文本和 HTML，以及追加和删除元素。它是对先前由 jsoup 生成的 DOM 树——`Document`完成的。

### 7.1。设置属性和内部文本/HTML

与在 jQuery 中一样，设置属性、文本和 HTML 的方法具有相同的名称，但也接收要设置的值:

*   `attr()`–设置属性的值(如果属性不存在，则创建属性)
*   `text()`–设置元素内部文本，替换内容
*   `html()`–设置 HTML 内部元素，替换内容

让我们来看看这些方法的一个简单例子:

```
timeElement.attr("datetime", "2016-12-16 15:19:54.3");
sectionDiv.text("foo bar");
firstArticle.select("h2").html("<div><span></span></div>"); 
```

### 7.2。创建和追加元素

要添加新元素，首先需要通过实例化`Element`来构建它。一旦`Element`被构建，您可以使用`appendChild`方法将它附加到另一个`Element`。新创建并追加的`Element`将被插入到调用`appendChild`的元素的末尾:

```
Element link = new Element(Tag.valueOf("a"), "")
  .text("Checkout this amazing website!")
  .attr("href", "http://baeldung.com")
  .attr("target", "_blank");
firstArticle.appendChild(link);
```

### 7.3。移除元件

要删除元素，您需要首先选择它们并运行`remove`方法。

例如，让我们从`Document,`中删除所有包含`navbar-link”`类的`<li>`标签，并删除第一篇文章中的所有图像:

```
doc.select("li.navbar-link").remove();
firstArticle.select("img").remove();
```

### 7.4。将修改后的文档转换为 HTML

最后，由于我们正在更改`Document`，我们可能想要检查我们的工作。

要做到这一点，我们可以通过使用给出的方法选择、遍历和提取来探索`Document` DOM 树，或者我们可以使用`html()`方法简单地将它的 HTML 提取为一个`String`:

```
String docHtml = doc.html();
```

`String`输出是一个整洁的 HTML。

## 8。结论

Jsoup 是一个很棒的库，可以抓取任何页面。如果您正在使用 Java，并且不需要基于浏览器的抓取，那么需要考虑一个库。它熟悉且易于使用，因为它利用了您可能拥有的前端开发知识，并遵循良好的实践和设计模式。

你可以通过研究 [jsoup API](https://web.archive.org/web/20221208143845/https://jsoup.org/apidocs/) 和阅读 [jsoup 食谱](https://web.archive.org/web/20221208143845/https://jsoup.org/cookbook/)来学习更多关于用 jsoup 抓取网页的知识。

本教程中使用的源代码可以在 [GitHub 项目](https://web.archive.org/web/20221208143845/https://github.com/eugenp/tutorials/tree/master/jsoup)中找到。