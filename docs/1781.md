# Apache CXF Aegis 数据绑定简介

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/aegis-data-binding-in-apache-cxf>

## 1。概述

本教程介绍了 [Aegis](https://web.archive.org/web/20220117074026/https://cxf.apache.org/docs/aegis-21.html) 数据绑定，这是一个可以在 Java 对象和 XML 模式描述的 XML 文档之间进行映射的子系统。Aegis 允许对映射过程进行详细控制，同时将编程工作保持在最低限度。

Aegis 是 [Apache CXF](https://web.archive.org/web/20220117074026/https://cxf.apache.org/) 的一部分，但并不仅限于在这个框架中使用。相反，这种数据绑定机制可以在任何地方使用，因此在本教程中，我们把它作为一个独立的子系统来使用。

## 2。Maven 依赖关系

激活 Aegis 数据绑定所需的唯一依赖项是:

```
<dependency>
    <groupId>org.apache.cxf</groupId>
    <artifactId>cxf-rt-databinding-aegis</artifactId>
    <version>3.1.8</version>
</dependency>
```

这个产品的最新版本可以在[这里](https://web.archive.org/web/20220117074026/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22org.apache.cxf%22%20AND%20a%3A%22cxf-rt-databinding-aegis%22)找到。

## 3。类型定义

本节将介绍用于说明 Aegis 的三种类型的定义。

### 3.1。`Course`

这是我们示例中最简单的类，定义如下:

```
public class Course {
    private int id;
    private String name;
    private String instructor;
    private Date enrolmentDate;

    // standard getters and setters
}
```

### 3.2。`CourseRepo`

`CourseRepo`是我们模型中的顶级类型。我们将它定义为一个接口，而不是一个类，以展示编组 Java 接口是多么容易，这在没有自定义适配器的 JAXB 中是不可能的:

```
public interface CourseRepo {
    String getGreeting();
    void setGreeting(String greeting);
    Map<Integer, Course> getCourses();
    void setCourses(Map<Integer, Course> courses);
    void addCourse(Course course);  
}
```

注意，我们用返回类型`Map`声明了`getCourses`方法。这是有意表达 Aegis 相对于 JAXB 的另一个优势。后者不能在没有自定义适配器的情况下封送映射，而前者可以。

### 3.3。`CourseRepoImpl`

这个类提供了对`CourseRepo`接口的实现:

```
public class CourseRepoImpl implements CourseRepo {
    private String greeting;
    private Map<Integer, Course> courses = new HashMap<>();

    // standard getters and setters

    @Override
    public void addCourse(Course course) {
        courses.put(course.getId(), course);
    }
}
```

## 4。自定义数据绑定

为了使定制生效，XML 映射文件必须存在于类路径中。要求将这些文件放在一个目录中，该目录的结构对应于相关 Java 类型的包层次结构。

例如，如果一个完全限定的名称类被命名为`package.ClassName`，那么它的相关映射文件必须位于类路径上的`package/ClassName`子目录中。映射文件的名称必须等于关联的 Java 类型，并在其后附加一个`.aegis.xml`后缀。

### 4.1。`CourseRepo` 制图

`CourseRepo`接口属于`com.baeldung.cxf.aegis`包，所以它对应的映射文件被命名为`CourseRepo.aegis.xml`，放在类路径上的`com/baeldung/cxf/aegis`目录中。

在`CourseRepo`映射文件中，我们更改了与`CourseRepo`接口相关联的 XML 元素的名称和命名空间，以及它的`greeting`属性的样式:

```
<mappings xmlns:ns="http://courserepo.baeldung.com">
    <mapping name="ns:Baeldung">
        <property name="greeting" style="attribute"/>
    </mapping>
</mappings>
```

### 4.2。路线映射

类似于`CourseRepo`类型，`Course`类的映射文件被命名为`Course.aegis.xml`，也位于`com/baeldung/cxf/aegis`目录中。

在这个映射文件中，我们指示 Aegis 在封送处理时忽略`Course`类的`instructor`属性，这样它的值在从输出 XML 文档重新创建的对象中不可用:

```
<mappings>
    <mapping>
        <property name="instructor" ignore="true"/>
    </mapping>
</mappings>
```

在 Aegis 的主页上，我们可以找到更多的定制选项。

## 5。测试

本节是一个分步指南，介绍如何设置和执行一个测试用例，演示 Aegis 数据绑定的用法。

为了方便测试过程，我们在测试类中声明了两个字段:

```
public class BaeldungTest {
    private AegisContext context;
    private String fileName = "baeldung.xml";

    // other methods
}
```

这些字段在这里被定义为由该类的其他方法使用。

### 5.1。`AegisContext`初始化

首先，必须创建一个`AegisContext`对象:

```
context = new AegisContext();
```

然后配置并初始化那个`AegisContext`实例。下面是我们如何为上下文设置根类:

```
Set<Type> rootClasses = new HashSet<Type>();
rootClasses.add(CourseRepo.class);
context.setRootClasses(rootClasses);
```

Aegis 为`Set<Type>`对象中的每个`Type`创建一个 XML 映射元素。在本教程中，我们只设置`CourseRepo`作为根类型。

现在，让我们为上下文设置实现映射，为`CourseRepo`接口指定代理类:

```
Map<Class<?>, String> beanImplementationMap = new HashMap<>();
beanImplementationMap.put(CourseRepoImpl.class, "CourseRepo");
context.setBeanImplementationMap(beanImplementationMap);
```

Aegis 上下文的最后一个配置告诉它在相应的 XML 文档中设置`xsi:type`属性。除非被映射文件覆盖，否则该属性携带关联 Java 对象的实际类型名:

```
context.setWriteXsiTypes(true);
```

我们的`AegisContext`实例现在可以初始化了:

```
context.initialize();
```

为了保持代码的整洁，我们将本小节中的所有代码片段收集到一个帮助器方法中:

```
private void initializeContext() {
    // ...
}
```

### 5.2。简单的数据设置

由于本教程的简单性质，我们在内存中生成样本数据，而不是依赖于一个持久的解决方案。让我们使用下面的设置逻辑来填充课程报告:

```
private CourseRepoImpl initCourseRepo() {
    Course restCourse = new Course();
    restCourse.setId(1);
    restCourse.setName("REST with Spring");
    restCourse.setInstructor("Eugen");
    restCourse.setEnrolmentDate(new Date(1234567890000L));

    Course securityCourse = new Course();
    securityCourse.setId(2);
    securityCourse.setName("Learn Spring Security");
    securityCourse.setInstructor("Eugen");
    securityCourse.setEnrolmentDate(new Date(1456789000000L));

    CourseRepoImpl courseRepo = new CourseRepoImpl();
    courseRepo.setGreeting("Welcome to Beldung!");
    courseRepo.addCourse(restCourse);
    courseRepo.addCourse(securityCourse);
    return courseRepo;
}
```

### 5.3。绑定 Java 对象和 XML 元素

下面的 helper 方法说明了将 Java 对象封送到 XML 元素所需的步骤:

```
private void marshalCourseRepo(CourseRepo courseRepo) throws Exception {
    AegisWriter<XMLStreamWriter> writer = context.createXMLStreamWriter();
    AegisType aegisType = context.getTypeMapping().getType(CourseRepo.class);
    XMLStreamWriter xmlWriter = XMLOutputFactory.newInstance()
      .createXMLStreamWriter(new FileOutputStream(fileName));

    writer.write(courseRepo, 
      new QName("http://aegis.cxf.baeldung.com", "baeldung"), false, xmlWriter, aegisType);

    xmlWriter.close();
}
```

正如我们所见，`AegisWriter`和`AegisType`对象必须从`AegisContext`实例中创建。然后，`AegisWriter`对象将给定的 Java 实例编组到指定的输出。

在这种情况下，这是一个与以文件系统中的`fileName`类级别字段的值命名的文件相关联的`XMLStreamWriter`对象。

以下方法将 XML 文档解组为给定类型的 Java 对象:

```
private CourseRepo unmarshalCourseRepo() throws Exception {       
    AegisReader<XMLStreamReader> reader = context.createXMLStreamReader();
    XMLStreamReader xmlReader = XMLInputFactory.newInstance()
      .createXMLStreamReader(new FileInputStream(fileName));

    CourseRepo courseRepo = (CourseRepo) reader.read(
      xmlReader, context.getTypeMapping().getType(CourseRepo.class));

    xmlReader.close();
    return courseRepo;
}
```

这里，从`AegisContext`实例生成一个`AegisReader`对象。然后,`AegisReader`对象用提供的输入创建一个 Java 对象。在这个例子中，输入是一个由我们在上面描述的`marshalCourseRepo`方法中生成的文件支持的`XMLStreamReader`对象。

### 5.4。断言

现在，是时候将前面小节中定义的所有助手方法组合成一个测试方法了:

```
@Test
public void whenMarshalingAndUnmarshalingCourseRepo_thenCorrect()
  throws Exception {
    initializeContext();
    CourseRepo inputRepo = initCourseRepo();
    marshalCourseRepo(inputRepo);
    CourseRepo outputRepo = unmarshalCourseRepo();
    Course restCourse = outputRepo.getCourses().get(1);
    Course securityCourse = outputRepo.getCourses().get(2);

    // JUnit assertions
}
```

我们首先创建一个`CourseRepo`实例，然后将其编组为一个 XML 文档，最后解组该文档以重新创建原始对象。让我们验证重新创建的对象是我们所期望的:

```
assertEquals("Welcome to Beldung!", outputRepo.getGreeting());
assertEquals("REST with Spring", restCourse.getName());
assertEquals(new Date(1234567890000L), restCourse.getEnrolmentDate());
assertNull(restCourse.getInstructor());
assertEquals("Learn Spring Security", securityCourse.getName());
assertEquals(new Date(1456789000000L), securityCourse.getEnrolmentDate());
assertNull(securityCourse.getInstructor());
```

很明显，除了`instructor`属性，所有其他属性的值都被恢复，包括值为`Date`类型的`enrolmentDate`属性。这正是我们所期望的，因为我们已经指示 Aegis 在编组`Course`对象时忽略`instructor`属性。

### 5.5。输出 XML 文档

为了明确 Aegis 映射文件的效果，我们在下面显示了没有定制的 XML 文档:

```
<ns1:baeldung xmlns:ns1="http://aegis.cxf.baeldung.com"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:type="ns1:CourseRepo">
    <ns1:courses>
        <ns2:entry xmlns:ns2="urn:org.apache.cxf.aegis.types">
            <ns2:key>1</ns2:key>
            <ns2:value xsi:type="ns1:Course">
                <ns1:enrolmentDate>2009-02-14T06:31:30+07:00
                </ns1:enrolmentDate>
                <ns1:id>1</ns1:id>
                <ns1:instructor>Eugen</ns1:instructor>
                <ns1:name>REST with Spring</ns1:name>
            </ns2:value>
        </ns2:entry>
        <ns2:entry xmlns:ns2="urn:org.apache.cxf.aegis.types">
            <ns2:key>2</ns2:key>
            <ns2:value xsi:type="ns1:Course">
                <ns1:enrolmentDate>2016-03-01T06:36:40+07:00
                </ns1:enrolmentDate>
                <ns1:id>2</ns1:id>
                <ns1:instructor>Eugen</ns1:instructor>
                <ns1:name>Learn Spring Security</ns1:name>
            </ns2:value>
        </ns2:entry>
    </ns1:courses>
    <ns1:greeting>Welcome to Beldung!</ns1:greeting>
</ns1:baeldung>
```

将此与 Aegis 自定义映射的情况进行比较:

```
<ns1:baeldung xmlns:ns1="http://aegis.cxf.baeldung.com"
    xmlns:ns="http://courserepo.baeldung.com"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:type="ns:Baeldung" greeting="Welcome to Beldung!">
    <ns:courses>
        <ns2:entry xmlns:ns2="urn:org.apache.cxf.aegis.types">
            <ns2:key>1</ns2:key>
            <ns2:value xsi:type="ns1:Course">
                <ns1:enrolmentDate>2009-02-14T06:31:30+07:00
                </ns1:enrolmentDate>
                <ns1:id>1</ns1:id>
                <ns1:name>REST with Spring</ns1:name>
            </ns2:value>
        </ns2:entry>
        <ns2:entry xmlns:ns2="urn:org.apache.cxf.aegis.types">
            <ns2:key>2</ns2:key>
            <ns2:value xsi:type="ns1:Course">
                <ns1:enrolmentDate>2016-03-01T06:36:40+07:00
                </ns1:enrolmentDate>
                <ns1:id>2</ns1:id>
                <ns1:name>Learn Spring Security</ns1:name>
            </ns2:value>
        </ns2:entry>
    </ns:courses>
</ns1:baeldung>
```

在运行本节中定义的测试之后，您可能会在项目主目录中的`baeldung.xml`中找到这个 XML 结构。

您将看到对应于`CourseRepo`对象的 XML 元素的`type`属性和名称空间根据我们在`CourseRepo.aegis.xml`文件中的设置而改变。`greeting`属性也被转换为属性，`Course`对象的`instructor`属性如预期的那样消失了。

值得注意的是，默认情况下，Aegis 将基本 Java 类型转换为最匹配的模式类型，例如，从`Date`对象转换为`xsd:dateTime`元素，如本教程所示。但是，我们可以通过在相应的映射文件中设置配置来更改特定的绑定。

如果您想了解更多信息，请导航至 [Aegis 主页](https://web.archive.org/web/20220117074026/https://cxf.apache.org/docs/aegis-21.html)。

## 6。结论

本教程演示了 Apache CXF Aegis 数据绑定作为独立子系统的使用。它演示了如何使用 Aegis 将 Java 对象映射到 XML 元素，反之亦然。

本教程还关注如何定制数据绑定行为。

和往常一样，所有这些例子和代码片段的实现都可以在 GitHub 项目中找到。