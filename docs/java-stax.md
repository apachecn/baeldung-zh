# 使用 StAX 解析 XML 文件

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-stax>

## 1.介绍

在本教程中，我们将演示如何使用 StAX 解析 XML 文件。我们将实现一个简单的 XML 解析器，并通过一个例子来看看它是如何工作的。

## 2.用 StAX 解析

StAX 是 Java 中几个 [XML 库之一。**这是一个从 Java 6 开始就包含在 JDK 中的高效内存库。** StAX 不会将整个 XML 加载到内存中。相反，它以只进的方式从流中提取数据。该流由一个`XMLEventReader`对象读取。](/web/20220628085416/https://www.baeldung.com/java-xml-libraries)

## 3.`XMLEventReader`阶级

在 StAX 中，任何开始标签或结束标签都是一个事件。 **`XMLEventReader`将 XML 文件作为事件流读取。**它还提供了解析 XML 所必需的方法。最重要的方法是:

*   `isStartElement()`:检查当前事件是否为`StartElement` (开始标签)
*   `isEndElement()`:检查当前事件是否为`EndElement` (结束标签)
*   `asCharacters()`:以字符形式返回当前事件
*   `getName()`:获取当前事件的名称
*   `getAttributes()`:返回当前事件属性的`Iterator`

## 4.实现一个简单的 XML 解析器

不用说，解析 XML 的第一步是读取它。我们需要一个`XMLInputFactory`来创建一个`XMLEventReader`来读取我们的文件:

```java
XMLInputFactory xmlInputFactory = XMLInputFactory.newInstance();
XMLEventReader reader = xmlInputFactory.createXMLEventReader(new FileInputStream(path));
```

现在`XMLEventReader`已经准备好了，我们带着`nextEvent()`继续前进:

```java
while (reader.hasNext()) {
    XMLEvent nextEvent = reader.nextEvent();
}
```

接下来，我们需要首先找到我们想要的开始标记:

```java
if (nextEvent.isStartElement()) {
    StartElement startElement = nextEvent.asStartElement();
    if (startElement.getName().getLocalPart().equals("desired")) {
        //...
    }
}
```

因此，我们可以读取属性和数据:

```java
String url = startElement.getAttributeByName(new QName("url")).getValue();
String name = nextEvent.asCharacters().getData();
```

我们还可以检查是否到达了结束标记:

```java
if (nextEvent.isEndElement()) {
    EndElement endElement = nextEvent.asEndElement();
}
```

## 5.解析示例

为了更好地理解，让我们在一个样本 XML 文件上运行我们的解析器:

```java
<?xml version="1.0" encoding="UTF-8"?>
<websites>
    <website url="https://baeldung.com">
        <name>Baeldung</name>
        <category>Online Courses</category>
        <status>Online</status>
    </website>
    <website url="http://example.com">
        <name>Example</name>
        <category>Examples</category>
        <status>Offline</status>
    </website>
    <website url="http://localhost:8080">
        <name>Localhost</name>
        <category>Tests</category>
        <status>Offline</status>
    </website>
</websites>
```

让我们解析 XML 并将所有数据存储到名为`websites`的实体对象列表中:

```java
while (reader.hasNext()) {
    XMLEvent nextEvent = reader.nextEvent();
    if (nextEvent.isStartElement()) {
        StartElement startElement = nextEvent.asStartElement();
        switch (startElement.getName().getLocalPart()) {
            case "website":
                website = new WebSite();
                Attribute url = startElement.getAttributeByName(new QName("url"));
                if (url != null) {
                    website.setUrl(url.getValue());
                }
                break;
            case "name":
                nextEvent = reader.nextEvent();
                website.setName(nextEvent.asCharacters().getData());
                break;
            case "category":
                nextEvent = reader.nextEvent();
                website.setCategory(nextEvent.asCharacters().getData());
                break;
            case "status":
                nextEvent = reader.nextEvent();
                website.setStatus(nextEvent.asCharacters().getData());
                break;
        }
    }
    if (nextEvent.isEndElement()) {
        EndElement endElement = nextEvent.asEndElement();
        if (endElement.getName().getLocalPart().equals("website")) {
            websites.add(website);
        }
    }
}
```

为了获得每个网站的所有属性，我们检查每个事件的`startElement.getName().getLocalPart()`。然后，我们相应地设置相应的属性。

当我们到达网站的结束元素时，我们知道我们的实体是完整的，所以我们将实体添加到我们的`websites`列表中。

## 6.结论

在本教程中，我们学习了如何使用 StAX 库解析 XML 文件。

示例 XML 文件和完整的解析器代码一如既往地在 Github 上的[处可用。](https://web.archive.org/web/20220628085416/https://github.com/eugenp/tutorials/tree/master/xml)