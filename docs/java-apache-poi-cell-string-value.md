# 使用 Apache POI 获取 Excel 单元格的字符串值

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-apache-poi-cell-string-value>

## 1.概观

一个 [Microsoft Excel](/web/20220626073851/https://www.baeldung.com/java-microsoft-excel) 单元格可以有不同的类型，如字符串、数字、布尔和公式。

在这个快速教程中，我们将展示如何使用 Apache POI 将单元格值作为字符串读取——不管单元格类型如何。

## 2.Apache 然后

首先，我们首先需要将 [poi](https://web.archive.org/web/20220626073851/https://search.maven.org/search?q=g:org.apache.poi%20a:poi) 依赖项添加到我们的项目`pom.xml`文件中:

```java
<dependency>
    <groupId>org.apache.poi</groupId>
    <artifactId>poi</artifactId>
    <version>5.2.0</version>
</dependency>
```

Apache POI 使用 [`Workbook`](https://web.archive.org/web/20220626073851/https://poi.apache.org/apidocs/dev/org/apache/poi/ss/usermodel/Workbook.html) 接口来表示一个 Excel 文件。它还使用 [`Sheet`](https://web.archive.org/web/20220626073851/https://poi.apache.org/apidocs/dev/org/apache/poi/ss/usermodel/Sheet.html) 、 [`Row`](https://web.archive.org/web/20220626073851/https://poi.apache.org/apidocs/dev/org/apache/poi/ss/usermodel/Row.html) 、 [`Cell`](https://web.archive.org/web/20220626073851/https://poi.apache.org/apidocs/dev/org/apache/poi/ss/usermodel/Cell.html) 接口对 Excel 文件中不同级别的元素进行建模。在`Cell`级别，我们可以使用它的 [`getCellType()`](https://web.archive.org/web/20220626073851/https://poi.apache.org/apidocs/dev/org/apache/poi/ss/usermodel/Cell.html#getCellType--) 方法得到[单元格类型](https://web.archive.org/web/20220626073851/https://poi.apache.org/apidocs/dev/org/apache/poi/ss/usermodel/CellType.html)。Apache POI 支持以下单元类型:

*   空白的
*   布尔型
*   错误
*   公式
*   数字的
*   线

如果我们想在屏幕上显示 Excel 文件的内容，我们希望得到一个单元格的字符串表示，而不是它的原始值。因此，**对于不属于 string 类型的单元格，我们需要将它们的数据转换成 STRING 值**。

## 3.获取单元格字符串值

**我们可以使用`[DataFormatter](https://web.archive.org/web/20220626073851/https://poi.apache.org/apidocs/dev/org/apache/poi/ss/usermodel/DataFormatter.html)` 来获取 Excel 单元格的字符串值。**它可以获取一个单元格中存储的值的格式化字符串表示。例如，如果一个单元格的数值是 1.234，并且该单元格的格式规则是两位小数点，我们将得到字符串表示“1.23”:

```java
Cell cell = // a numeric cell with value of 1.234 and format rule "0.00"

DataFormatter formatter = new DataFormatter();
String strValue = formatter.formatCellValue(cell);

assertEquals("1.23", strValue);
```

因此，`DataFormatter.formatCellValue()`的结果就是显示字符串，就像它在 Excel 中出现的一样。

## 4.获取公式单元格的字符串值

如果单元格的类型是公式，前面的方法将返回原始公式字符串，而不是计算出的公式值。因此，**要得到公式值的字符串表示，我们需要用`[FormulaEvaluator](https://web.archive.org/web/20220626073851/https://poi.apache.org/apidocs/dev/org/apache/poi/ss/usermodel/FormulaEvaluator.html)`对公式**求值:

```java
Workbook workbook = // existing Workbook setup
FormulaEvaluator evaluator = workbook.getCreationHelper().createFormulaEvaluator();

Cell cell = // a formula cell with value of "SUM(1,2)"

DataFormatter formatter = new DataFormatter();
String strValue = formatter.formatCellValue(cell, evaluator);

assertEquals("3", strValue);
```

这种方法适用于所有细胞类型。如果单元格类型是公式，我们将使用给定的`FormulaEvaluator`对其求值。否则，我们将返回不带任何求值的字符串表示。

## 5.摘要

在这篇简短的文章中，我们展示了如何获取 Excel 单元格的字符串表示，而不管它是什么类型。和往常一样，这篇文章的源代码可以在 GitHub 的[上找到。](https://web.archive.org/web/20220626073851/https://github.com/eugenp/tutorials/tree/master/apache-poi)