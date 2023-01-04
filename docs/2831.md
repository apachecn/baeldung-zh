# 使用 Java 移除 HTML 标签

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-remove-html-tags>

## 1.概观

有时，我们希望删除所有的 HTML 标签，并从 HTML 文档字符串中提取文本。

这个问题看起来很简单。然而，根据不同的需求，它可以有不同的变体。

在本教程中，我们将讨论如何使用 Java 来做到这一点。

## 2.使用正则表达式

因为我们已经得到了 HTML 作为一个`String`变量，我们需要做一种文本操作。

当面对文本操作问题时，[正则表达式](/web/20220907235807/https://www.baeldung.com/regular-expressions-java) (Regex)可能是第一个想到的。

从字符串中移除 HTML 标签对 Regex 来说并不是一个挑战，因为无论是开始还是结束的 HTML 元素，它们都遵循“< … >”模式。

如果我们把它翻译成 Regex，那就是`“<[^>]*>”`或者`“<.*?>”`。

我们应该注意到 **Regex 默认做贪婪匹配**。也就是说，正则表达式`“<.*>”`对我们的问题不起作用，因为我们想匹配从'`<`'到下一个'`>`'，而不是一行中的最后一个'`>`'。

现在，让我们测试它是否能从 HTML 源代码中移除标签。

### 2.1.从`example1.html`中移除标签

在我们测试移除 HTML 标签之前，首先让我们创建一个 HTML 示例，比如说`example1.html`:

```
<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN"
        "http://www.w3.org/TR/html4/loose.dtd">
<html>
<head>
    <title>This is the page title</title>
</head>
<body>
    <p>
        If the application X doesn't start, the possible causes could be:<br/>
        1\. <a href="maven.com">Maven</a> is not installed.<br/>
        2\. Not enough disk space.<br/>
        3\. Not enough memory.
    </p>
</body>
</html> 
```

现在，让我们编写一个测试并使用`[String.replaceAll()](/web/20220907235807/https://www.baeldung.com/string/replace-all)`来删除 HTML 标签:

```
String html = ... // load example1.html
String result = html.replaceAll("<[^>]*>", "");
System.out.println(result); 
```

如果我们运行测试方法，我们会看到结果:

```
 This is the page title

        If the application X doesn't start, the possible causes could be:
        1\. Maven is not installed.
        2\. Not enough disk space.
        3\. Not enough memory. 
```

输出看起来相当不错。这是因为所有的 HTML 标签都被删除了。

它保留了剥离的 HTML 中的空白。但是当我们处理提取的文本时，我们可以很容易地删除或跳过那些空行或空白。到目前为止，一切顺利。

### 2.2.从`example2.html`中移除标签

正如我们刚刚看到的，使用 Regex 删除 HTML 标签非常简单。然而，**这种方法可能有问题，因为我们无法预测我们将得到什么样的 HTML 源**。

例如，一个 HTML 文档可能有`<script>`或`<style>`标签，我们可能不希望在结果中包含它们的内容。

此外，`<script>`、`<style>`、甚至`<body>`标签中的文本可以包含“`<`或“`>`”字符。如果是这种情况，我们的正则表达式方法可能会失败。

现在，让我们看另一个 HTML 例子，比如说`example2.html`:

```
<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN"
        "http://www.w3.org/TR/html4/loose.dtd">
<html>
<head>
    <title>This is the page title</title>
</head>
<script>
    // some interesting script functions
</script>
<body>
    <p>
        If the application X doesn't start, the possible causes could be:<br/>
        1\. <a
            id="link"
            href="http://maven.apache.org/">
            Maven
            </a> is not installed.<br/>
        2\. Not enough (<1G) disk space.<br/>
        3\. Not enough (<64MB) memory.<br/>
    </p>
</body>
</html> 
```

这一次，我们有了一个`<script>`标签和`<body>`标签中的`<`字符。

如果我们在`example2.html`上使用相同的方法，我们将得到(空行已被删除):

```
 This is the page title
    // some interesting script functions    
        If the application X doesn't start, the possible causes could be:
        1\. 
            Maven
             is not installed.
        2\. Not enough (
        3\. Not enough (
```

显然，由于“

所以，**用 Regex 处理 XML 或者 HTML 是脆弱的**。相反，我们可以选择一个 HTML 解析器来完成这项工作。

接下来，我们将介绍几个易于使用的 HTML 库来提取文本。

## 3.使用 Jsoup

Jsoup 是一个流行的 HTML 解析器。要从 HTML 文档中提取文本，我们可以简单地调用`Jsoup.parse(htmlString).text()`。

首先，我们需要将 [Jsoup 库](https://web.archive.org/web/20220907235807/https://search.maven.org/search?q=a:jsoup)添加到类路径中。例如，假设我们正在使用 [Maven](/web/20220907235807/https://www.baeldung.com/maven) 来管理项目依赖性:

```
<dependency>
    <groupId>org.jsoup</groupId>
    <artifactId>jsoup</artifactId>
    <version>1.14.3</version>
</dependency> 
```

现在，让我们用我们的`example2.html`测试一下:

```
String html = ... // load example2.html
System.out.println(Jsoup.parse(html).text()); 
```

如果我们运行这个方法，它会打印:

```
This is the page title If the application X doesn't start, the possible causes could be: 1\. Maven is not installed. 2\. Not enough (<1G) disk space. 3\. Not enough (<64MB) memory. 
```

如输出所示，Jsoup 已经成功地从 HTML 文档中提取了文本。另外，`<script>`元素中的文本也被忽略了。

另外，**默认情况下，Jsoup 会删除所有的文本格式和空白，比如换行符**。

但是，如果需要的话，我们也可以要求 Jsoup[保留换行](/web/20220907235807/https://www.baeldung.com/jsoup-line-breaks)。

## 4.使用 html 清洁器

HTMLCleaner 是另一个 HTML 解析器。它的目标是使来自网络的“格式不良和肮脏的”HTML 适合于进一步处理。

首先，让我们在我们的`pom.xml`中添加 [HTMLCleaner 依赖项](https://web.archive.org/web/20220907235807/https://search.maven.org/search?q=a:htmlcleaner%20g:net.sourceforge.htmlcleaner):

```
<dependency>
    <groupId>net.sourceforge.htmlcleaner</groupId>
    <artifactId>htmlcleaner</artifactId>
    <version>2.25</version>
</dependency> 
```

我们可以设置[各种选项](https://web.archive.org/web/20220907235807/http://htmlcleaner.sourceforge.net/parameters.php)来控制 HTMLCleaner 的解析行为。

这里，作为一个例子，让我们告诉 HTMLCleaner 在解析`example2.html`时跳过`<script>`元素:

```
String html = ... // load example2.html
CleanerProperties props = new CleanerProperties();
props.setPruneTags("script");
String result = new HtmlCleaner(props).clean(html).getText().toString();
System.out.println(result); 
```

如果我们运行测试，HTMLCleaner 将产生以下输出:

```
 This is the page title

        If the application X doesn't start, the possible causes could be:
        1\. 
            Maven
             is not installed.
        2\. Not enough (<1G) disk space.
        3\. Not enough (<64MB) memory. 
```

正如我们所见，`<script>`元素中的内容被忽略了。

另外，**将`<br/>`标签转换成提取文本**中的换行符。如果格式很重要，这会很有帮助。

另一方面， **HTMLCleaner 从剥离的 HTML 源**中保留空白。例如，文本“`1\. Maven is not installed`”被分成三行。

## 5.利用杰里科

最后，我们将看到另一个 HTML 解析器—[Jericho](https://web.archive.org/web/20220907235807/http://jericho.htmlparser.net/)。它有一个很好的特性:用简单的文本格式呈现 HTML 标记。稍后我们将看到它的实际应用。

像往常一样，我们先在`pom.xml`中添加[杰里科属地](https://web.archive.org/web/20220907235807/https://search.maven.org/search?q=a:jericho):

```
<dependency>
    <groupId>net.htmlparser.jericho</groupId>
    <artifactId>jericho-html</artifactId>
    <version>3.4</version>
</dependency> 
```

在我们的`example2.html`中，我们有一个超链接“`Maven (http://maven.apache.org/)`”。现在，假设我们希望在结果中同时包含链接 URL 和链接文本。

为此，我们可以创建一个`Renderer`对象并使用`includeHyperlinkURLs`选项:

```
String html = ... // load example2.html
Source htmlSource = new Source(html);
Segment segment = new Segment(htmlSource, 0, htmlSource.length());
Renderer htmlRender = new Renderer(segment).setIncludeHyperlinkURLs(true);
System.out.println(htmlRender); 
```

接下来，让我们执行测试并检查输出:

```
If the application X doesn't start, the possible causes could be:
1\. Maven <http://maven.apache.org/> is not installed.
2\. Not enough (<1G) disk space.
3\. Not enough (<64MB) memory.
```

正如我们在上面的结果中看到的，文本已经被格式化了。另外，**`<title>`元素中的文本默认被忽略**。

链接 URL 也包括在内。除了渲染链接(`<a>`)， **Jericho 支持[渲染其他 HTML 标签](https://web.archive.org/web/20220907235807/http://jericho.htmlparser.net/docs/javadoc/net/htmlparser/jericho/Renderer.html)，例如`<hr/>, <br/>,`子弹列表(`<ul>`和`<li>`)，等等**。

## 6.结论

在本文中，我们讨论了移除 HTML 标签和提取 HTML 文本的不同方法。

我们应该注意到**使用 Regex 处理 XML/HTML** 并不是一个好的做法。

和往常一样，本文的完整源代码可以在 GitHub 上找到[。](https://web.archive.org/web/20220907235807/https://github.com/eugenp/tutorials/tree/master/xml)