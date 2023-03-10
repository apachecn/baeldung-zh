# 用 Spring MVC 上传和显示 Excel 文件

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-mvc-excel-files>

## 1。简介

在本文中，我们将演示如何使用`Spring MVC`框架**上传 Excel 文件并在网页**中显示其内容。

## 2。上传 Excel 文件

为了能够上传文件，我们将首先创建一个控制器映射，它接收一个`MultipartFile`并将其保存在当前位置:

```java
private String fileLocation;

@PostMapping("/uploadExcelFile")
public String uploadFile(Model model, MultipartFile file) throws IOException {
    InputStream in = file.getInputStream();
    File currDir = new File(".");
    String path = currDir.getAbsolutePath();
    fileLocation = path.substring(0, path.length() - 1) + file.getOriginalFilename();
    FileOutputStream f = new FileOutputStream(fileLocation);
    int ch = 0;
    while ((ch = in.read()) != -1) {
        f.write(ch);
    }
    f.flush();
    f.close();
    model.addAttribute("message", "File: " + file.getOriginalFilename() 
      + " has been uploaded successfully!");
    return "excel";
}
```

接下来，让我们用一个包含`type file`的`input`的表单创建一个`JSP`文件，该表单将`accept`属性设置为只允许 Excel 文件:

```java
<c:url value="/uploadExcelFile" var="uploadFileUrl" />
<form method="post" enctype="multipart/form-data"
  action="${uploadFileUrl}">
    <input type="file" name="file" accept=".xls,.xlsx" /> <input
      type="submit" value="Upload file" />
</form>
```

## 3。读取 Excel 文件

为了解析上传的 excel 文件，我们将使用`Apache POI`库，它可以处理`.xls`和`.xlsx`文件。

让我们创建一个名为`MyCell`的助手类，它将包含与内容和格式相关的 Excel 单元格的属性:

```java
public class MyCell {
    private String content;
    private String textColor;
    private String bgColor;
    private String textSize;
    private String textWeight;

    public MyCell(String content) {
        this.content = content;
    }

    //standard constructor, getters, setters
}
```

我们将把 Excel 文件的内容读入一个包含`MyCell`对象列表的`Map`中。

### 3.1。解析. xls 文件

**一个`.xls`文件在`Apache POI`库中由一个`HSSFWorkbook`** **类**表示，该类由`HSSFSheet`对象组成。为了打开和阅读一个`.xls`文件的内容，你可以查看我们关于[在 Java 中使用 Microsoft Excel](/web/20220815042739/https://www.baeldung.com/java-microsoft-excel)的文章。

**为了解析单元格的格式，我们将获得`HSSFCellStyle`** **对象，**，它可以帮助我们确定背景颜色和字体等属性。所有读取的属性将在`MyCell`对象的属性中设置:

```java
HSSFCellStyle cellStyle = cell.getCellStyle();

MyCell myCell = new MyCell();

HSSFColor bgColor = cellStyle.getFillForegroundColorColor();
if (bgColor != null) {
    short[] rgbColor = bgColor.getTriplet();
    myCell.setBgColor("rgb(" + rgbColor[0] + ","
      + rgbColor[1] + "," + rgbColor[2] + ")");
    }
HSSFFont font = cell.getCellStyle().getFont(workbook);
```

**颜色以`rgb(rVal, gVal, bVal)`格式读取，以便在`JSP`页面中使用`CSS`更容易显示。**

让我们也获得字体大小、粗细和颜色:

```java
myCell.setTextSize(font.getFontHeightInPoints() + "");
if (font.getBold()) {
    myCell.setTextWeight("bold");
}
HSSFColor textColor = font.getHSSFColor(workbook);
if (textColor != null) {
    short[] rgbColor = textColor.getTriplet();
    myCell.setTextColor("rgb(" + rgbColor[0] + ","
      + rgbColor[1] + "," + rgbColor[2] + ")");
}
```

### 3.2。解析. xlsx 文件

**对于较新的`.xlsx`格式的文件，我们可以使用`XSSFWorkbook`类**和类似的用于工作簿内容的类，也可以在[在 Java 中使用 Microsoft Excel](/web/20220815042739/https://www.baeldung.com/java-microsoft-excel)一文中找到。

让我们仔细看看如何以`.xlsx`格式读取单元格的格式。首先，**我们将检索与单元格相关联的`XSSFCellStyle`对象**，并使用它来确定背景颜色和字体:

```java
XSSFCellStyle cellStyle = cell.getCellStyle();

MyCell myCell = new MyCell();
XSSFColor bgColor = cellStyle.getFillForegroundColorColor();
if (bgColor != null) {
    byte[] rgbColor = bgColor.getRGB();
    myCell.setBgColor("rgb(" 
      + (rgbColor[0] < 0 ? (rgbColor[0] + 0xff) : rgbColor[0]) + ","
      + (rgbColor[1] < 0 ? (rgbColor[1] + 0xff) : rgbColor[1]) + ","
      + (rgbColor[2] < 0 ? (rgbColor[2] + 0xff) : rgbColor[2]) + ")");
}
XSSFFont font = cellStyle.getFont();
```

在这种情况下，颜色的`RGB`值将是`signed byte`值，因此我们将通过将`0xff`与负值相加来获得`unsigned`值。

让我们也确定字体的属性:

```java
myCell.setTextSize(font.getFontHeightInPoints() + "");
if (font.getBold()) {
    myCell.setTextWeight("bold");
}
XSSFColor textColor = font.getXSSFColor();
if (textColor != null) {
    byte[] rgbColor = textColor.getRGB();
    myCell.setTextColor("rgb("
      + (rgbColor[0] < 0 ? (rgbColor[0] + 0xff) : rgbColor[0]) + "," 
      + (rgbColor[1] < 0 ? (rgbColor[1] + 0xff) : rgbColor[1]) + "," 
      + (rgbColor[2] < 0 ? (rgbColor[2] + 0xff) : rgbColor[2]) + ")");
}
```

### 3.3。处理空行

**上述方法不考虑 Excel 文件中的空行。**如果我们想要忠实地再现显示空行的文件，我们将需要在我们的结果`HashMap`中模拟这些，用包含空`Strings`的`MyCell`对象的`ArrayList`作为内容。

最初，读取 Excel 文件后，文件中的空行将是大小为 0 的`ArrayList`对象。

为了确定我们应该添加多少空的`String`对象，我们将首先使用`maxNrCols`变量确定 Excel 文件中最长的一行。然后，我们将把这个数量的空的`String`对象添加到我们的`HashMap`中所有大小为 0:

```java
int maxNrCols = data.values().stream()
  .mapToInt(List::size)
  .max()
  .orElse(0);

data.values().stream()
  .filter(ls -> ls.size() < maxNrCols)
  .forEach(ls -> {
      IntStream.range(ls.size(), maxNrCols)
        .forEach(i -> ls.add(new MyCell("")));
  });
```

## 4。显示 Excel 文件

为了显示使用`Spring MVC`读取的 Excel 文件，我们需要定义一个控制器映射和`JSP`页面。

### 4.1。弹簧 MVC 控制器

让我们创建一个`@RequestMapping`方法，该方法将调用上面的代码来读取上传文件的内容，然后添加返回的`Map`作为一个`Model`属性:

```java
@Resource(name = "excelPOIHelper")
private ExcelPOIHelper excelPOIHelper;

@RequestMapping(method = RequestMethod.GET, value = "/readPOI")
public String readPOI(Model model) throws IOException {

  if (fileLocation != null) {
      if (fileLocation.endsWith(".xlsx") || fileLocation.endsWith(".xls")) {
          Map<Integer, List<MyCell>> data
            = excelPOIHelper.readExcel(fileLocation);
          model.addAttribute("data", data);
      } else {
          model.addAttribute("message", "Not a valid excel file!");
      }
  } else {
      model.addAttribute("message", "File missing! Please upload an excel file.");
  }
  return "excel";
}
```

### 4.2。JSP

为了直观地显示文件的内容，我们将创建一个`HTML` `table`，并在每个表格单元格的`style`属性中添加 Excel 文件中每个单元格对应的格式属性:

```java
<c:if test="${not empty data}">
    <table style="border: 1px solid black; border-collapse: collapse;">
        <c:forEach items="${data}" var="row">
            <tr>
                <c:forEach items="${row.value}" var="cell">
                    <td style="border:1px solid black;height:20px;width:100px;
                      background-color:${cell.bgColor};color:${cell.textColor};
                      font-weight:${cell.textWeight};font-size:${cell.textSize}pt;">
                      ${cell.content}
                    </td>
                </c:forEach>
            </tr>
        </c:forEach>
    </table>
</c:if>
```

## 5。结论

在本文中，我们展示了一个使用`Spring MVC`框架上传 Excel 文件并在网页中显示它们的示例项目。

完整的源代码可以在 [GitHub 项目](https://web.archive.org/web/20220815042739/https://github.com/eugenp/tutorials/tree/master/spring-web-modules/spring-mvc-java)中找到。