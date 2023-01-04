# 使用 Apache POI 设置日期格式

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-apache-poi-date-format>

## 1。简介

当我们在 Apache POI 中处理日期时，我们希望确保它们的格式正确。

幸运的是，使用 Apache POI 很容易设置日期格式。在本教程中，我们将展示如何将日期的自定义`DataFormat`定义为带有 [Apache POI](/web/20221212045748/https://www.baeldung.com/java-microsoft-excel) 的`CellStyle`，以及如何使用现有的`DataFormats`。

## 2。起点

我们的起点将是一个新的`XSSFWorkbook,` 一个`XSSFCell,` 和一个已经创建的`CellStyle:`

```java
XSSFWorkbook wb = new XSSFWorkbook();
CellStyle cellStyle = wb.createCellStyle();
wb.createSheet();
XSSFSheet sheet = wb.getSheetAt(0);
XSSFCell dateCell = sheet.createRow(0).createCell(0);
dateCell.setCellValue(new Date());
```

由于我们还没有设置我们的愿望`DataFormat`，我们的`Date`将被转换成一个数字，并显示为:

> 44898,9262857176

这种表现方式对于我们人类来说可读性很差。因此，在下面，我们将看看如何通过格式化来创建更好的可视化。

## 2。`DataFormat`创建自定义

首先，我们需要创建一个新的`CreationHelper`。有了`CreationHelper, `，我们可以创建一个新的`DataFormat `和一个特定的`Format`。这个`DataFormat`是内部存储的，由一个短整型变量引用。我们必须将它添加到`CellStyle`本身，并将`CellStyle`应用到`Cell`:

```java
CreationHelper createHelper = wb.getCreationHelper();
short format = createHelper.createDataFormat().getFormat("m.d.yy h:mm");
cellStyle.setDataFormat(format);
dateCell.setCellStyle(cellStyle);
```

在我们设置了这个自定义`CellStyle`之后，我们的日期将被格式化为:

```java
02.12.2022 21:30
```

然而，如果我们创建一个新的自定义`DataFormat,`,我们应该记住 Excel 工作簿最多支持 65.000 个单元格样式。所以**我们应该总是重用现有的单元格样式**，并尽可能将它们应用于多个单元格。

## 3。使用默认的`DataFormat`

正如我们所了解的，Apache POI 使用缩写链接到不同的`DataFormats.` Excel 已经有许多**内置`DataFormats`，我们可以通过使用它们的缩写直接调用它们**来使用它们:

```java
cellStyle.setDataFormat((short) 14);
dateCell.setCellStyle(cellStyle);
```

之后，我们可以用下面的代码行获得一个`String`表示中的`DataFormat`:

```java
cellStyle.getDataFormatString();
```

在我们的示例中，我们会得到以下结果:

```java
m/d/yy
```

最常见的`DataFormats`有:

| **短值** | **格式** |
| Fourteen | 年月日 |
| Fifteen | 嗯，yy |
| Sixteen | 嗯嗯 |
| Seventeen | 嗯-yy |
| Eighteen | h:mm 上午/下午 |
| Nineteen | 小时:分钟:上午/下午 |
| Twenty | 亨:嗯 |
| Twenty-one | h:mm:ss |
| Twenty-two | 月/日/年时:分 |

如果符合我们的需要，我们应该总是使用其中一种数据格式，因为 Excel 会将它们显示为其中一种格式，而不是自定义格式。这也将**触发 Excel 使用格式**的本地化可视化。

## 4。结论

正如我们所看到的，使用 Apache POI 设置日期格式既快速又简单，但也是以人类可读的方式可视化日期所必需的。下次在电子表格中处理日期时，我们应该尝试一下。

GitHub 上的[提供了完整的示例。](https://web.archive.org/web/20221212045748/https://github.com/eugenp/tutorials/tree/master/apache-poi-2)