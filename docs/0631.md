# 使用 Apache POI 在 Excel 中设置公式

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-apache-poi-set-formulas>

## 1.概观

在这个快速教程中，我们将通过一个简单的例子来讨论如何使用 **[Apache POI](https://web.archive.org/web/20221128040353/https://poi.apache.org/)** 在 Microsoft Excel 电子表格中**设置公式。**

## 2.Apache 然后

Apache POI 是一个流行的开源 Java 库，它为程序员提供 API 来**创建、修改和显示 MS Office** 文件。

它用 [`Workbook`](https://web.archive.org/web/20221128040353/https://poi.apache.org/apidocs/dev/org/apache/poi/ss/usermodel/Workbook.html) 来表示一个 Excel 文件及其元素。Excel 文件中的 [`Cell`](https://web.archive.org/web/20221128040353/https://poi.apache.org/apidocs/dev/org/apache/poi/ss/usermodel/Cell.html) 可以有不同的类型，如`FORMULA`。

为了查看 Apache POI 的运行情况，我们将设置一个公式来减去一个 [Excel](https://web.archive.org/web/20221128040353/https://github.com/eugenp/tutorials/blob/master/apache-poi-2/src/main/resources/com/baeldung/poi/excel/setformula/SetFormulaTest.xlsx) 文件中 A 列和 B 列的值之和。链接文件包含以下数据:

[![Sample Excel File](img/d7017d0d19bb7149ed57647613f41f63.png)](/web/20221128040353/https://www.baeldung.com/wp-content/uploads/2020/07/Sample-Excel-File-1-300x223-1-1.png)

## 3.属国

首先，我们需要将 POI 依赖项添加到我们的项目`pom.xml`文件中。**要使用 Excel 2007+工作簿，我们应该使用`[poi-ooxml](https://web.archive.org/web/20221128040353/https://search.maven.org/search?q=g:org.apache.poi%20a:poi-ooxml)`** :

```
<dependency>
    <groupId>org.apache.poi</groupId>
    <artifactId>poi-ooxml</artifactId>
    <version>5.2.0</version>
</dependency>
```

**注意，对于早期版本的 Excel，我们应该使用`[poi](https://web.archive.org/web/20221128040353/https://search.maven.org/search?q=g:org.apache.poi%20a:poi)`依赖项。**

## 4.单元格查找

首先，让我们打开文件并构建适当的工作簿:

```
FileInputStream inputStream = new FileInputStream(new File(fileLocation));
XSSFWorkbook excel = new XSSFWorkbook(inputStream);
```

然后，**我们需要创建或查找我们将要使用的单元格**。使用之前共享的数据，我们想要编辑单元 C1。

这是在第一张纸上的第一行，我们可以要求 POI 提供第一个空白列:

```
XSSFSheet sheet = excel.getSheetAt(0);
int lastCellNum = sheet.getRow(0).getLastCellNum();
XSSFCell formulaCell = sheet.getRow(0).createCell(lastCellNum + 1);
```

## 5.公式

接下来，我们要在已经查找过的单元格上设置一个公式。

如前所述，让我们从 a 列的总和中减去 B 列的总和。在 Excel 中，这将是:

```
=SUM(A:A)-SUM(B:B)
```

我们可以用`setCellFormula`方法将它写入我们的`formulaCell`:

```
formulaCell.setCellFormula("SUM(A:A)-SUM(B:B)");
```

**现在，这个就不评价公式了。**为此，我们需要使用 POI 的`XSSFFormulaEvaluator`:

```
XSSFFormulaEvaluator formulaEvaluator = 
  excel.getCreationHelper().createFormulaEvaluator();
formulaEvaluator.evaluateFormulaCell(formulaCell);
```

结果将被设置在下一个空列的第一个`Cell `:[![Sample Excel File2](img/8b3a5795eba20bf459b0ab238f9eaa05.png)](/web/20221128040353/https://www.baeldung.com/wp-content/uploads/2020/07/Sample-Excel-File2-1.png)

如我们所见，计算结果保存在 c 列的第一个单元格中。公式也显示在公式栏中。

注意，`[FormulaEvaluator](https://web.archive.org/web/20221128040353/https://poi.apache.org/apidocs/dev/org/apache/poi/ss/usermodel/FormulaEvaluator.html)`类为我们提供了在 Excel 工作簿中评估`FORMULA`的其他方法，比如`evaluateAll`，它将遍历所有单元格并评估它们。

## 6.结论

在本教程中，我们展示了如何使用 Apache POI API 在 Java 中的 Excel 文件的单元格上设置公式。

这篇文章的完整源代码可以在 GitHub 的[上找到。](https://web.archive.org/web/20221128040353/https://github.com/eugenp/tutorials/tree/master/apache-poi-2)