# 使用 Jsoup 时保留换行符

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/jsoup-line-breaks>

## 1.概观

在本教程中，我们将简要地看一下使用 [Jsoup](/web/20220926191721/https://www.baeldung.com/java-with-jsoup) 将 HTML 解析为纯文本时**保留换行符的不同方式。我们将介绍如何保留与换行符(`\n`)相关的换行符，以及与`<br>`和`<p>`标签相关的换行符。**

## 2.解析 HTML 文本时保留`\n`

**Jsoup 默认从 HTML 文本**中删除换行符(`\n`)，并用空格字符替换每个换行符。

但是，为了防止 Jsoup 删除换行符，我们可以更改 Jsoup 的 [`OutputSetting`](https://web.archive.org/web/20220926191721/https://jsoup.org/apidocs/org/jsoup/nodes/Document.OutputSettings.html#prettyPrint(boolean)) 并禁用 pretty-print。**如果禁用了漂亮打印，HTML 输出方法将不会重新格式化输出，输出将类似于输入:**

```java
Document.OutputSettings outputSettings = new Document.OutputSettings();
outputSettings.prettyPrint(false);
```

此外，我们可以使用 [`Jsoup` # `clean`](https://web.archive.org/web/20220926191721/https://jsoup.org/apidocs/org/jsoup/Jsoup.html#clean(java.lang.String,java.lang.String,org.jsoup.safety.Whitelist,org.jsoup.nodes.Document.OutputSettings)) 从字符串中删除所有的 HTML 标签:

```java
String strHTML = "<html><body>Hello\nworld</body></html>";
String strWithNewLines = Jsoup.clean(strHTML, "", Whitelist.none(), outputSettings);
```

让我们看看我们的输出字符串`strWithNewLines`是什么样子的:

```java
assertEquals("Hello\nworld", strWithNewLines);
```

因此，我们可以看到，通过用`Whitelist` # `none`调用`Jsoup` # `clean`并禁用 Jsoup 的漂亮打印输出设置，我们能够保留与换行符相关联的换行符。

让我们看看我们还能做什么！

## 3.保留与`<br>`和`<p>`标签相关联的换行符

**在使用`Jsoup` # `clean`方法清理 HTML 文本时，它会删除由`<br>`和`<p>`** 等 HTML 标签创建的换行符。

为了保留与这些标签相关联的换行符，我们首先需要从我们的 HTML 字符串创建一个 [Jsoup `Document`](https://web.archive.org/web/20220926191721/https://jsoup.org/apidocs/org/jsoup/nodes/Document.html) :

```java
String strHTML = "<html><body>Hello<br>World<p>Paragraph</p></body></html>";
Document jsoupDoc = Jsoup.parse(strHTML);
```

接下来，我们在`<br>`和`<p>`标签前添加一个换行符——同样，我们也禁用了漂亮打印输出设置:

```java
Document.OutputSettings outputSettings = new Document.OutputSettings();
outputSettings.prettyPrint(false);
jsoupDoc.outputSettings(outputSettings);
jsoupDoc.select("br").before("\\n");
jsoupDoc.select("p").before("\\n");
```

这里，我们使用了 Jsoup `Document`的 [`select`](/web/20220926191721/https://www.baeldung.com/java-with-jsoup#1-selecting) 方法和`before`方法来添加换行符。

之后，我们从`jsoupDoc`获取 HTML 字符串，保留原来的新行:

```java
String str = jsoupDoc.html().replaceAll("\\\\n", "\n");
```

最后，我们用`Whitelist` # `none`调用`Jsoup` # `clean`，并且禁用漂亮打印输出设置:

```java
String strWithNewLines = Jsoup.clean(str, "", Whitelist.none(), outputSettings);
```

我们的输出字符串`strWithNewLines`看起来像:

```java
assertEquals("Hello\nWorld\nParagraph", strWithNewLines);
```

**因此，通过在`<br>`和`<p>` HTML 标签前添加换行符，并禁用 Jsoup 的漂亮打印输出设置，我们可以保留与它们相关的换行符。**

## 4.结论

在这篇短文中，我们学习了在用 Jsoup 将 HTML 解析成纯文本时，如何保留与换行符(`\n`)以及`<br>`和`<p>`标签相关的换行符。

和往常一样，所有这些代码示例都可以在 GitHub 上的[处获得。](https://web.archive.org/web/20220926191721/https://github.com/eugenp/tutorials/tree/master/jsoup)