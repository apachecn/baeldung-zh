# XMLUnit 2.x 简介

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/xmlunit2>

## 1。概述

XMLUnit 2.x 是一个强大的库，可以帮助我们测试和验证 XML 内容，当我们确切地知道 XML 应该包含什么内容时，这个库会非常方便。

因此，我们将主要在单元测试**中使用 XMLUnit 来验证我们所拥有的是有效的 XML** ，它包含某些信息或者符合某种风格的文档。

此外，使用 XMLUnit，**我们可以控制哪种差异对我们来说是重要的**，以及将样式引用的哪一部分与比较 XML 的哪一部分进行比较。

因为我们关注的是 XMLUnit 2.x 而不是 XMLUnit 1.x，所以每当我们使用 XMLUnit 这个词时，我们严格地指的是 2.x。

最后，我们还将使用 Hamcrest 匹配器来匹配断言，所以如果你不熟悉的话，最好温习一下 [Hamcrest](/web/20220926010849/https://www.baeldung.com/java-junit-hamcrest-guide) 。

## 2。XMLUnit Maven 设置

为了在我们的 maven 项目中使用这个库，我们需要在`pom.xml`中有以下依赖项:

```
<dependency>
    <groupId>org.xmlunit</groupId>
    <artifactId>xmlunit-core</artifactId>
    <version>2.2.1</version>
</dependency>
```

最新版本的 `xmlunit-core`可以通过关注[这个链接](https://web.archive.org/web/20220926010849/https://search.maven.org/classic/#search%7Cga%7C1%7Ca%3A%22xmlunit-core%22)找到。并且:

```
<dependency>
    <groupId>org.xmlunit</groupId>
    <artifactId>xmlunit-matchers</artifactId>
    <version>2.2.1</version>
</dependency>
```

最新版本的`xmlunit-matchers`可在[此链接](https://web.archive.org/web/20220926010849/https://search.maven.org/classic/#search%7Cga%7C1%7Ca%3A%22xmlunit-matchers%22)获得。

## 3。比较 XML

### 3.1。简单差异示例

假设我们有两段 XML。当文档中节点的内容和顺序完全相同时，它们被认为是相同的，因此下面的测试将通过:

```
@Test
public void given2XMLS_whenIdentical_thenCorrect() {
    String controlXml = "<struct><int>3</int><boolean>false</boolean></struct>";
    String testXml = "<struct><int>3</int><boolean>false</boolean></struct>";
    assertThat(testXml, CompareMatcher.isIdenticalTo(controlXml));
}
```

下一个测试失败了，因为两段 XML 相似但不相同，因为它们的**节点出现在不同的序列**:

```
@Test
public void given2XMLSWithSimilarNodesButDifferentSequence_whenNotIdentical_thenCorrect() {
    String controlXml = "<struct><int>3</int><boolean>false</boolean></struct>";
    String testXml = "<struct><boolean>false</boolean><int>3</int></struct>";
    assertThat(testXml, assertThat(testXml, not(isIdenticalTo(controlXml)));
}
```

### 3.2。详细差异示例

上面两个 XML 文档之间的差异由**差异引擎**检测。

默认情况下，出于效率原因，一旦发现第一个差异，它就停止比较过程。

为了获得两段 XML 之间的所有差异，我们使用了一个类似于下面这样的`Diff` 类的实例:

```
@Test
public void given2XMLS_whenGeneratesDifferences_thenCorrect(){
    String controlXml = "<struct><int>3</int><boolean>false</boolean></struct>";
    String testXml = "<struct><boolean>false</boolean><int>3</int></struct>";
    Diff myDiff = DiffBuilder.compare(controlXml).withTest(testXml).build();

    Iterator<Difference> iter = myDiff.getDifferences().iterator();
    int size = 0;
    while (iter.hasNext()) {
        iter.next().toString();
        size++;
    }
    assertThat(size, greaterThan(1));
}
```

如果我们打印在`while`循环中返回的值，结果如下:

```
Expected element tag name 'int' but was 'boolean' - 
  comparing <int...> at /struct[1]/int[1] to <boolean...> 
    at /struct[1]/boolean[1] (DIFFERENT)
Expected text value '3' but was 'false' - 
  comparing <int ...>3</int> at /struct[1]/int[1]/text()[1] to 
    <boolean ...>false</boolean> at /struct[1]/boolean[1]/text()[1] (DIFFERENT)
Expected element tag name 'boolean' but was 'int' - 
  comparing <boolean...> at /struct[1]/boolean[1] 
    to <int...> at /struct[1]/int[1] (DIFFERENT)
Expected text value 'false' but was '3' - 
  comparing <boolean ...>false</boolean> at /struct[1]/boolean[1]/text()[1] 
    to <int ...>3</int> at /struct[1]/int[1]/text()[1] (DIFFERENT)
```

每个实例描述了控制节点和测试节点之间的差异类型以及这些节点的详细信息(包括每个节点的 XPath 位置)。

如果我们想在发现第一个差异后强制差异引擎**停止，并且不继续枚举进一步的差异——我们需要提供一个`ComparisonController`:**

```
@Test
public void given2XMLS_whenGeneratesOneDifference_thenCorrect(){
    String myControlXML = "<struct><int>3</int><boolean>false</boolean></struct>";
    String myTestXML = "<struct><boolean>false</boolean><int>3</int></struct>";

    Diff myDiff = DiffBuilder
      .compare(myControlXML)
      .withTest(myTestXML)
      .withComparisonController(ComparisonControllers.StopWhenDifferent)
       .build();

    Iterator<Difference> iter = myDiff.getDifferences().iterator();
    int size = 0;
    while (iter.hasNext()) {
        iter.next().toString();
        size++;
    }
    assertThat(size, equalTo(1));
}
```

**差异信息更简单:**

```
Expected element tag name 'int' but was 'boolean' - 
  comparing <int...> at /struct[1]/int[1] 
    to <boolean...> at /struct[1]/boolean[1] (DIFFERENT)
```

## 4。输入源

有了 XMLUnit `,`,我们可以从各种来源中挑选 XML 数据，这对于我们的应用程序的需求来说可能是方便的。在这种情况下，我们使用带有静态方法数组的`Input`类。

为了从位于项目根的 XML 文件中选取输入，我们执行以下操作:

```
@Test
public void givenFileSource_whenAbleToInput_thenCorrect() {
    ClassLoader classLoader = getClass().getClassLoader();
    String testPath = classLoader.getResource("test.xml").getPath();
    String controlPath = classLoader.getResource("control.xml").getPath();

    assertThat(
      Input.fromFile(testPath), isSimilarTo(Input.fromFile(controlPath)));
}
```

从 XML 字符串中选择输入源，如下所示:

```
@Test
public void givenStringSource_whenAbleToInput_thenCorrect() {
    String controlXml = "<struct><int>3</int><boolean>false</boolean></struct>";
    String testXml = "<struct><int>3</int><boolean>false</boolean></struct>";

    assertThat(
      Input.fromString(testXml),isSimilarTo(Input.fromString(controlXml)));
}
```

现在让我们使用一个流作为输入:

```
@Test
public void givenStreamAsSource_whenAbleToInput_thenCorrect() {
    assertThat(Input.fromStream(XMLUnitTests.class
      .getResourceAsStream("/test.xml")),
        isSimilarTo(
          Input.fromStream(XMLUnitTests.class
            .getResourceAsStream("/control.xml"))));
}
```

我们也可以使用`Input.from(Object)`,在这里我们传递任何要由 XMLUnit 解析的有效源。

例如，我们可以传入一个文件:

```
@Test
public void givenFileSourceAsObject_whenAbleToInput_thenCorrect() {
    ClassLoader classLoader = getClass().getClassLoader();

    assertThat(
      Input.from(new File(classLoader.getResource("test.xml").getFile())), 
      isSimilarTo(Input.from(new File(classLoader.getResource("control.xml").getFile()))));
}
```

还是一个`String:`

```
@Test
public void givenStringSourceAsObject_whenAbleToInput_thenCorrect() {
    assertThat(
      Input.from("<struct><int>3</int><boolean>false</boolean></struct>"),
      isSimilarTo(Input.from("<struct><int>3</int><boolean>false</boolean></struct>")));
}
```

还是一个`Stream:`

```
@Test
public void givenStreamAsObject_whenAbleToInput_thenCorrect() {
    assertThat(
      Input.from(XMLUnitTest.class.getResourceAsStream("/test.xml")), 
      isSimilarTo(Input.from(XMLUnitTest.class.getResourceAsStream("/control.xml"))));
}
```

这些问题都会得到解决。

## 5。比较特定节点

在上面的第 2 节中，我们只查看了相同的 XML，因为相似的 XML 需要使用来自`xmlunit-core`库的特性进行一点点定制:

```
@Test
public void given2XMLS_whenSimilar_thenCorrect() {
    String controlXml = "<struct><int>3</int><boolean>false</boolean></struct>";
    String testXml = "<struct><boolean>false</boolean><int>3</int></struct>";

    assertThat(testXml, isSimilarTo(controlXml));
}
```

上面的测试应该通过，因为 XML 有相似的节点，但是它失败了。这是因为 **XMLUnit 在相对于根节点**的相同深度比较控制和测试节点。

所以测试一个`isSimilarTo`条件比测试一个`isIdenticalTo`条件更有趣一点。`controlXml`中的节点`<int>3</int>`将与`testXml`中的`<boolean>false</boolean>`进行比较，自动给出故障信息:

```
java.lang.AssertionError: 
Expected: Expected element tag name 'int' but was 'boolean' - 
  comparing <int...> at /struct[1]/int[1] to <boolean...> at /struct[1]/boolean[1]:
<int>3</int>
   but: result was: 
<boolean>false</boolean>
```

这就是 XMLUnit 的`DefaultNodeMatcher`和`ElementSelector`类派上用场的地方

XMLUnit 在比较阶段查询 `DefaultNodeMatcher`类，因为它在`controlXml,`的节点上循环，以确定将来自`testXml`的哪个 XML 节点与它在`controlXml`中遇到的当前 XML 节点进行比较。

在此之前，`DefaultNodeMatcher`将已经咨询过`ElementSelector`来决定如何匹配节点。

我们的测试失败了，因为在默认状态下，XMLUnit 将使用深度优先的方法来遍历 XML，并基于文档顺序来匹配节点，因此`<int>`与`<boolean>`匹配。

让我们调整我们的测试，让它通过:

```
@Test
public void given2XMLS_whenSimilar_thenCorrect() {
    String controlXml = "<struct><int>3</int><boolean>false</boolean></struct>";
    String testXml = "<struct><boolean>false</boolean><int>3</int></struct>";

    assertThat(testXml, 
      isSimilarTo(controlXml).withNodeMatcher(
      new DefaultNodeMatcher(ElementSelectors.byName)));
}
```

在这种情况下，我们告诉`DefaultNodeMatcher`当 XMLUnit 要求比较一个节点时，您应该已经按照元素名称对节点进行了排序和匹配。

最初失败的例子类似于将`ElementSelectors.Default`传递给`DefaultNodeMatcher`。

或者，我们可以使用来自`xmlunit-core`的`Diff`，而不是使用`xmlunit-matchers`:

```
@Test
public void given2XMLs_whenSimilarWithDiff_thenCorrect() throws Exception {
    String myControlXML = "<struct><int>3</int><boolean>false</boolean></struct>";
    String myTestXML = "<struct><boolean>false</boolean><int>3</int></struct>";
    Diff myDiffSimilar = DiffBuilder.compare(myControlXML).withTest(myTestXML)
      .withNodeMatcher(new DefaultNodeMatcher(ElementSelectors.byName))
      .checkForSimilar().build();

    assertFalse("XML similar " + myDiffSimilar.toString(),
      myDiffSimilar.hasDifferences());
}
```

## 6。`DifferenceEvaluator`风俗

A `DifferenceEvaluator` 确定比较的结果。它的作用仅限于确定比较结果的严重性。

是这个类决定了两个 XML 片段是`identical`、`similar`还是`different`。

考虑以下 XML 片段:

```
<a>
    <b attr="abc">
    </b>
</a>
```

并且:

```
<a>
    <b attr="xyz">
    </b>
</a>
```

在默认状态下，它们在技术上被认为是不同的，因为它们的`attr`属性具有不同的值。让我们来看一个测试:

```
@Test
public void given2XMLsWithDifferences_whenTestsDifferentWithoutDifferenceEvaluator_thenCorrect(){
    final String control = "<a><b attr=\"abc\"></b></a>";
    final String test = "<a><b attr=\"xyz\"></b></a>";
    Diff myDiff = DiffBuilder.compare(control).withTest(test)
      .checkForSimilar().build();
    assertFalse(myDiff.toString(), myDiff.hasDifferences());
}
```

失败消息:

```
java.lang.AssertionError: Expected attribute value 'abc' but was 'xyz' - 
  comparing <b attr="abc"...> at /a[1]/b[1]/@attr 
  to <b attr="xyz"...> at /a[1]/b[1]/@attr
```

如果我们真的不关心属性，我们可以改变`DifferenceEvaluator`的行为来忽略它。为此，我们创建了自己的:

```
public class IgnoreAttributeDifferenceEvaluator implements DifferenceEvaluator {
    private String attributeName;
    public IgnoreAttributeDifferenceEvaluator(String attributeName) {
        this.attributeName = attributeName;
    }

    @Override
    public ComparisonResult evaluate(Comparison comparison, ComparisonResult outcome) {
        if (outcome == ComparisonResult.EQUAL)
            return outcome;
        final Node controlNode = comparison.getControlDetails().getTarget();
        if (controlNode instanceof Attr) {
            Attr attr = (Attr) controlNode;
            if (attr.getName().equals(attributeName)) {
                return ComparisonResult.SIMILAR;
            }
        }
        return outcome;
    }
}
```

然后，我们重写最初失败的测试，并提供我们自己的`DifferenceEvaluator`实例，就像这样:

```
@Test
public void given2XMLsWithDifferences_whenTestsSimilarWithDifferenceEvaluator_thenCorrect() {
    final String control = "<a><b attr=\"abc\"></b></a>";
    final String test = "<a><b attr=\"xyz\"></b></a>";
    Diff myDiff = DiffBuilder.compare(control).withTest(test)
      .withDifferenceEvaluator(new IgnoreAttributeDifferenceEvaluator("attr"))
      .checkForSimilar().build();

    assertFalse(myDiff.toString(), myDiff.hasDifferences());
}
```

这次它过去了。

## 7。验证

XMLUnit 使用`Validator`类执行 XML 验证。您使用`forLanguage`工厂方法创建它的一个实例，同时传入用于验证的模式。

模式作为指向其位置的 URI 传入，XMLUnit 将它在`Languages`类中支持的模式位置抽象为常量。

我们通常像这样创建一个`Validator`类的实例:

```
Validator v = Validator.forLanguage(Languages.W3C_XML_SCHEMA_NS_URI);
```

在这一步之后，如果我们有自己的 XSD 文件来验证我们的 XML，我们只需指定它的源，然后用我们的 XML 文件源调用`Validator`的`validateInstance`方法。

以我们的`students.xsd`为例:

```
<?xml version = "1.0"?>
<xs:schema xmlns:xs="http://www.w3.org/2001/XMLSchema">
    <xs:element name='class'>
        <xs:complexType>
            <xs:sequence>
                <xs:element name='student' type='StudentObject'
                   minOccurs='0' maxOccurs='unbounded' />
            </xs:sequence>
        </xs:complexType>
    </xs:element>
    <xs:complexType name="StudentObject">
        <xs:sequence>
            <xs:element name="name" type="xs:string" />
            <xs:element name="age" type="xs:positiveInteger" />
        </xs:sequence>
        <xs:attribute name='id' type='xs:positiveInteger' />
    </xs:complexType>
</xs:schema>
```

和`students.xml`:

```
<?xml version = "1.0"?>
<class>
    <student id="393">
        <name>Rajiv</name>
        <age>18</age>
    </student>
    <student id="493">
        <name>Candie</name>
        <age>19</age>
    </student>
</class>
```

接下来让我们进行一个测试:

```
@Test
public void givenXml_whenValidatesAgainstXsd_thenCorrect() {
    Validator v = Validator.forLanguage(Languages.W3C_XML_SCHEMA_NS_URI);
    v.setSchemaSource(Input.fromStream(
      XMLUnitTests.class.getResourceAsStream("/students.xsd")).build());
    ValidationResult r = v.validateInstance(Input.fromStream(
      XMLUnitTests.class.getResourceAsStream("/students.xml")).build());
    Iterator<ValidationProblem> probs = r.getProblems().iterator();
    while (probs.hasNext()) {
        probs.next().toString();
    }
    assertTrue(r.isValid());
}
```

验证的结果是一个`ValidationResult`实例，它包含一个布尔标志，指示文档是否已经被成功验证。

`ValidationResult`还包含一个带有`ValidationProblem` s 的`Iterable`，以防出现故障。让我们创建一个新的包含错误的 XML，名为`students_with_error.xml.` 而不是`<student>`，我们的开始标签都是`</studet>`:

```
<?xml version = "1.0"?>
<class>
    <studet id="393">
        <name>Rajiv</name>
        <age>18</age>
    </student>
    <studet id="493">
        <name>Candie</name>
        <age>19</age>
    </student>
</class>
```

然后对其运行以下测试:

```
@Test
public void givenXmlWithErrors_whenReturnsValidationProblems_thenCorrect() {
    Validator v = Validator.forLanguage(Languages.W3C_XML_SCHEMA_NS_URI);
    v.setSchemaSource(Input.fromStream(
       XMLUnitTests.class.getResourceAsStream("/students.xsd")).build());
    ValidationResult r = v.validateInstance(Input.fromStream(
      XMLUnitTests.class.getResourceAsStream("/students_with_error.xml")).build());
    Iterator<ValidationProblem> probs = r.getProblems().iterator();
    int count = 0;
    while (probs.hasNext()) {
        count++;
        probs.next().toString();
    }
    assertTrue(count > 0);
}
```

如果我们打印出`while`循环中的错误，它们看起来会像:

```
ValidationProblem { line=3, column=19, type=ERROR,message='cvc-complex-type.2.4.a: 
  Invalid content was found starting with element 'studet'. 
    One of '{student}' is expected.' }
ValidationProblem { line=6, column=4, type=ERROR, message='The element type "studet" 
  must be terminated by the matching end-tag "</studet>".' }
ValidationProblem { line=6, column=4, type=ERROR, message='The element type "studet" 
  must be terminated by the matching end-tag "</studet>".' }
```

## 8。XPath

当一个 XPath 表达式根据一段 XML 进行计算时，会创建一个包含匹配的`Nodes.`的`NodeList`

考虑保存在名为`teachers.xml`的文件中的这段 XML:

```
<teachers>
    <teacher department="science" id='309'>
        <subject>math</subject>
        <subject>physics</subject>
    </teacher>
    <teacher department="arts" id='310'>
        <subject>political education</subject>
        <subject>english</subject>
    </teacher>
</teachers>
```

XMLUnit 提供了许多与 XPath 相关的断言方法，如下所示。

我们可以检索所有名为`teacher` 的节点，并分别对它们执行断言:

```
@Test
public void givenXPath_whenAbleToRetrieveNodes_thenCorrect() {
    Iterable<Node> i = new JAXPXPathEngine()
      .selectNodes("//teacher", Input.fromFile(new File("teachers.xml")).build());
    assertNotNull(i);
    int count = 0;
    for (Iterator<Node> it = i.iterator(); it.hasNext();) {
        count++;
        Node node = it.next();
        assertEquals("teacher", node.getNodeName());

        NamedNodeMap map = node.getAttributes();
        assertEquals("department", map.item(0).getNodeName());
        assertEquals("id", map.item(1).getNodeName());
        assertEquals("teacher", node.getNodeName());
    }
    assertEquals(2, count);
}
```

请注意我们如何验证子节点的数量、每个节点的名称以及每个节点中的属性。取回`Node`后，更多选项可用。

要验证路径是否存在，我们可以执行以下操作:

```
@Test
public void givenXmlSource_whenAbleToValidateExistingXPath_thenCorrect() {
    assertThat(Input.fromFile(new File("teachers.xml")), hasXPath("//teachers"));
    assertThat(Input.fromFile(new File("teachers.xml")), hasXPath("//teacher"));
    assertThat(Input.fromFile(new File("teachers.xml")), hasXPath("//subject"));
    assertThat(Input.fromFile(new File("teachers.xml")), hasXPath("//@department"));
}
```

要验证路径不存在，我们可以这样做:

```
@Test
public void givenXmlSource_whenFailsToValidateInExistentXPath_thenCorrect() {
    assertThat(Input.fromFile(new File("teachers.xml")), not(hasXPath("//sujet")));
}
```

当文档主要由已知的、不变的内容组成，只有少量由系统创建的变化内容时，XPaths 特别有用。

## 9。结论

在本教程中，我们介绍了`XMLUnit 2.x`的大部分基本特性，以及如何在应用程序中使用它们来验证 XML 文档。

所有这些例子和代码片段的完整实现可以在 GitHub 项目的 `XMLUnit` [中找到。](https://web.archive.org/web/20220926010849/https://github.com/eugenp/tutorials/tree/master/testing-modules/xmlunit-2)