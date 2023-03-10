# 根据 XSD 文件验证 XML 文件

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-validate-xml-xsd>

## 1.概观

在本教程中，我们将演示如何根据 XSD 文件验证一个 [XML](/web/20220830095532/https://www.baeldung.com/java-xml) 文件。

## 2.一个 XML 和两个 XSD 文件的定义

让我们考虑下面的 XML 文件`baeldung.xml`，它包含一个名称和一个地址，地址本身由邮政编码和城市组成:

```java
<?xml version="1.0" encoding="UTF-8" ?>
<individual>
    <name>Baeldung</name>
    <address>
        <zip>00001</zip>
        <city>New York</city>
    </address>
</individual>
```

`baeldung.xml`的内容与`person.xsd` 文件的描述完全匹配:

```java
<?xml version="1.0" encoding="UTF-8" ?>
<xs:schema xmlns:xs="http://www.w3.org/2001/XMLSchema">
    <xs:element name="individual">
        <xs:complexType>
            <xs:sequence>
                <xs:element name="name" type="xs:string" />
                <xs:element name="address">
                    <xs:complexType>
                        <xs:sequence>
                            <xs:element name="zip" type="xs:positiveInteger" />
                            <xs:element name="city" type="xs:string" />
                        </xs:sequence>
                    </xs:complexType>
                </xs:element>
            </xs:sequence>
        </xs:complexType>
    </xs:element>
</xs:schema>
```

然而，我们的 XML 对于下面的 XSD 文件`full-person.xsd`是无效的:

```java
<?xml version="1.0" encoding="UTF-8" ?>
<xs:schema xmlns:xs="http://www.w3.org/2001/XMLSchema">
    <xs:element name="individual">
        <xs:complexType>
            <xs:sequence>
                <xs:element name="name">
                    <xs:simpleType>
                        <xs:restriction base="xs:string">
                            <xs:maxLength value="5" />
                        </xs:restriction>
                    </xs:simpleType>
                </xs:element>
                <xs:element name="address">
                    <xs:complexType>
                        <xs:sequence>
                            <xs:element name="zip" type="xs:positiveInteger" />
                            <xs:element name="city" type="xs:string" />
                            <xs:element name="street" type="xs:string" />
                        </xs:sequence>
                    </xs:complexType>
                </xs:element>
            </xs:sequence>
        </xs:complexType>
    </xs:element>
</xs:schema>
```

有两个问题:

*   名称属性最多限于 5 个字符
*   该地址需要街道属性

让我们看看如何使用 Java 来获取这些信息。

## 3.根据 XSD 文件验证 XML 文件

**`javax.xml.validation`包定义了一个验证 XML 文档的 API。
**

首先，我们将准备一个能够读取遵循 XML Schema 1.0 规范的文件的`SchemaFactory` 。然后，我们将使用这个`SchemaFactory`来创建对应于我们的 XSD 文件的`Schema`。一个`Schema`代表一组约束。

最后，我们将从`Schema`中检索`Validator`。`Validator`是根据`Schema`检查 XML 文档的处理器:

```java
private Validator initValidator(String xsdPath) throws SAXException {
    SchemaFactory factory = SchemaFactory.newInstance(XMLConstants.W3C_XML_SCHEMA_NS_URI);
    Source schemaFile = new StreamSource(getFile(xsdPath));
    Schema schema = factory.newSchema(schemaFile);
    return schema.newValidator();
}
```

在这段代码中，`getFile`方法允许我们[将 XSD](/web/20220830095532/https://www.baeldung.com/reading-file-in-java) 读成一个 [`File`](/web/20220830095532/https://www.baeldung.com/java-io-file) 。在我们的示例中，我们将文件放在 resources 目录下，因此该方法读作:

```java
private File getFile(String location) {
    return new File(getClass().getClassLoader().getResource(location).getFile());
}
```

让我们注意，当我们创建`Schema`时，如果 XSD 文件无效，就会抛出一个`SAXException`。

**我们现在可以使用`Validator`来验证 XML 文件是否匹配 XSD 描述。**`validate`方法要求我们将`File`转化为`StreamSource`:

```java
public boolean isValid(String xsdPath, String xmlPath) throws IOException, SAXException {
    Validator validator = initValidator(xsdPath);
    try {
        validator.validate(new StreamSource(getFile(xmlPath)));
        return true;
    } catch (SAXException e) {
        return false;
    }
}
```

如果解析过程中出现错误,`validate`方法抛出一个`SAXException `。这表明根据 XSD 规范，XML 文件是无效的。

如果在读取文件时存在潜在问题，`validate`方法也可以抛出一个`[IOException](https://web.archive.org/web/20220830095532/https://baeldung-cn.com/java-checked-unchecked-exceptions)` 。

我们现在可以将代码包装在一个`XmlValidator`类中，并检查`baeldung.xml` 是否匹配`person.xsd`描述，而不是`full-person.xsd`:

```java
@Test
public void givenValidXML_WhenIsValid_ThenTrue() throws IOException, SAXException {
    assertTrue(new XmlValidator().isValid("person.xsd", "baeldung.xml"));
}

@Test
public void givenInvalidXML_WhenIsValid_ThenFalse() throws IOException, SAXException {
    assertFalse(new XmlValidator().isValid("full-person.xsd", "baeldung.xml"));
}
```

## 4.列出所有验证错误

`validate`方法的基本行为是一旦解析抛出一个`SAXException`就退出。

既然我们想要收集所有的验证错误，我们需要改变这种行为。为此，**我们要定义自己的`ErrorHandler` :**

```java
public class XmlErrorHandler implements ErrorHandler {

    private List<SAXParseException> exceptions;

    public XmlErrorHandler() {
        this.exceptions = new ArrayList<>();
    }

    public List<SAXParseException> getExceptions() {
        return exceptions;
    }

    @Override
    public void warning(SAXParseException exception) {
        exceptions.add(exception);
    }

    @Override
    public void error(SAXParseException exception) {
        exceptions.add(exception);
    }

    @Override
    public void fatalError(SAXParseException exception) {
        exceptions.add(exception);
    }
}
```

我们现在可以告诉`Validator`使用这个特定的`ErrorHandler`:

```java
public List<SAXParseException> listParsingExceptions(String xsdPath, String xmlPath) throws IOException, SAXException {
    XmlErrorHandler xsdErrorHandler = new XmlErrorHandler();
    Validator validator = initValidator(xsdPath);
    validator.setErrorHandler(xsdErrorHandler);
    try {
        validator.validate(new StreamSource(getFile(xmlPath)));
    } catch (SAXParseException e) 
    {
        // ...
    }
    xsdErrorHandler.getExceptions().forEach(e -> LOGGER.info(e.getMessage()));
    return xsdErrorHandler.getExceptions();
}
```

由于`baeldung.xml`满足`person.xsd`的要求，在这种情况下没有列出错误。但是，用`full-person.xsd`调用，我们会打印以下错误信息:

```java
XmlValidator - cvc-maxLength-valid: Value 'Baeldung' with length = '8' is not facet-valid with respect to maxLength '5' for type '#AnonType_nameindividual'.
XmlValidator - cvc-type.3.1.3: The value 'Baeldung' of element 'name' is not valid.
XmlValidator - cvc-complex-type.2.4.b: The content of element 'address' is not complete. One of '{street}' is expected.
```

我们在第一节提到的所有错误。被程序发现了。

## 5.结论

在本文中，我们看到了如何根据 XSD 文件验证 XML 文件，并且我们还可以列出所有的验证错误。

和往常一样，代码可以在 GitHub 的[上获得。](https://web.archive.org/web/20220830095532/https://github.com/eugenp/tutorials/tree/master/xml-2)