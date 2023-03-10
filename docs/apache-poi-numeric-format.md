# 使用 POI 的数字格式

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/apache-poi-numeric-format>

## 1.概观

在这个快速教程中，**我们将演示如何使用 Apache POI** 在 Excel 中格式化数字单元格。

## 2.Apache 然后

[Apache POI](https://web.archive.org/web/20220625174656/https://poi.apache.org/) 是一个开源的纯 Java 项目。它提供了读写微软 Office 格式文件的库，比如 Word、PowerPoint 和 [Excel](/web/20220625174656/https://www.baeldung.com/java-microsoft-excel "Microsoft Excel") 。

在处理新的`.xlsx`文件格式时，我们将使用`XSSFWorkbook`类，对于 `the .xls`格式，我们使用`HSSFWorkbook `类`.`

## 3.数字格式

Apache POI 的`setCellValue`方法只接受`double`作为输入，或者可以隐式转换为`double`的输入，并返回一个`double`作为数值。 **`setCellStyle`方法用于添加所需的样式**。Excel 数字格式中的#字符表示在需要时在这里放置一个数字，而字符 0 表示总是在这里放置一个数字，即使它是一个不必要的 0。

### 3.1.仅用于显示值的格式

让我们使用像 0.00 或#这样的模式来格式化一个`double`值。##.首先，我们将创建一个简单的实用方法来格式化单元格值:

```java
public static void applyNumericFormat(Workbook outWorkbook, Row row, Cell cell, Double value, String styleFormat) {
    CellStyle style = outWorkbook.createCellStyle();
    DataFormat format = outWorkbook.createDataFormat();
    style.setDataFormat(format.getFormat(styleFormat));
    cell.setCellValue(value);
    cell.setCellStyle(style);
}
```

让我们验证一个简单的代码来验证上述方法:

```java
File file = new File("number_test.xlsx");
try (Workbook outWorkbook = new XSSFWorkbook()) {
    Sheet sheet = outWorkbook.createSheet("Numeric Sheet");
    Row row = sheet.createRow(0);
    Cell cell = row.createCell(0);
    ExcelNumericFormat.applyNumericFormat(outWorkbook, row, cell, 10.251, "0.00");
    FileOutputStream fileOut = new FileOutputStream(file);
    outWorkbook.write(fileOut);
    fileOut.close();
}
```

这将在电子表格中添加数字单元格:

[![](img/3e2dad7e5f51474d2e6dd0dedf47b622.png)](/web/20220625174656/https://www.baeldung.com/wp-content/uploads/2021/12/RoundedValue.png)

注意:**显示值为格式化值，实际值保持不变**。如果我们尝试访问同一个单元，我们仍然会得到 10.251。

让我们验证实际值:

```java
try (Workbook inWorkbook = new XSSFWorkbook("number_test.xlsx")) {
    Sheet sheet = inWorkbook.cloneSheet(0);
    Row row = sheet.getRow(0);
    Assertions.assertEquals(10.251, row.getCell(0).getNumericCellValue());
}
```

### 3.2.实际值和显示值的格式

让我们使用以下模式格式化显示值和实际值:

```java
File file = new File("number_test.xlsx");
try (Workbook outWorkbook = new HSSFWorkbook()) {
    Sheet sheet = outWorkbook.createSheet("Numeric Sheet");
    Row row = sheet.createRow(0);
    Cell cell = row.createCell(0);
    DecimalFormat df = new DecimalFormat("#,###.##");
    ExcelNumericFormat.applyNumericFormat(outWorkbook, row, cell, Double.valueOf(df.format(10.251)), "#,###.##");
    FileOutputStream fileOut = new FileOutputStream(file);
    outWorkbook.write(fileOut);
    fileOut.close();
}
```

这将在电子表格中添加数字单元格并显示格式化的值，并且**还会更改实际值**:

[![](img/1308e8ca013acfabd303d6e10e6fd9e8.png)](/web/20220625174656/https://www.baeldung.com/wp-content/uploads/2021/12/excel-numeric.png)

让我们断言上述情况中的实际值:

```java
try (Workbook inWorkbook = new XSSFWorkbook("number_test.xlsx")) {
    Sheet sheet = inWorkbook.cloneSheet(0);
    Row row = sheet.getRow(0);
    Assertions.assertEquals(10.25, row.getCell(0).getNumericCellValue());
}
```

## 4.结论

在本文中，我们演示了如何在改变或不改变 Excel 表格中数值单元格的实际值的情况下显示格式化值。

本教程中使用的所有代码示例都可以从 GitHub 上的[处获得。](https://web.archive.org/web/20220625174656/https://github.com/eugenp/tutorials/tree/master/apache-poi-2)