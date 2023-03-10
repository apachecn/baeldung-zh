# 用 Asciidoctor 生成图书

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/asciidoctor-book>

## 1。简介

在这篇简短的文章中，我们将演示如何从 AsciiDoc 文档生成一本书，以及如何使用各种样式选项定制您的书。

如果你不熟悉 Java 中的 AsciiDoc，可以看看我们的[对 asciidor](/web/20220626080553/https://www.baeldung.com/asciidoctor)的介绍。

## 2。后端图书类型

用 AsciiDoctorj 生成一本书的最简单的方法是用 Maven，就像前面提到的文章一样。**唯一的区别是你必须指定`doctype`标签并将其设置为`“book”.`**

```java
<backend>pdf</backend>
<doctype>book</doctype>
```

通过定义 doctype，AsciiDoctorj 知道您想要构建一本书，所以它创建:

*   标题页
*   目录
*   正文内容的第一页
*   部分和章节

为了得到提到的部分，Asciidoc 文档应该有定义的标题、章节和其他部分，这对于一本书来说是正常的。

## 3。 **定义自定义样式**

在写一本书的时候，我们很自然地想要使用一些自定义的样式。用简单的 YAML 文件中定义的特定于 AsciiDoc 的格式化语言可以做到这一点。

例如，这段代码将定义一本书中每一页的外观。我们希望采用纵向模式，在 A4 纸格式上，顶部和底部的边距为 0.75 英寸，两侧的边距为 1 英寸:

```java
page:
    layout: portrait
    margin: [0.75in, 1in, 0.75in, 1in]
    size: A4
```

此外，我们可以为图书的页脚和页眉定义自定义样式:

```java
header:
  height: 0.5in
  line_height: 1
  recto_content:
    center: '{document-title}'
  verso_content:
    center: '{document-title}'

footer:
  height: 0.5in
  line_height: 1
  recto_content:
    right: '{chapter-title} | *{page-number}*'
  verso_content:
    left: '*{page-number}* | {chapter-title}
```

更多格式选项可以在 AsciiDoctorj 的 [Github 页面上找到。](https://web.archive.org/web/20220626080553/https://github.com/asciidoctor/asciidoctor-pdf/blob/master/docs/theming-guide.adoc)

要在图书生成过程中包含自定义主题，我们必须定义样式文件所在的路径。该位置在`pom.xml:`的属性部分指定

```java
<pdf-stylesdir>${project.basedir}/src/themes</pdf-stylesdir>
<pdf-style>custom</pdf-style>
```

第一行定义了定义样式的路径，第二行指定了不带扩展名的文件名。

有了这些变化，我们的`pom.xml`看起来是这样的:

```java
<configuration>
    <sourceDirectory>src/docs/asciidoc</sourceDirectory>
    <outputDirectory>target/docs/asciidoc</outputDirectory>
    <attributes>
        <pdf-stylesdir>${project.basedir}/src/themes</pdf-stylesdir>
        <pdf-style>custom</pdf-style>
    </attributes>
    <backend>pdf</backend>
    <doctype>book</doctype>
</configuration>
```

## 4.生成图书

要生成你的书，只需要在项目目录下运行 Maven，生成的书可以在`target/docs/asciidoctor/`目录下找到。

## 5。结论

在本教程中，我们向您展示了如何使用 Maven 生成简单风格的书籍。

和往常一样，这篇文章的代码可以在 GitHub 上找到[。](https://web.archive.org/web/20220626080553/https://github.com/eugenp/tutorials/tree/master/asciidoctor)