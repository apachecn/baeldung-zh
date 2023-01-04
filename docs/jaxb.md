# JAXB 指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/jaxb>

## 1。概述

这是一篇关于 JAXB(XML 绑定的 Java 架构)的入门教程。

首先，我们将展示如何将 Java 对象转换成 XML，反之亦然。

然后，我们将重点关注使用 JAXB-2 Maven 插件从 XML schema 生成 Java 类，反之亦然。

## 2。JAXB 简介

JAXB 提供了一种快速便捷的方法来将 Java 对象封送(写入)到 XML 中，并将 XML 解封送(读取)到对象中。它支持使用 Java 注释将 XML 元素和属性映射到 Java 字段和属性的绑定框架。

JAXB-2 Maven 插件将其大部分工作委托给 JDK 提供的两个工具 [XJC](https://web.archive.org/web/20220926190144/https://docs.oracle.com/javase/7/docs/technotes/tools/share/xjc.html) 和 [Schemagen](https://web.archive.org/web/20220926190144/https://docs.oracle.com/javase/7/docs/technotes/tools/share/schemagen.html) 中的一个。

## 3。JAXB 注释

JAXB 使用 Java 注释为生成的类增加附加信息。将这样的注释添加到现有的 Java 类中可以为 JAXB 运行时做好准备。

让我们首先创建一个简单的 Java 对象来说明编组和解组:

```
@XmlRootElement(name = "book")
@XmlType(propOrder = { "id", "name", "date" })
public class Book {
    private Long id;
    private String name;
    private String author;
    private Date date;

    @XmlAttribute
    public void setId(Long id) {
        this.id = id;
    }

    @XmlElement(name = "title")
    public void setName(String name) {
        this.name = name;
    }

    @XmlTransient
    public void setAuthor(String author) {
        this.author = author;
    }

    // constructor, getters and setters
}
```

上面的类包含这些注释:

*   **@ xmlroot element**:TXML 根元素的名字来源于类名，我们也可以用它的 name 属性来指定 XML 的根元素的名字。
*   **@XmlType** :定义字段写入 XML 文件的顺序
*   **@XmlElement** :定义将要使用的实际 XML 元素名称
*   **@XmlAttribute** : 定义 id 字段被映射为属性而不是元素
*   **@XmlTransient** : 注释我们不想包含在 XML 中的字段

关于 JAXB 注释的更多细节，请查看这个[链接](https://web.archive.org/web/20220926190144/https://docs.oracle.com/javaee/7/api/javax/xml/bind/annotation/package-summary.html)。

## 4。编组–将 Java 对象转换成 XML

编组使客户机应用程序能够将 JAXB 派生的 Java 对象树转换成 XML 数据。默认情况下，`Marshaller` 在生成 XML 数据时使用 UTF-8 编码。接下来，我们将从 Java 对象生成 XML 文件。

让我们使用`JAXBContext`创建一个简单的程序，它为管理实现 JAXB 绑定框架操作所必需的 XML/Java 绑定信息提供了一个抽象:

```
public void marshal() throws JAXBException, IOException {
    Book book = new Book();
    book.setId(1L);
    book.setName("Book1");
    book.setAuthor("Author1");
    book.setDate(new Date());

    JAXBContext context = JAXBContext.newInstance(Book.class);
    Marshaller mar= context.createMarshaller();
    mar.setProperty(Marshaller.JAXB_FORMATTED_OUTPUT, Boolean.TRUE);
    mar.marshal(book, new File("./book.xml"));
}
```

`javax.xml.bind.JAXBContext` 类提供了客户端到 JAXB API 的入口点。默认情况下，JAXB 不格式化 XML 文档。这可以节省空间，并防止任何空白被意外地解释为有意义的。

为了让 JAXB 格式化输出，我们只需设置*编组器。在*编组器*上将 JAXB_FORMATTED_OUTPUT* 属性设置为 *true* 。marshal 方法使用对象和输出文件将生成的 XML 存储为参数。

当我们运行上面的代码时，我们可以检查`book.xml`中的结果，以验证我们已经成功地将 Java 对象转换为 XML 数据:

```
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<book id="1">
    <title>Book1</title>
    <date>2016-11-12T11:25:12.227+07:00</date>
</book>
```

## 5。解组——将 XML 转换成 Java 对象

解组使客户机应用程序能够将 XML 数据转换成 JAXB 派生的 Java 对象。

让我们使用 JAXB `Unmarshaller`将我们的`book.xml`解组回一个 Java 对象:

```
public Book unmarshall() throws JAXBException, IOException {
    JAXBContext context = JAXBContext.newInstance(Book.class);
    return (Book) context.createUnmarshaller()
      .unmarshal(new FileReader("./book.xml"));
}
```

当我们运行上面的代码时，我们可以检查控制台输出，以验证我们已经成功地将 XML 数据转换为 Java 对象:

```
Book [id=1, name=Book1, author=null, date=Sat Nov 12 11:38:18 ICT 2016]
```

## 6。复杂数据类型

当处理 JAXB 中可能无法直接使用的复杂数据类型时，我们可以编写一个适配器来指示 JAXB 如何管理特定的类型。

为此，我们将使用 JAXB 的`XmlAdapter`来定义一个定制代码，将一个不可映射的类转换成 JAXB 可以处理的东西。`@XmlJavaTypeAdapter`注释使用一个适配器来扩展`XmlAdapter`类进行定制编组。

让我们创建一个适配器来指定编组时的日期格式:

```
public class DateAdapter extends XmlAdapter<String, Date> {

    private static final ThreadLocal<DateFormat> dateFormat 
      = new ThreadLocal<DateFormat>() {

        @Override
        protected DateFormat initialValue() {
            return new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
        }
    };

    @Override
    public Date unmarshal(String v) throws Exception {
        return dateFormat.get().parse(v);
    }

    @Override
    public String marshal(Date v) throws Exception {
        return dateFormat.get().format(v);
    }
}
```

我们使用日期格式`yyyy-MM-dd HH:mm:ss`在编组时将`Date`转换为`String` ，并使用`ThreadLocal`使`DateFormat`线程安全。

让我们将`DateAdapter`应用于我们的`Book`:

```
@XmlRootElement(name = "book")
@XmlType(propOrder = { "id", "name", "date" })
public class Book {
    private Long id;
    private String name;
    private String author;
    private Date date;

    @XmlAttribute
    public void setId(Long id) {
        this.id = id;
    }

    @XmlTransient
    public void setAuthor(String author) {
        this.author = author;
    }

    @XmlElement(name = "title")
    public void setName(String name) {
        this.name = name;
    }

    @XmlJavaTypeAdapter(DateAdapter.class)
    public void setDate(Date date) {
        this.date = date;
    }
}
```

当我们运行上面的代码时，我们可以检查`book.xml`中的结果，以验证我们已经使用新的日期格式`yyyy-MM-dd HH:mm:ss`成功地将 Java 对象转换为 XML:

```
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<book id="1">
    <title>Book1</title>
    <date>2016-11-10 23:44:18</date>final
</book>
```

## 7。JAXB-2 Maven 插件

这个插件使用 Java API for XML Binding (JAXB)版本 2+从 XML 模式(和可选的绑定文件)生成 Java 类，或者从带注释的 Java 类创建 XML 模式。

注意，有两种构建 web 服务的基本方法，`Contract Last`和`Contract First`。关于这些方法的更多细节，请查看这个[链接](https://web.archive.org/web/20220926190144/https://docs.spring.io/spring-ws/site/reference/html/why-contract-first.html)。

### 7.1。从 XSD 生成 Java 类

JAXB-2 Maven 插件使用 JDK 提供的工具 XJC，这是一个 JAXB 绑定编译器工具，它从 XSD (XML 模式定义)生成 Java 类。

让我们创建一个简单的`user.xsd`文件，并使用 JAXB-2 Maven 插件从这个 XSD 模式生成 Java 类:

```
<?xml version="1.0" encoding="UTF-8"?>
<schema 
    targetNamespace="/jaxb/gen"
    xmlns:userns="/jaxb/gen"
    elementFormDefault="qualified">

    <element name="userRequest" type="userns:UserRequest"></element>
    <element name="userResponse" type="userns:UserResponse"></element>

    <complexType name="UserRequest">
        <sequence>
            <element name="id" type="int" />
            <element name="name" type="string" />
        </sequence>
    </complexType>

    <complexType name="UserResponse">
        <sequence>
            <element name="id" type="int" />
            <element name="name" type="string" />
            <element name="gender" type="string" />
            <element name="created" type="dateTime" />
        </sequence>
    </complexType>
</schema>
```

让我们配置 JAXB-2 Maven 插件:

```
<plugin>
    <groupId>org.codehaus.mojo</groupId>
    <artifactId>jaxb2-maven-plugin</artifactId>
    <version>2.3</version>
    <executions>
        <execution>
            <id>xjc</id>
            <goals>
                <goal>xjc</goal>
            </goals>
        </execution>
    </executions>
    <configuration>
        <xjbSources>
            <xjbSource>src/main/resources/global.xjb</xjbSource>
        </xjbSources>
        <sources>
            <source>src/main/resources/user.xsd</source>
        </sources>
        <outputDirectory>${basedir}/src/main/java</outputDirectory>
        <clearOutputDir>false</clearOutputDir>
    </configuration>
</plugin>
```

默认情况下，这个插件在`src/main/xsd`中定位 XSD 文件。我们可以通过相应地修改`pom.xml`中这个插件的配置部分来配置 XSD 查找。

同样默认情况下，这些 Java 类是在`target/generated-resources/jaxb`文件夹中生成的。我们可以通过在插件配置中添加一个`outputDirectory` 元素来改变输出目录。我们还可以添加一个值为 false 的`clearOutputDir`元素来防止这个目录中的文件被删除。

此外，我们可以配置一个覆盖默认绑定规则的全局 JAXB 绑定:

```
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<jaxb:bindings version="2.0" xmlns:jaxb="http://java.sun.com/xml/ns/jaxb"
    xmlns:xjc="http://java.sun.com/xml/ns/jaxb/xjc"
    xmlns:xs="http://www.w3.org/2001/XMLSchema"
    jaxb:extensionBindingPrefixes="xjc">

    <jaxb:globalBindings>
        <xjc:simple />
        <xjc:serializable uid="-1" />
        <jaxb:javaType name="java.util.Calendar" xmlType="xs:dateTime"
            parse="javax.xml.bind.DatatypeConverter.parseDateTime"
            print="javax.xml.bind.DatatypeConverter.printDateTime" />
    </jaxb:globalBindings>
</jaxb:bindings>
```

上面的`global.xjb`将`dateTime`类型覆盖为`java.util.Calendar`类型。

当我们构建项目时，它会在`src/main/java`文件夹和包`com.baeldung.jaxb.gen`中生成类文件。

### 7.2。从 Java 生成 XSD 模式

同一插件使用 JDK 提供的工具`Schemagen`。这是一个 JAXB 绑定编译器工具，可以从 Java 类生成 XSD 模式。为了让一个 Java 类有资格成为 XSD 模式的候选，这个类必须用一个`@XmlType`注释进行注释。

我们将重用上一个示例中的 Java 类文件来配置插件:

```
<plugin>
    <groupId>org.codehaus.mojo</groupId>
    <artifactId>jaxb2-maven-plugin</artifactId>
    <version>2.3</version>
    <executions>
        <execution>
            <id>schemagen</id>
            <goals>
                <goal>schemagen</goal>
            </goals>
        </execution>
    </executions>
    <configuration>
        <sources>
            <source>src/main/java/com/baeldung/jaxb/gen</source>
        </sources>
        <outputDirectory>src/main/resources</outputDirectory>
        <clearOutputDir>false</clearOutputDir>
        <transformSchemas>
            <transformSchema>
                <uri>/jaxb/gen</uri>
                <toPrefix>user</toPrefix>
                <toFile>user-gen.xsd</toFile>
            </transformSchema>
        </transformSchemas>
    </configuration>
</plugin>
```

默认情况下，JAXB 递归扫描`src/main/java`下的所有文件夹，寻找带注释的 JAXB 类。通过在插件配置中添加一个`source`元素，我们可以为 JAXB 注释的类指定一个不同的`source`文件夹。

我们还可以注册一个`transformSchemas`，一个负责命名 XSD 模式的后处理器。它通过将`namespace`与我们的 Java 类的`@XmlType`的名称空间相匹配来工作。

当我们构建项目时，它会在`src/main/resources`目录中生成一个`user-gen.xsd`文件。

## 8。 **结论**

在本文中，我们讨论了 JAXB 的介绍性概念。更多细节，请看一下 [JAXB 主页](https://web.archive.org/web/20220926190144/http://www.oracle.com/technetwork/articles/javase/index-140168.html)。

我们可以在 GitHub 上找到这篇文章[的源代码。](https://web.archive.org/web/20220926190144/https://github.com/eugenp/tutorials/tree/master/jaxb)