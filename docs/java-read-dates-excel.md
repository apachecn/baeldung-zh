# 用 Java 从 Excel 中读取值

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-read-dates-excel>

## 1。概述

当涉及到 Microsoft Excel 文件时，从不同的单元格中读取值可能有点棘手。`Excel`文件是按行和单元格组织的电子表格，可以包含`String, Numeric, Date, Boolean, and even Formula`值。 [Apache POI](/web/20220627183230/https://www.baeldung.com/java-microsoft-excel) 是一个库，为**提供一整套工具来处理不同的 excel 文件和值类型**。

在本教程中，我们将重点学习如何处理 excel 文件，遍历行和单元格，并使用正确的方式读取每个单元格值类型。

## 2.Maven 依赖性

让我们从将 Apache POI 依赖项添加到`pom.xml`开始:

```java
<dependency>
    <groupId>org.apache.poi</groupId>
    <artifactId>poi-ooxml</artifactId>
    <version>5.2.0</version>
</dependency>
```

最新版本的`[poi-ooxml](https://web.archive.org/web/20220627183230/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22org.apache.poi%22%20AND%20a%3A%22poi-ooxml%22)`可以在 Maven Central 找到。

## 3.Apache POI 概述

层次结构从工作簿开始，它代表整个 Excel 文件。每个文件可以包含一个或多个工作表，这些工作表是行和单元格的集合。根据 excel 文件的版本**，HSSF 是代表旧 Excel 文件(`.xls`)的类的前缀，而 XSSF 用于最新版本(`.xlsx`)。**因此我们有:

*   `XSSFWorkbook` 和`HSSFWorkbook` 类代表 Excel 工作簿
*   `Sheet`界面代表 Excel 工作表
*   `Row` 接口表示行
*   `Cell` 界面代表单元格

### 3.1.处理 Excel 文件

首先，我们打开我们想要读取的文件，并将其转换成一个`FileInputStream`以供进一步处理。`FileInputStream`构造函数抛出了一个`java.io.FileNotFoundException`,所以我们需要将它放在一个 try-catch 块周围，并在最后关闭流:

```java
public static void readExcel(String filePath) {
    File file = new File(filePath);
    try {
        FileInputStream inputStream = new FileInputStream(file);
        ...
        inputStream.close();
    } catch (IOException e) {
        e.printStackTrace();
    }
} 
```

### 3.2.遍历 Excel 文件

在我们成功打开`InputStream`之后，是时候创建`XSSFWorkbook `并遍历每个工作表的行和单元格了。如果我们知道确切的张数或某一张的名称，我们可以分别使用`XSSFWorkbook, `的`getSheetAt(int index)`和`getSheet(String sheetName)`方法。

因为我们想要读取任何类型的 Excel 文件，我们将使用**三个嵌套的 for 循环遍历所有的工作表，一个用于工作表，一个用于每个工作表的行，最后一个用于每个工作表的单元格**。

出于本教程的考虑，我们将只把数据打印到控制台:

```java
FileInputStream inputStream = new FileInputStream(file);
Workbook baeuldungWorkBook = new XSSFWorkbook(inputStream);
for (Sheet sheet : baeuldungWorkBook) {
...
} 
```

然后，为了遍历工作表中的行，我们需要找到从 sheet 对象获得的第一行和最后一行的索引:

```java
int firstRow = sheet.getFirstRowNum();
int lastRow = sheet.getLastRowNum();
for (int index = firstRow + 1; index <= lastRow; index++) {
    Row row = sheet.getRow(index);
}
```

最后，我们对细胞做同样的事情。此外，在访问每个单元格时，我们可以选择传递一个`MissingCellPolicy`，它基本上告诉 POI 当单元格值为空或 null 时返回什么。`MissingCellPolicy`枚举包含三个枚举值:

*   `RETURN_NULL_AND_BLANK`
*   `RETURN_BLANK_AS_NULL`
*   `CREATE_NULL_AS_BLANK`；

单元迭代的代码如下:

```java
for (int cellIndex = row.getFirstCellNum(); cellIndex < row.getLastCellNum(); cellIndex++) {
    Cell cell = row.getCell(cellIndex, Row.MissingCellPolicy.CREATE_NULL_AS_BLANK);
    ...
}
```

### 3.3.读取 Excel 中的单元格值

正如我们之前提到的，Microsoft Excel 的单元格可以包含不同的值类型，因此能够区分不同的单元格值类型并使用适当的方法提取值非常重要。下面是所有值类型的列表:

*   `NONE`
*   `NUMERIC`
*   `STRING`
*   `FORMULA`
*   `BLANK`
*   `BOOLEAN`
*   `ERROR`

我们将关注四种主要的单元格值类型:`Numeric, String, Boolean, and Formula`，其中最后一种包含前三种类型的计算值。

让我们创建一个 helper 方法，它主要检查每个值类型，并基于此使用适当的方法来访问值。也可以[将单元格值视为一个字符串，并用相应的方法检索它](/web/20220627183230/https://www.baeldung.com/java-apache-poi-cell-string-value)。

有两件重要的事情值得注意。首先，`Date`值存储为`Numeric`值，同样如果单元格的值类型为`FORMULA`，我们需要使用`getCachedFormulaResultType()`而不是`getCellType()` 方法[来检查公式计算的结果](/web/20220627183230/https://www.baeldung.com/apache-poi-read-cell-value-formula):

```java
public static void printCellValue(Cell cell) {
    CellType cellType = cell.getCellType().equals(CellType.FORMULA)
      ? cell.getCachedFormulaResultType() : cell.getCellType();
    if (cellType.equals(CellType.STRING)) {
        System.out.print(cell.getStringCellValue() + " | ");
    }
    if (cellType.equals(CellType.NUMERIC)) {
        if (DateUtil.isCellDateFormatted(cell)) {
            System.out.print(cell.getDateCellValue() + " | ");
        } else {
            System.out.print(cell.getNumericCellValue() + " | ");
        }
    }
    if (cellType.equals(CellType.BOOLEAN)) {
        System.out.print(cell.getBooleanCellValue() + " | ");
    }
}
```

现在，我们需要做的就是在单元循环中调用`printCellValue`方法，我们就完成了。以下是完整代码的一部分:

```java
...
for (int cellIndex = row.getFirstCellNum(); cellIndex < row.getLastCellNum(); cellIndex++) {
    Cell cell = row.getCell(cellIndex, Row.MissingCellPolicy.CREATE_NULL_AS_BLANK);
    printCellValue(cell);
}
...
```

## 4.结论

在本文中，我们展示了一个使用 Apache POI 读取 Excel 文件和访问不同单元格值的示例项目。

完整的源代码可以在 GitHub 上找到[。](https://web.archive.org/web/20220627183230/https://github.com/eugenp/tutorials/tree/master/apache-poi)