# 使用 Apache POI 合并 Excel 中的单元格

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-apache-poi-merge-cells>

## 1.概观

在本教程中，我们将展示如何用 Apache POI 合并 [Excel](/web/20220630140759/https://www.baeldung.com/java-microsoft-excel) 中的单元格。

## 2.Apache 然后

首先，我们首先需要将 [poi](https://web.archive.org/web/20220630140759/https://search.maven.org/search?q=g:org.apache.poi%20a:poi) 依赖项添加到我们的项目`pom.xml`文件中:

```java
<dependency>
    <groupId>org.apache.poi</groupId>
    <artifactId>poi</artifactId>
    <version>5.2.0</version>
</dependency>
```

Apache POI 使用 [`Workbook`](https://web.archive.org/web/20220630140759/https://poi.apache.org/apidocs/dev/org/apache/poi/ss/usermodel/Workbook.html) 接口来表示一个 Excel 文件。它还使用 [`Sheet`](https://web.archive.org/web/20220630140759/https://poi.apache.org/apidocs/dev/org/apache/poi/ss/usermodel/Sheet.html) 、 [`Row`](https://web.archive.org/web/20220630140759/https://poi.apache.org/apidocs/dev/org/apache/poi/ss/usermodel/Row.html) 、 [`Cell`](https://web.archive.org/web/20220630140759/https://poi.apache.org/apidocs/dev/org/apache/poi/ss/usermodel/Cell.html) 接口来对 Excel 文件中不同级别的元素进行建模。

## 3.合并单元

在 Excel 中，我们有时希望在两个或多个单元格中显示一个字符串。例如，我们可以水平合并几个单元格，以创建跨越几列的表格标题:

[![](img/bfbc1fc7810cadcb89dd821bd1c92df5.png)](/web/20220630140759/https://www.baeldung.com/wp-content/uploads/2020/01/merge.png)

**为了实现这一点，我们可以使用** `[addMergedRegion](https://web.archive.org/web/20220630140759/https://poi.apache.org/apidocs/dev/org/apache/poi/ss/usermodel/Sheet.html#addMergedRegion-org.apache.poi.ss.util.CellRangeAddress-)` **来合并几个由`[CellRangeAddress](https://web.archive.org/web/20220630140759/https://poi.apache.org/apidocs/dev/org/apache/poi/ss/util/CellRangeAddress.html)`定义的单元格。**设置单元格范围有两种方式。首先，我们可以使用四个从零开始的索引来定义左上角的单元格位置和右下角的单元格位置:

```java
sheet = // existing Sheet setup
int firstRow = 0;
int lastRow = 0;
int firstCol = 0;
int lastCol = 2
sheet.addMergedRegion(new CellRangeAddress(firstRow, lastRow, firstCol, lastCol));
```

我们还可以使用单元格区域引用字符串来提供合并区域:

```java
sheet = // existing Sheet setup
sheet.addMergedRegion(CellRangeAddress.valueOf("A1:C1"));
```

如果单元格在合并前有数据，Excel 将使用左上角的单元格值作为合并区域值。对于其他单元格，Excel 将丢弃它们的数据。

当我们在一个 Excel 文件上添加多个合并区域时，我们不应该创建任何重叠。否则，Apache POI 将在运行时抛出异常。

## 4.摘要

在这篇简短的文章中，我们展示了如何用 Apache POI 合并几个单元格。我们还讨论了定义合并区域的两种方法。

和往常一样，这篇文章的源代码可以在 GitHub 的[上找到。](https://web.archive.org/web/20220630140759/https://github.com/eugenp/tutorials/tree/master/apache-poi)