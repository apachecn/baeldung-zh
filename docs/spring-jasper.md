# 茉莉春天报道

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-jasper>

## 1。概述

JasperReports 是一个开源的报告库，它使用户能够创建像素级完美的报告，这些报告可以以多种格式打印或导出，包括 PDF、HTML 和 XLS。

在本文中，我们将探索它的关键特性和类，并实现示例来展示它的功能。

## 2。Maven 依赖关系

首先，我们需要将`jasperreports`依赖项添加到我们的`pom.xml`中:

```java
<dependency>
    <groupId>net.sf.jasperreports</groupId>
    <artifactId>jasperreports</artifactId>
    <version>6.4.0</version>
</dependency>
```

这个产品的最新版本可以在[这里](https://web.archive.org/web/20220524060340/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22net.sf.jasperreports%22%20AND%20a%3A%22jasperreports%22)找到。

## 3。报告模板

报告设计在 JRXML 文件中定义。这些是普通的 XML 文件，具有 JasperReports 引擎可以解释的特定结构。

现在让我们只看一下 JRXML 文件的相关结构——以便更好地理解报告生成过程的 Java 部分，这是我们的主要关注点。

让我们创建一个简单的报告来显示员工信息:

```java
<jasperReport ... >
    <field name="FIRST_NAME" class="java.lang.String"/>
    <field name="LAST_NAME" class="java.lang.String"/>
    <field name="SALARY" class="java.lang.Double"/>
    <field name="ID" class="java.lang.Integer"/>
    <detail>
        <band height="51" splitType="Stretch">
            <textField>
                <reportElement x="0" y="0" width="100" height="20"/>
                <textElement/>
                <textFieldExpression class="java.lang.String">
                  <![CDATA[$F{FIRST_NAME}]]></textFieldExpression>
            </textField>
            <textField>
                <reportElement x="100" y="0" width="100" height="20"/>
                <textElement/>
                <textFieldExpression class="java.lang.String">
                  <![CDATA[$F{LAST_NAME}]]></textFieldExpression>
            </textField>
            <textField>
                <reportElement x="200" y="0" width="100" height="20"/>
                <textElement/>
                <textFieldExpression class="java.lang.String">
                  <![CDATA[$F{SALARY}]]></textFieldExpression>
            </textField>
        </band>
    </detail>
</jasperReport>
```

### 3.1。汇编报告

JRXML 文件需要编译，以便报表引擎可以用数据填充它们。

让我们在`JasperCompilerManager`类的帮助下执行这个操作:

```java
InputStream employeeReportStream
  = getClass().getResourceAsStream("/employeeReport.jrxml");
JasperReport jasperReport
  = JasperCompileManager.compileReport(employeeReportStream);
```

为了避免每次都编译它，我们可以将它保存到一个文件中:

```java
JRSaver.saveObject(jasperReport, "employeeReport.jasper");
```

## 4。填充 **报告**

填充已编译报告的最常见方式是使用数据库中的记录。这要求报告包含引擎将执行以获取数据的 SQL 查询。

首先，让我们修改我们的报告，添加一个 SQL 查询:

```java
<jasperReport ... >
    <queryString>
        <![CDATA[SELECT * FROM EMPLOYEE]]>
    </queryString>
    ...
</jasperReport>
```

现在，让我们创建一个简单的数据源:

```java
@Bean
public DataSource dataSource() {
    return new EmbeddedDatabaseBuilder()
      .setType(EmbeddedDatabaseType.HSQL)
      .addScript("classpath:employee-schema.sql")
      .build();
}
```

现在，我们可以填写报告:

```java
JasperPrint jasperPrint = JasperFillManager.fillReport(
  jasperReport, null, dataSource.getConnection());
```

注意，我们将`null`传递给第二个参数，因为我们的报告还没有收到任何参数。

### 4.1。参数

参数对于将报表引擎在其数据源中找不到的数据传递给报表引擎，或者当数据根据不同的运行时条件而发生变化时，非常有用。

我们还可以使用在报告填充操作中接收到的参数来更改部分甚至整个 SQL 查询。

首先，让我们修改报告以接收三个参数:

```java
<jasperReport ... >
    <parameter name="title" class="java.lang.String" />
    <parameter name="minSalary" class="java.lang.Double" />
    <parameter name="condition" class="java.lang.String">
        <defaultValueExpression>
          <![CDATA["1 = 1"]]></defaultValueExpression>
    </parameter>
    // ...
</jasperreport>
```

现在，让我们添加一个标题部分来显示`title`参数:

```java
<jasperreport ... >
    // ...
    <title>
        <band height="20" splitType="Stretch">
            <textField>
                <reportElement x="238" y="0" width="100" height="20"/>
                <textElement/>
                <textFieldExpression class="java.lang.String">
                  <![CDATA[$P{title}]]></textFieldExpression>
            </textField>
        </band>
    </title>
    ...
</jasperreport/>
```

接下来，让我们修改查询以使用`minSalary`和`condition`参数:

```java
SELECT * FROM EMPLOYEE
  WHERE SALARY >= $P{minSalary} AND $P!{condition}
```

注意使用`condition`参数时的不同语法。这告诉引擎该参数不应该被用作标准的`PreparedStatement`参数，而是好像该参数的值原本已经被写入 SQL 查询中。

最后，让我们准备参数并填写报告:

```java
Map<String, Object> parameters = new HashMap<>();
parameters.put("title", "Employee Report");
parameters.put("minSalary", 15000.0);
parameters.put("condition", " LAST_NAME ='Smith' ORDER BY FIRST_NAME");

JasperPrint jasperPrint
  = JasperFillManager.fillReport(..., parameters, ...);
```

注意`parameters`的键对应于报告中的参数名。如果引擎检测到某个参数丢失，它将从该参数的`defaultValueExpression`中获取值(如果有)。

## 5。导出

要导出一个报告，首先，我们实例化一个 exporter 类的对象，它匹配我们需要的文件格式。

然后，我们将之前填写的报告设置为输入，并定义输出结果文件的位置。

或者，我们可以设置相应的报告和导出配置对象来定制导出流程。

### 5.1。PDF

```java
JRPdfExporter exporter = new JRPdfExporter();

exporter.setExporterInput(new SimpleExporterInput(jasperPrint));
exporter.setExporterOutput(
  new SimpleOutputStreamExporterOutput("employeeReport.pdf"));

SimplePdfReportConfiguration reportConfig
  = new SimplePdfReportConfiguration();
reportConfig.setSizePageToContent(true);
reportConfig.setForceLineBreakPolicy(false);

SimplePdfExporterConfiguration exportConfig
  = new SimplePdfExporterConfiguration();
exportConfig.setMetadataAuthor("baeldung");
exportConfig.setEncrypted(true);
exportConfig.setAllowedPermissionsHint("PRINTING");

exporter.setConfiguration(reportConfig);
exporter.setConfiguration(exportConfig);

exporter.exportReport();
```

### 5.2。XLS

```java
JRXlsxExporter exporter = new JRXlsxExporter();

// Set input and output ...
SimpleXlsxReportConfiguration reportConfig
  = new SimpleXlsxReportConfiguration();
reportConfig.setSheetNames(new String[] { "Employee Data" });

exporter.setConfiguration(reportConfig);
exporter.exportReport();
```

### 5.3。CSV

```java
JRCsvExporter exporter = new JRCsvExporter();

// Set input ...
exporter.setExporterOutput(
  new SimpleWriterExporterOutput("employeeReport.csv"));

exporter.exportReport();
```

### 5.4。HTML

```java
HtmlExporter exporter = new HtmlExporter();

// Set input ...
exporter.setExporterOutput(
  new SimpleHtmlExporterOutput("employeeReport.html"));

exporter.exportReport();
```

## 6。子报告

子报表只不过是嵌入在另一个报表中的标准报表。

首先，让我们创建一个报告来显示员工的电子邮件:

```java
<jasperReport ... >
    <parameter name="idEmployee" class="java.lang.Integer" />
    <queryString>
        <![CDATA[SELECT * FROM EMAIL WHERE ID_EMPLOYEE = $P{idEmployee}]]>
    </queryString>
    <field name="ADDRESS" class="java.lang.String"/>
    <detail>
        <band height="20" splitType="Stretch">
            <textField>
                <reportElement x="0" y="0" width="156" height="20"/>
                <textElement/>
                <textFieldExpression class="java.lang.String">
                  <![CDATA[$F{ADDRESS}]]></textFieldExpression>
            </textField>
        </band>
    </detail>
</jasperReport>
```

现在，让我们修改我们的员工报告，使其包含上一份报告:

```java
<detail>
    <band ... >
        <subreport>
            <reportElement x="0" y="20" width="300" height="27"/>
            <subreportParameter name="idEmployee">
                <subreportParameterExpression>
                  <![CDATA[$F{ID}]]></subreportParameterExpression>
            </subreportParameter>
            <connectionExpression>
              <![CDATA[$P{REPORT_CONNECTION}]]></connectionExpression>
            <subreportExpression class="java.lang.String">
              <![CDATA["employeeEmailReport.jasper"]]></subreportExpression>
        </subreport>
    </band>
</detail>
```

注意，我们通过编译文件的名称引用子报表，并将`idEmployee`和当前报表连接作为参数传递给它。

接下来，让我们编译这两个报告:

```java
InputStream employeeReportStream
  = getClass().getResourceAsStream("/employeeReport.jrxml");
JasperReport jasperReport
  = JasperCompileManager.compileReport(employeeReportStream);
JRSaver.saveObject(jasperReport, "employeeReport.jasper");

InputStream emailReportStream
  = getClass().getResourceAsStream("/employeeEmailReport.jrxml");
JRSaver.saveObject(
  JasperCompileManager.compileReport(emailReportStream),
  "employeeEmailReport.jasper");
```

我们填充和导出报告的代码不需要修改。

## 7 .**。结论**

在本文中，我们简要介绍了 JasperReports 库的核心特性。

我们能够用数据库中的记录编辑和填充报告；我们传递参数来根据不同的运行时条件改变报告中显示的数据，嵌入子报告并将它们导出为最常见的格式。

本文的完整源代码可以在 GitHub 上找到[。](https://web.archive.org/web/20220524060340/https://github.com/eugenp/tutorials/tree/master/libraries-2)