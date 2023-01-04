# opencv 简介

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/opencsv>

## 1。简介

在这个快速教程中，我们将介绍 OpenCSV 4，这是一个非常棒的库，用于编写、读取、序列化、反序列化和/或解析`.csv`文件。然后我们将通过几个例子演示如何设置和使用 OpenCSV 4。

## 2。设置

首先，我们将通过`pom.xml`依赖关系将 OpenCSV 添加到我们的项目中:

```
<dependency>
    <groupId>com.opencsv</groupId>
    <artifactId>opencsv</artifactId>
    <version>4.1</version>
</dependency> 
```

OpenCSV 的`.jars`可以在[官方网站](https://web.archive.org/web/20221208143841/http://opencsv.sourceforge.net/)找到，或者在 [Maven 资源库](https://web.archive.org/web/20221208143841/https://mvnrepository.com/artifact/com.opencsv/opencsv)快速搜索。

我们的`.csv`文件将非常简单；我们将保持两列四行:

```
colA,colB
A,B
C,D
G,G
G,F
```

## 3。去豆还是不去豆

将 OpenCSV 添加到我们的`pom.xml`之后，我们可以用两种方便的方式实现 CSV 处理方法:

1.  使用方便的`CSVReader`和`CSVWriter`对象(用于更简单的操作)
2.  使用`CsvToBean`将`.csv`文件转换成 beans(作为带注释的`plain-old-java-objects`实现)

在本文中，我们将坚持使用`synchronous`(或`blocking`)示例，因此我们可以专注于基础知识。

记住，`synchronous`方法会阻止周围或后续的代码执行，直到它完成。生产环境可能会使用`asynchronous`(或`non-blocking`)方法，当`asynchronous`方法完成时，允许其他过程或方法完成。

我们将在以后的文章中深入探讨 OpenCSV 的`asynchronous`示例。

### 3.1。`CSVReader`

让我们通过提供的`readAll()` 和`readNext()`方法来探索`CSVReader`。我们将看看如何同步使用`readAll`():

```
public List<String[]> readAllLines(Path filePath) throws Exception {
    try (Reader reader = Files.newBufferedReader(filePath)) {
        try (CSVReader csvReader = new CSVReader(reader)) {
            return csvReader.readAll();
        }
    }
}
```

然后我们可以通过传入一个文件`Path`来调用该方法:

```
public List<String[]> readAllExample() throws Exception {
    Path path = Paths.get(
      ClassLoader.getSystemResource("csv/twoColumn.csv").toURI())
    );
    return CsvReaderExamples.readAllLines(path);
}
```

同样，我们可以抽象出`readNext`()，它逐行读取一个提供的`.csv`:

```
public List<String[]> readLineByLine(Path filePath) throws Exception {
    List<String[]> list = new ArrayList<>();
    try (Reader reader = Files.newBufferedReader(filePath)) {
        try (CSVReader csvReader = new CSVReader(reader)) {
            String[] line;
            while ((line = csvReader.readNext()) != null) {
                list.add(line);
            }
        }
    }
    return list;
}
```

最后，我们可以通过传入一个文件`Path:`来调用这个方法

```
public List<String[]> readLineByLineExample() throws Exception {
    Path path = Paths.get(
      ClassLoader.getSystemResource("csv/twoColumn.csv").toURI())
    );
    return CsvReaderExamples.readLineByLine(path);
} 
```

或者，为了获得更大的灵活性和配置选项，我们可以使用`CSVReaderBuilder`:

```
CSVParser parser = new CSVParserBuilder()
    .withSeparator(',')
    .withIgnoreQuotations(true)
    .build();

CSVReader csvReader = new CSVReaderBuilder(reader)
    .withSkipLines(0)
    .withCSVParser(parser)
    .build();
```

`CSVReaderBuilder` 允许我们跳过列标题，通过`CSVParserBuilder`设置解析规则。

**使用`CSVParserBuilder`，我们可以选择自定义的列分隔符，忽略或处理引号，声明我们将如何处理空字段，以及我们将如何解释转义字符**。有关这些配置设置的更多信息，请参考官方规范[文档](https://web.archive.org/web/20221208143841/http://opencsv.sourceforge.net/apidocs/index.html)。

像往常一样，我们需要记住关闭所有的`Readers`来防止内存泄漏。

### 3.2。`CSVWriter`

*CSVWriter* 同样提供一次或逐行写入一个`.csv`文件的能力。

让我们看看如何逐行写入一个 `.csv`:

```
public String writeLineByLine(List<String[]> lines, Path path) throws Exception {
    try (CSVWriter writer = new CSVWriter(new FileWriter(path.toString()))) {
        for (String[] line : lines) {
            writer.writeNext(array);
        }
    return Helpers.readFile(path);
} 
```

然后，我们将指定保存文件的位置，并调用我们刚刚编写的方法:

```
public String writeLinebylineExample() throws Exception {
    Path path = Paths.get(
      ClassLoader.getSystemResource("csv/writtenOneByOne.csv").toURI()
    ); 
    return CsvWriterExamples.writeLineByLine(Helpers.fourColumnCsvString(), path); 
}
```

我们也可以通过传入代表我们的`.csv:`的行的`String`数组的`List`来一次写入我们的`.csv`

```
public String writeAllLines(List<String[]> lines, Path path) throws Exception {
    try (CSVWriter writer = new CSVWriter(new FileWriter(path.toString()))) {
        writer.writeAll(stringArray);
    }
    return Helpers.readFile(path);
}
```

最后，我们这样称呼它:

```
public String writeAllExample() throws Exception {
    Path path = Paths.get(
      ClassLoader.getSystemResource("csv/writtenAll.csv").toURI()
    ); 
    return CsvWriterExamples.writeAllLines(Helpers.fourColumnCsvString(), path);
}
```

### 3.3。基于 Bean 的阅读

OpenCSV 能够将`.csv`文件序列化为预设的、可重用的模式，作为带注释的 Java`pojo`bean 来实现。`CsvToBean` 是用 `CsvToBeanBuilder`建造的。从 OpenCSV 4 开始， **`CsvToBeanBuilder` 是与** `**com.opencsv.bean.CsvToBean**.`协同工作的推荐方式

这里有一个简单的 bean，我们可以用它来序列化前面的两列 `.csv`:

```
public class SimplePositionBean  {
    @CsvBindByPosition(position = 0)
    private String exampleColOne;

    @CsvBindByPosition(position = 1)
    private String exampleColTwo;

    // getters and setters
} 
```

`.csv`文件中的每一列都与 bean 中的一个字段相关联。我们可以使用`@CsvBindByPosition`或`@CsvBindByName`注释**来执行`.csv` 列标题之间的映射，这两个注释分别通过位置或标题字符串匹配**来指定映射。

首先，我们将创建一个名为`CsvBean`的超类，它将允许我们重用和概括我们将在下面构建的方法:

```
public class CsvBean { }
```

下面是一个子类示例:

```
public class NamedColumnBean extends CsvBean {

    @CsvBindByName(column = "name")
    private String name;

    // Automatically infer column name as 'Age'
    @CsvBindByName
    private int age;

    // getters and setters
}
```

接下来，我们将使用`CsvToBean`抽象一个同步返回的`List` :

```
public List<CsvBean> beanBuilderExample(Path path, Class clazz) throws Exception {
    CsvTransfer csvTransfer = new CsvTransfer();

    try (Reader reader = Files.newBufferedReader(path)) {
        CsvToBean<CsvBean> cb = new CsvToBeanBuilder<CsvBean>(reader)
         .withType(clazz)
         .build();

        csvTransfer.setCsvList(cb.parse());
    }
    return csvTransfer.getCsvList();
}
```

然后我们传入我们的 bean ( `clazz`)，这样，我们将 bean 的字段与我们的`.csv`行的相应列相关联。

我们可以用上面写的 `CsvBean` 的子类`SimplePositionBean` 来调用它:

```
public List<CsvBean> simplePositionBeanExample() throws Exception {
    Path path = Paths.get(
      ClassLoader.getSystemResource("csv/twoColumn.csv").toURI()); 
    return BeanExamples.beanBuilderExample(path, SimplePositionBean.class); 
}
```

我们也可以在这里用 `CsvBean`的另一个子类`NamedColumnBean,` 来调用它:

```
public List<CsvBean> namedColumnBeanExample() throws Exception {
    Path path = Paths.get(
      ClassLoader.getSystemResource("csv/namedColumn.csv").toURI()); 
    return BeanExamples.beanBuilderExample(path, NamedColumnBean.class);
}
```

### 3.4.基于 Bean 的写作

最后，让我们来看看如何使用`StatefulBeanToCsv` 类写入一个`.csv`文件:

```
public String writeCsvFromBean(Path path) throws Exception {

    List<CsvBean> sampleData = Arrays.asList(
        new WriteExampleBean("Test1", "sfdsf", "fdfd"),
        new WriteExampleBean("Test2", "ipso", "facto")
    );

    try (Writer writer  = new FileWriter(path.toString())) {

        StatefulBeanToCsv<CsvBean> sbc = new StatefulBeanToCsvBuilder<CsvBean>(writer)
          .withQuotechar('\'')
          .withSeparator(CSVWriter.DEFAULT_SEPARATOR)
          .build();

        sbc.write(sampleData);
    }
    return Helpers.readFile(path);
}
```

在这里，我们指定如何界定和引用我们的数据，这些数据作为指定的`CsvBean`对象的`List`提供。

在传入所需的输出文件路径后，我们可以调用我们的方法`writeCsvFromBean()`:

```
public String writeCsvFromBeanExample() {
    Path path = Paths.get(
      ClassLoader.getSystemResource("csv/writtenBean.csv").toURI()); 
    return BeanExamples.writeCsvFromBean(path); 
}
```

## 4。结论

在这篇简短的文章中，我们讨论了使用 beans、`CSVReader`和`CSVWriter`的 **OpenCSV** 的同步代码示例。更多信息，请点击这里查看官方文件[。](https://web.archive.org/web/20221208143841/http://opencsv.sourceforge.net/)

与往常一样，GitHub 上的[提供了代码示例。](https://web.archive.org/web/20221208143841/https://github.com/eugenp/tutorials/tree/master/libraries-data-io)