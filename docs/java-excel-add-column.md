# 使用 Apache POI 向 Excel 表添加列

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-excel-add-column>

## 1.概观

在本教程中，我们将展示如何使用 Apache POI 向一个 [Excel](/web/20220625234314/https://www.baeldung.com/java-microsoft-excel) 文件中的工作表添加一列。

## 2.Apache 然后

首先，我们首先需要将 [poi-ooxml](https://web.archive.org/web/20220625234314/https://search.maven.org/search?q=g:org.apache.poi%20a:poi) 依赖项添加到我们项目的`pom.xml`文件中:

```
<dependency>
    <groupId>org.apache.poi</groupId>
    <artifactId>poi-ooxml</artifactId>
    <version>5.0.0</version>
</dependency>
```

Apache POI 使用 [`Workbook`](https://web.archive.org/web/20220625234314/https://poi.apache.org/apidocs/dev/org/apache/poi/ss/usermodel/Workbook.html) 接口来表示一个 Excel 文件。它还使用 [`Sheet`](https://web.archive.org/web/20220625234314/https://poi.apache.org/apidocs/dev/org/apache/poi/ss/usermodel/Sheet.html) 、 [`Row`](https://web.archive.org/web/20220625234314/https://poi.apache.org/apidocs/dev/org/apache/poi/ss/usermodel/Row.html) 、 [`Cell`](https://web.archive.org/web/20220625234314/https://poi.apache.org/apidocs/dev/org/apache/poi/ss/usermodel/Cell.html) 接口对 Excel 文件中的不同元素进行建模。

## 3.添加新列

在 Excel 中，我们有时希望在现有行上添加一个新列。**为此，我们可以遍历每一行，并在行尾创建一个新的单元格**:

```
void addColumn(Sheet sheet, CellType cellType) {
    for (Row currentRow : sheet) {
        currentRow.createCell(currentRow.getLastCellNum(), cellType);
    }
}
```

在这个方法中，我们使用一个循环遍历输入 Excel `sheet`的所有行。对于每一行，我们首先找到它的最后一个单元格编号，然后在最后一个单元格之后创建一个新的单元格。

## 4.摘要

在这篇简短的文章中，我们展示了如何使用 Apache POI 添加新列。和往常一样，这篇文章的源代码可以在 GitHub 的[上找到。](https://web.archive.org/web/20220625234314/https://github.com/eugenp/tutorials/tree/master/apache-poi-2)