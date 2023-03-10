# 读取 Excel 单元格值，而不是使用 Apache POI 的公式

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/apache-poi-read-cell-value-formula>

## 1.介绍

在 Java 中读取 Excel 文件时，我们通常希望读取单元格的值来执行一些计算或生成报告。但是，我们可能会遇到一个或多个包含公式而不是原始数据值的单元格。那么，我们如何获得这些单元格的实际数据值呢？

在本教程中，我们将研究使用 [Apache POI](/web/20220628160923/https://www.baeldung.com/java-microsoft-excel) Java 库读取 Excel 单元格值的不同方法，而不是计算单元格值的公式。

有两种方法可以解决这个问题:

*   获取单元格的最后一个缓存值
*   在运行时评估公式以获取单元格值

## 2.Maven 依赖性

我们需要在 Apache POI 的 pom.xml 文件中添加以下依赖项:

```java
<dependency>
    <groupId>org.apache.poi</groupId>
    <artifactId>poi-ooxml</artifactId>
    <version>5.2.0</version>
</dependency>
```

最新版本的 poi-ooxml 可以从 Maven Central 下载。

## 3.获取上一次缓存的值

当公式计算值时，Excel 为单元格存储两个对象。一是公式本身，二是缓存的值。**缓存值包含由公式**计算的最后一个值。

因此，这里的想法是，我们可以获取最后缓存的值，并将其视为单元格值。最后缓存的值不一定是正确的单元格值。但是，当我们处理一个保存的 Excel 文件时，并且最近没有对该文件进行修改，那么最后缓存的值应该是单元格值。

让我们看看如何获取单元格的最后一个缓存值:

```java
FileInputStream inputStream = new FileInputStream(new File("temp.xlsx"));
Workbook workbook = new XSSFWorkbook(inputStream);
Sheet sheet = workbook.getSheetAt(0);

CellAddress cellAddress = new CellAddress("C2");
Row row = sheet.getRow(cellAddress.getRow());
Cell cell = row.getCell(cellAddress.getColumn());

if (cell.getCellType() == CellType.FORMULA) {
    switch (cell.getCachedFormulaResultType()) {
        case BOOLEAN:
            System.out.println(cell.getBooleanCellValue());
            break;
        case NUMERIC:
            System.out.println(cell.getNumericCellValue());
            break;
        case STRING:
            System.out.println(cell.getRichStringCellValue());
            break;
    }
}
```

## 4.评估公式以获取单元格值

Apache POI 提供了一个 **`FormulaEvaluator `类，使我们能够计算 Excel 表格中公式**的结果。

所以，我们可以使用`FormulaEvaluator `在运行时直接计算单元格值。`FormulaEvaluator `类提供了一个名为`evaluateFormulaCell,`的方法，该方法评估给定`Cell` 对象的单元格值，并返回一个`CellType `对象，该对象表示单元格值的数据类型。

让我们来看看这种方法的实际应用:

```java
// existing Workbook setup

FormulaEvaluator evaluator = workbook.getCreationHelper().createFormulaEvaluator(); 

// existing Sheet, Row, and Cell setup

if (cell.getCellType() == CellType.FORMULA) {
    switch (evaluator.evaluateFormulaCell(cell)) {
        case BOOLEAN:
            System.out.println(cell.getBooleanCellValue());
            break;
        case NUMERIC:
            System.out.println(cell.getNumericCellValue());
            break;
        case STRING:
            System.out.println(cell.getStringCellValue());
            break;
    }
} 
```

## 5.选择哪种方法

这两种方法的简单区别在于，第一种方法使用最后缓存的值，第二种方法在运行时计算公式。

如果我们正在处理一个已经保存的 Excel 文件，并且我们不打算在运行时对该电子表格进行更改，那么缓存值方法更好，因为我们不必计算公式。

然而，如果我们知道我们将在运行时进行频繁的更改，那么最好在运行时评估公式以获取单元格值。

## 6.结论

在这篇简短的文章中，我们看到了获取 Excel 单元格的值的两种方法，而不是计算它的公式。

本文的完整源代码可以在 GitHub 上的  [找到。](https://web.archive.org/web/20220628160923/https://github.com/eugenp/tutorials/tree/master/apache-poi)