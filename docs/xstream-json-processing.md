# XStream 用户指南:JSON

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/xstream-json-processing>

## 1。概述

这是关于 XStream 系列的第三篇文章。如果你想了解它在[将 Java 对象转换成 XML](/web/20220122063317/https://www.baeldung.com/xstream-serialize-object-to-xml) 和[反之亦然](/web/20220122063317/https://www.baeldung.com/xstream-deserialize-xml-to-object)中的基本用法，请参考之前的文章。

除了 XML 处理能力之外，XStream 还可以将 Java 对象与 JSON 相互转换。在本教程中，我们将了解这些功能。

## 2。先决条件

在阅读本教程之前，请阅读本系列的第一篇文章`,` ，在这篇文章中我们解释了库的基础知识。

## 3。依赖性

```java
<dependency>
    <groupId>com.thoughtworks.xstream</groupId>
    <artifactId>xstream</artifactId>
    <version>1.4.18</version>
</dependency>
```

## 4。JSON 驱动程序

在前面的文章中，我们学习了如何设置 XStream 实例和选择 XML 驱动程序。同样，有两个驱动程序可以在 JSON 和对象之间相互转换:`**[JsonHierarchicalStreamDriver](https://web.archive.org/web/20220122063317/https://x-stream.github.io/javadoc/com/thoughtworks/xstream/io/json/JsonHierarchicalStreamDriver.html)**` 和 [`**JettisonMappedXmlDriver**`](https://web.archive.org/web/20220122063317/https://x-stream.github.io/javadoc/com/thoughtworks/xstream/io/json/JettisonMappedXmlDriver.html) 。

### 4.1。`JsonHierarchicalStreamDriver`

这个驱动程序类可以将对象序列化为 JSON，但是不能反序列化回对象。它不需要任何额外的依赖，并且它的驱动程序类是独立的。

### 4.2。`JettisonMappedXmlDriver`

这个驱动程序类能够在 JSON 和对象之间进行转换。使用这个驱动程序类，我们需要为`jettison**.**`添加一个额外的依赖项

```java
<dependency>
    <groupId>org.codehaus.jettison</groupId>
    <artifactId>jettison</artifactId>
    <version>1.4.1</version>
</dependency>
```

## 5。将对象序列化为 JSON

让我们创建一个`Customer`类:

```java
public class Customer {

    private String firstName;
    private String lastName;
    private Date dob;
    private String age;
    private List<ContactDetails> contactDetailsList;

    // getters and setters
}
```

请注意，我们已经(可能出乎意料地)将`age`创建为`String`。我们将在后面解释这个选择。

### 5.1。使用`JsonHierarchicalStreamDriver`

我们将传递一个`JsonHierarchicalStreamDriver`来创建一个 XStream 实例。

```java
xstream = new XStream(new JsonHierarchicalStreamDriver());
dataJson = xstream.toXML(customer);
```

这会生成以下 JSON:

```java
{
  "com.baeldung.pojo.Customer": {
    "firstName": "John",
    "lastName": "Doe",
    "dob": "1986-02-14 16:22:18.186 UTC",
    "age": "30",
    "contactDetailsList": [
      {
        "mobile": "6673543265",
        "landline": "0124-2460311"
      },
      {
        "mobile": "4676543565",
        "landline": "0120-223312"
      }
    ]
  }
}
```

### 5.2。`JettisonMappedXmlDriver`实施

我们将传递一个`JettisonMappedXmlDriver`类来创建一个实例。

```java
xstream = new XStream(new JettisonMappedXmlDriver());
dataJson = xstream.toXML(customer);
```

这会生成以下 JSON:

```java
{
  "com.baeldung.pojo.Customer": {
    "firstName": "John",
    "lastName": "Doe",
    "dob": "1986-02-14 16:25:50.745 UTC",
    "age": 30,
    "contactDetailsList": [
      {
        "com.baeldung.pojo.ContactDetails": [
          {
            "mobile": 6673543265,
            "landline": "0124-2460311"
          },
          {
            "mobile": 4676543565,
            "landline": "0120-223312"
          }
        ]
      }
    ]
  }
}
```

### 5.3。分析

基于两个驱动程序的输出，我们可以清楚地看到生成的 JSON 有一些细微的差别。例如，`JettisonMappedXmlDriver`省略了数值的双引号，尽管数据类型是`java.lang.String` :

```java
"mobile": 4676543565,
"age": 30,
```

另一方面，`JsonHierarchicalStreamDriver`保留了双引号。**T1**

## 6。将 JSON 反序列化为一个对象

让我们用下面的 JSON 将其转换回一个`Customer` 对象:

```java
{
  "customer": {
    "firstName": "John",
    "lastName": "Doe",
    "dob": "1986-02-14 16:41:01.987 UTC",
    "age": 30,
    "contactDetailsList": [
      {
        "com.baeldung.pojo.ContactDetails": [
          {
            "mobile": 6673543265,
            "landline": "0124-2460311"
          },
          {
            "mobile": 4676543565,
            "landline": "0120-223312"
          }
        ]
      }
    ]
  }
}
```

回想一下，只有其中一个驱动程序(`JettisonMappedXMLDriver`)可以反序列化 JSON。试图为此目的使用`JsonHierarchicalStreamDriver`将导致`UnsupportedOperationException`。

使用抛弃驱动程序，我们可以反序列化`Customer`对象:

```java
customer = (Customer) xstream.fromXML(dataJson);
```

## 7 .**。结论**

在本文中，我们讨论了 JSON 处理能力 XStream，在 JSON 和对象之间进行转换。我们还研究了如何调整我们的 JSON 输出，使它更短、更简单、更易读。

与 XStream 的 XML 处理一样，我们还可以通过使用注释或编程配置来配置实例，从而进一步定制 JSON 的序列化方式。有关更多细节和示例，请参考本系列的第一篇文章。

带有示例的完整源代码可以从[链接的 GitHub 库](https://web.archive.org/web/20220122063317/https://github.com/eugenp/tutorials/tree/master/xstream)下载。