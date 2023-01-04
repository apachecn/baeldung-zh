# 使用 Apache POI 在 Excel 中插入一行

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/apache-poi-insert-excel-row>

## 1.概观

有时，我们可能需要在 Java 应用程序中操作 Excel 文件。

在本教程中，我们将特别关注使用 [Apache POI](/web/20220701010912/https://www.baeldung.com/java-microsoft-excel) 库在 Excel 文件的两行之间插入新行。

## 2.Maven 依赖性

首先，我们必须将 [poi-ooxml](https://web.archive.org/web/20220701010912/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22org.apache.poi%22%20AND%20a%3A%22poi-ooxml%22) Maven 依赖项添加到 pom.xml 文件中:

```java
<dependency>
    <groupId>org.apache.poi</groupId>
    <artifactId>poi-ooxml</artifactId>
    <version>5.2.0</version>
</dependency>
```

## 3.在两行之间插入行

### 3.1.Apache POI 相关类

Apache POI 是一个库的集合——每个库都专用于操作特定类型的文件。`XSSF`库包含处理`xlsx` Excel 格式的类。下图显示了用于操作`xlsx` Excel 文件的 Apache POI 相关接口和类:

[![](img/5fe928da5cdf3bd793220ea004036526.png)](/web/20220701010912/https://www.baeldung.com/wp-content/uploads/2021/02/Figures-Page-4.png)

### 3.2.实现行插入

对于在现有 Excel 表中间插入`m`行，从插入点到最后一行的所有行都应该下移`m`行。

首先，我们需要读取 Excel 文件。对于这一步，我们使用了`XSSFWorkbook`类:

```java
Workbook workbook = new XSSFWorkbook(fileLocation);
```

第二步是使用`getSheet()`方法访问工作簿中的工作表:

```java
Sheet sheet = workbook.getSheetAt(0);
```

第三步是移动行，从我们希望开始插入新行的当前行开始，移动到工作表的最后一行:

```java
int lastRow = sheet.getLastRowNum(); 
sheet.shiftRows(startRow, lastRow, rowNumber, true, true); 
```

在这一步中，我们通过使用`getLastRowNum()`方法获得最后一个行号，并使用`shiftRows()` 方法移动这些行。该方法根据`rowNumber`的大小在`startRow`和`lastRow`之间移动行。

最后，我们使用`createRow()`方法插入新行:

```java
sheet.createRow(startRow);
```

值得注意的是，上面的实现将保留被移动的行的格式。此外，如果在我们要移动的区域中有隐藏的行，它们会在插入新行时移动。

### 3.3.单元测试

让我们编写一个测试用例，它读取资源目录中的工作簿，然后在位置 2 插入一行，并将内容写入一个新的 Excel 文件。最后，我们用主文件断言结果文件的行号。

让我们定义一个测试用例:

```java
public void givenWorkbook_whenInsertRowBetween_thenRowCreated() {
    int startRow = 2;
    int rowNumber = 1;
    Workbook workbook = new XSSFWorkbook(fileLocation);
    Sheet sheet = workbook.getSheetAt(0);

    int lastRow = sheet.getLastRowNum();
    if (lastRow < startRow) {
        sheet.createRow(startRow);
    }

    sheet.shiftRows(startRow, lastRow, rowNumber, true, true);
    sheet.createRow(startRow);

    FileOutputStream outputStream = new FileOutputStream(NEW_FILE_NAME);
    workbook.write(outputStream);

    File file = new File(NEW_FILE_NAME);

    final int expectedRowResult = 5;
    Assertions.assertEquals(expectedRowResult, workbook.getSheetAt(0).getLastRowNum());

    outputStream.close();
    file.delete();
    workbook.close();
}
```

## 4.结论

总之，我们已经学习了如何使用 Apache POI 库在 Excel 文件的两行之间插入一行。和往常一样，GitHub 上的[提供了这篇文章的完整源代码。](https://web.archive.org/web/20220701010912/https://github.com/eugenp/tutorials/tree/master/apache-poi)