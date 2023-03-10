# 使用 Apache POI 更改单元格字体样式

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/apache-poi-change-cell-font>

## 1.介绍

Apache POI 是一个开源库，供软件开发人员创建和操作微软 Office 文档。在其他特性中，它允许开发人员以编程方式更改文档格式。

在本文中，我们将讨论如何在使用名为`CellStyle`的类时改变 Microsoft Excel 中单元格的样式。也就是说，使用这个类，我们可以编写代码来修改 Microsoft Excel 文档中单元格的样式。首先，它是 Apache POI 库提供的一个特性，允许在一个工作簿中创建一个具有多种格式属性的样式。其次，该样式可以应用于该工作簿中的多个单元格。

除此之外，我们还将看看使用`CellStyle`类的常见陷阱。

## 2.Apache POI 和 Maven 依赖关系

让我们将 [Apache POI](https://web.archive.org/web/20220528125435/https://search.maven.org/search?q=g:org.apache.poi%20a:poi) 作为依赖项添加到我们的项目`pom.xml`文件中:

```java
<dependency>
    <groupId>org.apache.poi</groupId> 
    <artifactId>poi</artifactId> 
    <version>5.2.0</version> 
</dependency>
```

## 3.正在创建`CellStyle`

让我们从实例化`CellStyle`开始:

```java
Workbook workbook = new XSSFWorkbook(fileLocation);
CellStyle cellStyle = wb.createCellStyle();
```

接下来，我们设置所需的格式属性。例如，下面的代码会将其设置为日期格式:

```java
cellStyle.setDataFormat(createHelper.createDataFormat().getFormat("m/d/yy h:mm"));
```

最重要的是，我们可以设置一个`CellStyle`的多个格式属性来获得想要的样式组合。例如，我们将下面的代码应用于同一个`CellStyle`对象。因此，它具有日期格式样式和居中对齐的文本样式:

```java
cellStyle.setAlignment(HorizontalAlignment.CENTER); 
```

注意 **`CellStyle`有几个我们可以修改**的格式属性:

| 财产 | 描述 |
| `DataFormat` | 单元格的数据格式，如日期 |
| `Alignment` | 单元格的水平对齐类型 |
| `Hidden` | 是否要隐藏单元格 |
| `Indention` | 单元格中文本缩进的空格数 |
| `BorderBottom`,`BorderLeft`，`BorderRight`，`BorderTop` | 用于单元格下、左、右和上边框的边框类型 |
| `Font` | 此样式的字体属性，如字体颜色 |

稍后当我们使用属性`Font`来改变字体样式时，我们将再次看到它。

## 4.使用`CellStyle`格式化字体

`CellStyle`的`Font`属性是我们设置字体相关格式的地方。例如，我们可以设置字体名称、颜色和大小。我们可以设置字体是粗体还是斜体。`Font`的两个属性可以是`true`也可以是`false`。我们还可以将下划线样式设置为:

| 价值 | 财产 |
| `U_NONE` | 不带下划线的文本 |
| `U_SINGLE` | 只有单词加下划线的单下划线文本 |
| `U_SINGLE_ACCOUNTING` | 几乎整个单元格宽度都加下划线的单下划线文本 |
| `U_DOUBLE` | 只有单词加下划线的文本加双下划线 |
| `U_DOUBLE_ACCOUNTING` | 几乎整个单元格宽度都加下划线的文本加双下划线 |

让我们从前面的例子继续。我们将编写一个名为`CellStyler`的类，其中包含一个为警告文本创建样式的方法:

```java
public class CellStyler {
    public CellStyle createWarningColor(Workbook workbook) {
        CellStyle style = workbook.createCellStyle();
        Font font = workbook.createFont();
        font.setFontName("Courier New");
        font.setBold(true);
        font.setUnderline(Font.U_SINGLE);
        font.setColor(HSSFColorPredefined.DARK_RED.getIndex());
        style.setFont(font);

        style.setAlignment(HorizontalAlignment.CENTER);
        style.setVerticalAlignment(VerticalAlignment.CENTER);
        return style;
    }
}
```

现在，让我们创建一个 Apache POI 工作簿并获取第一张工作表:

```java
Workbook workbook = new XSSFWorkbook(fileLocation);
Sheet sheet = workbook.getSheetAt(0);
```

请注意，我们正在**设置行高，以便我们可以看到文本对齐的效果**:

```java
Row row1 = sheet.createRow(0);
row1.setHeightInPoints((short) 40);
```

让我们实例化这个类，并用它来设置样式

```java
CellStyler styler = new CellStyler();
CellStyle style = styler.createWarningColor(workbook);

Cell cell1 = row1.createCell(0);
cell1.setCellStyle(style);
cell1.setCellValue("Hello");

Cell cell2 = row1.createCell(1);
cell2.setCellStyle(style);
cell2.setCellValue("world!");
```

现在，让我们将此工作簿保存到一个文件中，并在 Microsoft Excel 中打开该文件以查看字体样式效果，我们应该会看到:

[![Result of applying font style](img/de0f8e944e63797e3d6d7d71098aebdb.png)](/web/20220528125435/https://www.baeldung.com/wp-content/uploads/2022/01/Change_Cell_Font_Style_with_Apache_POI-e1638669975805.png)

## 5.常见陷阱

我们来看两个使用`CellStyle`时常见的错误。

### 5.1.意外修改所有单元格样式

首先，从单元格获取`CellStyle`并开始修改它是一个常见的错误。关于`getCellStyle`方法的 Apache POI 文档提到**一个单元格的`getCellStyle`方法将总是返回一个非空值**。这意味着单元格有默认值，这也是工作簿中所有单元格最初使用的默认样式。因此，下面的代码将使所有单元格都具有日期格式:

```java
cell.setCellValue(rdf.getEffectiveDate());
cell.getCellStyle().setDataFormat(HSSFDataFormat.getBuiltinFormat("d-mmm-yy"));
```

### 5.2.为每个单元格创建新样式

另一个常见的错误是在一个工作簿中有太多相似的样式:

```java
CellStyle style1 = codeToCreateCellStyle();
Cell cell1 = row1.createCell(0);
cell1.setCellStyle(style1);

CellStyle style2 = codeToCreateCellStyle();
Cell cell2 = row1.createCell(1);
cell2.setCellStyle(style2);
```

一个 **`CellStyle`的作用域是一个工作簿**。因此，相似的样式应该由多个单元格共享。在上面的例子中，样式应该只创建一次，并在`cell1`和`cell2`之间共享:

```java
CellStyle style1 = codeToCreateCellStyle();
Cell cell1 = row1.createCell(0);
cell1.setCellStyle(style1);
cell1.setCellValue("Hello");

Cell cell2 = row1.createCell(1);
cell2.setCellStyle(style1);
cell2.setCellValue("world!");
```

## 6.摘要

在本文中，我们学习了如何使用`CellStyle`及其`Font`属性来设计 Apache POI 中的单元格。如本文所述，如果我们设法避免陷阱，对单元格进行样式化就相对简单。

在代码示例中，我们展示了如何根据需要以编程方式设置电子表格文档的样式，就像我们使用 Excel 应用程序本身一样。当需要生成一个具有漂亮的数据显示的电子表格时，这是最重要的。

和往常一样，这篇文章的源代码可以在 GitHub 的[上找到。](https://web.archive.org/web/20220528125435/https://github.com/eugenp/tutorials/tree/master/apache-poi)