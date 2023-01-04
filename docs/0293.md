# 在 Java 中使用 Microsoft Excel

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-microsoft-excel>

## 1。概述

在本教程中，我们将演示如何使用 **Apache POI 和 JExcel APIs 来处理 Excel 电子表格。**

这两个库都可以用来动态地读取、写入和修改 Excel 电子表格的内容，并提供了一种将 Microsoft Excel 集成到 Java 应用程序中的有效方法。

## 延伸阅读:

## [使用 Apache POI 在 Excel 中插入一行](/web/20221128104701/https://www.baeldung.com/apache-poi-insert-excel-row)

Learn how to inser a new row between two rows in an Excel file using the Apache POI library[Read more](/web/20221128104701/https://www.baeldung.com/apache-poi-insert-excel-row) →

## [使用 Apache POI 向 Excel 表添加列](/web/20221128104701/https://www.baeldung.com/java-excel-add-column)

Learn how to add a column to an Excel sheet using Java with the Apache POI library.[Read more](/web/20221128104701/https://www.baeldung.com/java-excel-add-column) →

## [用 Java 从 Excel 中读取数值](/web/20221128104701/https://www.baeldung.com/java-read-dates-excel)

Learn how to access different cell values using Apache POI.[Read more](/web/20221128104701/https://www.baeldung.com/java-read-dates-excel) →

## 2。Maven 依赖关系

首先，我们需要将以下依赖项添加到我们的`pom.xml`文件中:

```
<dependency>
    <groupId>org.apache.poi</groupId>
    <artifactId>poi</artifactId>
    <version>5.2.0</version>
</dependency>
<dependency>
    <groupId>org.apache.poi</groupId>
    <artifactId>poi-ooxml</artifactId>
    <version>5.2.0</version>
</dependency>
```

poi-ooxml 和 [jxls-jexcel](https://web.archive.org/web/20221128104701/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22org.jxls%22%20AND%20a%3A%22jxls-jexcel%22) 的最新版本可以从 Maven Central 下载。

## 3。阿帕奇兴趣点

**Apache POI 库支持`.xls`和 `.xlsx`文件**，是一个比其他处理 Excel 文件的 Java 库更复杂的库。

它提供了用于建模`Excel`文件的`Workbook`接口，以及用于建模 Excel 文件元素的`Sheet`、`Row`和`Cell`接口，以及两种文件格式的每个接口的实现。

当处理较新的`.xlsx`文件格式时，我们将使用`XSSFWorkbook`、 `XSSFSheet`、 `XSSFRow and XSSFCell` 类`.`

为了使用旧的`.xls`格式，我们使用了`HSSFWorkbook`、 `HSSFSheet`、 `HSSFRow` 和 `HSSFCell` 类`.`

### 3.1。从 Excel 中读取

让我们创建一个方法，打开一个`.xlsx`文件，然后从文件的第一页读取内容。

读取单元格内容的方法因单元格中数据的类型而异。使用`Cell`接口的`getCellType()`方法可以确定单元格内容的类型。

首先，让我们从给定的位置打开文件:

```
FileInputStream file = new FileInputStream(new File(fileLocation));
Workbook workbook = new XSSFWorkbook(file);
```

接下来，让我们检索文件的第一页，并遍历每一行:

```
Sheet sheet = workbook.getSheetAt(0);

Map<Integer, List<String>> data = new HashMap<>();
int i = 0;
for (Row row : sheet) {
    data.put(i, new ArrayList<String>());
    for (Cell cell : row) {
        switch (cell.getCellType()) {
            case STRING: ... break;
            case NUMERIC: ... break;
            case BOOLEAN: ... break;
            case FORMULA: ... break;
            default: data.get(new Integer(i)).add(" ");
        }
    }
    i++;
}
```

**Apache POI 有不同的方法来读取每种类型的数据。**让我们展开上面每个开关案例的内容。

当单元格类型枚举值为`STRING`时，将使用`Cell`接口的`getRichStringCellValue()`方法读取内容:

```
data.get(new Integer(i)).add(cell.getRichStringCellValue().getString());
```

内容类型为`NUMERIC`的单元格可以包含日期或数字，读取方式如下:

```
if (DateUtil.isCellDateFormatted(cell)) {
    data.get(i).add(cell.getDateCellValue() + "");
} else {
    data.get(i).add(cell.getNumericCellValue() + "");
}
```

对于`BOOLEAN`值，我们有`getBooleanCellValue()`方法:

```
data.get(i).add(cell.getBooleanCellValue() + "");
```

当单元格类型为`FORMULA`时，我们可以使用`getCellFormula()`方法:

```
data.get(i).add(cell.getCellFormula() + "");
```

### 3.2。写入 Excel

Apache POI 使用上一节中介绍的相同接口来写入 Excel 文件，并且比 JExcel 对样式有更好的支持。

让我们创建一个方法，将人员列表写到标题为`“Persons”`的工作表中。

首先，我们将创建并样式化一个包含`“Name”`和`“Age”`单元格的标题行:

```
Workbook workbook = new XSSFWorkbook();

Sheet sheet = workbook.createSheet("Persons");
sheet.setColumnWidth(0, 6000);
sheet.setColumnWidth(1, 4000);

Row header = sheet.createRow(0);

CellStyle headerStyle = workbook.createCellStyle();
headerStyle.setFillForegroundColor(IndexedColors.LIGHT_BLUE.getIndex());
headerStyle.setFillPattern(FillPatternType.SOLID_FOREGROUND);

XSSFFont font = ((XSSFWorkbook) workbook).createFont();
font.setFontName("Arial");
font.setFontHeightInPoints((short) 16);
font.setBold(true);
headerStyle.setFont(font);

Cell headerCell = header.createCell(0);
headerCell.setCellValue("Name");
headerCell.setCellStyle(headerStyle);

headerCell = header.createCell(1);
headerCell.setCellValue("Age");
headerCell.setCellStyle(headerStyle);
```

接下来，让我们用不同的风格来书写表格的内容:

```
CellStyle style = workbook.createCellStyle();
style.setWrapText(true);

Row row = sheet.createRow(2);
Cell cell = row.createCell(0);
cell.setCellValue("John Smith");
cell.setCellStyle(style);

cell = row.createCell(1);
cell.setCellValue(20);
cell.setCellStyle(style);
```

最后，让我们将内容写入当前目录中的一个`“temp.xlsx”` 文件并关闭工作簿:

```
File currDir = new File(".");
String path = currDir.getAbsolutePath();
String fileLocation = path.substring(0, path.length() - 1) + "temp.xlsx";

FileOutputStream outputStream = new FileOutputStream(fileLocation);
workbook.write(outputStream);
workbook.close();
```

让我们在一个`JUnit`测试中测试上述方法，该测试将内容写入`temp.xlsx`文件，然后读取同一个文件以验证它是否包含我们所写的文本:

```
public class ExcelTest {

    private ExcelPOIHelper excelPOIHelper;
    private static String FILE_NAME = "temp.xlsx";
    private String fileLocation;

    @Before
    public void generateExcelFile() throws IOException {
        File currDir = new File(".");
        String path = currDir.getAbsolutePath();
        fileLocation = path.substring(0, path.length() - 1) + FILE_NAME;

        excelPOIHelper = new ExcelPOIHelper();
        excelPOIHelper.writeExcel();
    }

    @Test
    public void whenParsingPOIExcelFile_thenCorrect() throws IOException {
        Map<Integer, List<String>> data
          = excelPOIHelper.readExcel(fileLocation);

        assertEquals("Name", data.get(0).get(0));
        assertEquals("Age", data.get(0).get(1));

        assertEquals("John Smith", data.get(1).get(0));
        assertEquals("20", data.get(1).get(1));
    }
}
```

## 4。杰克斯尔

JExcel 库是一个轻量级的库，它的优点是比 Apache POI 更容易使用，但缺点是它只支持处理 `.xls` (1997-2003)格式的 Excel 文件。

**目前不支持 `.xlsx`文件。**

### 4.1。从 Excel 中读取

为了使用 Excel 文件，该库提供了一系列表示 Excel 文件不同部分的类。**`Workbook`类代表整个工作表集合。**`Sheet`类表示单个工作表，`Cell`类表示电子表格的单个单元格。

让我们编写一个方法，从指定的 Excel 文件创建一个工作簿，获取文件的第一个工作表，然后遍历其内容，并将每一行添加到一个`HashMap`:

```
public class JExcelHelper {

    public Map<Integer, List<String>> readJExcel(String fileLocation) 
      throws IOException, BiffException {

        Map<Integer, List<String>> data = new HashMap<>();

        Workbook workbook = Workbook.getWorkbook(new File(fileLocation));
        Sheet sheet = workbook.getSheet(0);
        int rows = sheet.getRows();
        int columns = sheet.getColumns();

        for (int i = 0; i < rows; i++) {
            data.put(i, new ArrayList<String>());
            for (int j = 0; j < columns; j++) {
                data.get(i)
                  .add(sheet.getCell(j, i)
                  .getContents());
            }
        }
        return data;
    }
}
```

### 4.2。写入 Excel

为了写入 Excel 文件，JExcel 库提供了类似于上面使用的类，它们模拟了一个电子表格文件:`WritableWorkbook`、`WritableSheet`和`WritableCell`。

**`WritableCell`类有对应于不同类型内容**的子类，可以编写:`Label`、`DateTime`、`Number`、`Boolean`、`Blank`和`Formula`。

这个库还提供对基本格式的支持，比如控制字体、颜色和单元格宽度。

让我们编写一个方法，在当前目录中创建一个名为`“temp.xls”`的工作簿，然后编写与 Apache POI 部分相同的内容。

首先，让我们创建工作簿:

```
File currDir = new File(".");
String path = currDir.getAbsolutePath();
String fileLocation = path.substring(0, path.length() - 1) + "temp.xls";

WritableWorkbook workbook = Workbook.createWorkbook(new File(fileLocation));
```

接下来，让我们创建第一个工作表并编写 excel 文件的标题，包含`“Name”`和`“Age”`单元格:

```
WritableSheet sheet = workbook.createSheet("Sheet 1", 0);

WritableCellFormat headerFormat = new WritableCellFormat();
WritableFont font
  = new WritableFont(WritableFont.ARIAL, 16, WritableFont.BOLD);
headerFormat.setFont(font);
headerFormat.setBackground(Colour.LIGHT_BLUE);
headerFormat.setWrap(true);

Label headerLabel = new Label(0, 0, "Name", headerFormat);
sheet.setColumnView(0, 60);
sheet.addCell(headerLabel);

headerLabel = new Label(1, 0, "Age", headerFormat);
sheet.setColumnView(0, 40);
sheet.addCell(headerLabel);
```

使用新的样式，让我们编写我们创建的表格的内容:

```
WritableCellFormat cellFormat = new WritableCellFormat();
cellFormat.setWrap(true);

Label cellLabel = new Label(0, 2, "John Smith", cellFormat);
sheet.addCell(cellLabel);
Number cellNumber = new Number(1, 2, 20, cellFormat);
sheet.addCell(cellNumber);
```

记住写入文件并在结束时关闭它是非常重要的，这样它就可以被其他进程使用，使用`Workbook`类的`write()`和`close()`方法:

```
workbook.write();
workbook.close();
```

## 5。 **结论**

本文展示了如何使用`Apache POI` API 和 `JExcel` API 从 Java 程序中读取和写入 Excel 文件。

本文的完整源代码可以在 [GitHub 项目](https://web.archive.org/web/20221128104701/https://github.com/eugenp/tutorials/tree/master/apache-poi)中找到。