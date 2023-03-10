# XStream 用户指南:将 XML 转换为对象

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/xstream-deserialize-xml-to-object>

## 1。概述

在[之前的一篇文章](/web/20220120024944/https://www.baeldung.com/xstream-serialize-object-to-xml)中，我们学习了如何使用 XStream 将 Java 对象序列化为 XML。在本教程中，我们将学习如何反向操作:将 XML 反序列化为 Java 对象。这些任务可以使用注释或通过编程来完成。

要了解设置 XStream 及其依赖项的基本要求，请参考上一篇文章。

## 2。从 XML 反序列化一个对象

首先，假设我们有以下 XML:

```java
<com.baeldung.pojo.Customer>
    <firstName>John</firstName>
    <lastName>Doe</lastName>
    <dob>1986-02-14 03:46:16.381 UTC</dob>
</com.baeldung.pojo.Customer>
```

我们需要将它转换成一个 Java `Customer`对象:

```java
public class Customer {

    private String firstName;
    private String lastName;
    private Date dob;

    // standard setters and getters
} 
```

XML 可以通过多种方式输入，包括`File`、`InputStream`、`Reader`或`String`。为了简单起见，我们假设上面的 XML 在一个`String`对象中。

```java
Customer convertedCustomer = (Customer) xstream.fromXML(customerXmlString);
Assert.assertTrue(convertedCustomer.getFirstName().equals("John"));
```

## 3.安全方面

因为 XStream 使用未记录的 Java 功能和 Java 反射，所以可能容易受到任意代码执行或远程命令执行攻击。

深入的安全考虑超出了本教程的范围，但是我们有一篇专门的文章解释了这种威胁。此外，值得看看 [XStream 的官方页面](https://web.archive.org/web/20220120024944/https://x-stream.github.io/security.html)。

出于我们教程的目的，让我们假设我们所有的类都是“安全的”。因此，我们需要配置 XStream:

```java
XStream xstream = new XStream();
xstream.allowTypesByWildcard(new String[]{"com.baeldung.**"}); 
```

## 4。别名

在第一个例子中，XML 在最外层的 XML 标记中有类的完全限定名，匹配我们的`Customer`类的位置。有了这个设置，XStream 无需任何额外的配置就可以轻松地将 XML 转换成我们的对象。但我们可能并不总是具备这些条件。我们可能无法控制 XML 标记命名，或者我们可能决定为字段添加别名。

例如，假设我们修改了 XML，不使用外部标记的全限定类名:

```java
<customer>
    <firstName>John</firstName>
    <lastName>Doe</lastName>
    <dob>1986-02-14 03:46:16.381 UTC</dob>
</customer>
```

我们可以通过创建别名来转换这个 XML。

### 4.1。阶级别名

我们通过编程或使用注释向 XStream 实例注册别名。我们可以用`@XStreamAlias`来注释我们的`Customer`类:

```java
@XStreamAlias("customer")
public class Customer {
    //...
}
```

现在我们需要配置 XStream 实例来使用这个注释:

```java
xstream.processAnnotations(Customer.class);
```

或者，如果我们希望以编程方式配置别名，我们可以使用下面的代码:

```java
xstream.alias("customer", Customer.class);
```

### 4.2。字段别名

假设我们有以下 XML:

```java
<customer>
    <fn>John</fn>
    <lastName>Doe</lastName>
    <dob>1986-02-14 03:46:16.381 UTC</dob>
</customer>
```

`fn`标签与我们的`Customer`对象中的任何字段都不匹配，所以如果我们希望反序列化它，我们将需要为该字段定义一个别名。我们可以使用以下注释来实现这一点:

```java
@XStreamAlias("fn")
private String firstName;
```

或者，我们可以通过编程实现相同的目标:

```java
xstream.aliasField("fn", Customer.class, "firstName");
```

## 5。 **隐含收藏**

假设我们有下面的 XML，包含一个简单的`ContactDetails`列表:

```java
<customer>
    <firstName>John</firstName>
    <lastName>Doe</lastName>
    <dob>1986-02-14 04:14:20.541 UTC</dob>
    <ContactDetails>
        <mobile>6673543265</mobile>
        <landline>0124-2460311</landline>
    </ContactDetails>
    <ContactDetails>...</ContactDetails>
</customer>
```

我们希望将列表`ContactDetails`加载到 Java 对象的`List<ContactDetails>`字段中。我们可以通过使用以下注释来实现这一点:

```java
@XStreamImplicit
private List<ContactDetails> contactDetailsList;
```

或者，我们可以通过编程实现相同的目标:

```java
xstream.addImplicitCollection(Customer.class, "contactDetailsList");
```

## 6。 **忽略字段**

假设我们有以下 XML:

```java
<customer>
    <firstName>John</firstName>
    <lastName>Doe</lastName>
    <dob>1986-02-14 04:14:20.541 UTC</dob>
    <fullName>John Doe</fullName>
</customer>
```

在上面的 XML 中，我们有额外的元素`<fullName>`，这是 Java `Customer`对象所缺少的。

如果我们试图反序列化上面的 xml 而不考虑任何额外的元素，程序会抛出一个`UnknownFieldException`。

```java
No such field com.baeldung.pojo.Customer.fullName
```

正如异常明确指出的，XStream 不识别字段`fullName`。

为了解决这个问题，我们需要将其配置为忽略未知元素:

```java
xstream.ignoreUnknownElements();
```

## 7。属性字段

假设我们有带有属性的 XML 作为元素的一部分，我们希望将其反序列化为对象中的一个字段。我们将为我们的`ContactDetails`对象添加一个`contactType`属性:

```java
<ContactDetails contactType="Office">
    <mobile>6673543265</mobile>
    <landline>0124-2460311</landline>
</ContactDetails>
```

如果我们想要反序列化`contactType` XML 属性，我们可以在希望它出现的字段上使用`@XStreamAsAttribute`注释:

```java
@XStreamAsAttribute
private String contactType;
```

或者，我们可以通过编程实现相同的目标:

```java
xstream.useAttributeFor(ContactDetails.class, "contactType");
```

## 8。结论

在本文中，我们探讨了使用 XStream 将 XML 反序列化为 Java 对象时可用的选项。

本文的完整源代码可以从链接的 [GitHub 库](https://web.archive.org/web/20220120024944/https://github.com/eugenp/tutorials/tree/master/xstream)下载。