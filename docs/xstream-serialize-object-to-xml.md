# XStream 用户指南:将对象转换为 XML

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/xstream-serialize-object-to-xml>

## 1。概述

在本教程中，我们将学习如何使用 [XStream](https://web.archive.org/web/20220120140643/https://x-stream.github.io/) 库将 Java 对象序列化为 XML。

## 2。功能

使用 XStream 序列化和反序列化 XML 有很多有趣的好处:

*   如果配置得当，它会生成非常干净的 XML
*   为 XML 输出的**定制**提供了重要机会
*   支持**对象图**，包括循环引用
*   对于大多数用例来说，XStream 实例是线程安全的，一旦配置好(使用注释时有一些注意事项)
*   在**异常处理**期间提供明确的消息，以帮助诊断问题
*   从版本 1.4.7 开始，我们有了**安全特性**来禁止某些类型的序列化

## 3。项目设置

为了在我们的项目中使用 XStream，我们将添加以下 Maven 依赖项:

```java
<dependency>
    <groupId>com.thoughtworks.xstream</groupId>
    <artifactId>xstream</artifactId>
    <version>1.4.18</version>
</dependency>
```

## 4。基本用法

`XStream`类是 API 的一个门面。当创建`XStream`的实例时，我们还需要注意线程安全问题:

```java
XStream xstream = new XStream();
```

创建和配置实例后，除非启用注释处理，否则它可能会在多个线程之间共享，以便进行编组/解组。

### 4.1。驱动程序

支持几个驱动，比如`DomDriver`、 `StaxDriver`、 `XppDriver`等等。这些驱动程序具有不同的性能和资源使用特征。

默认情况下使用 XPP3 驱动程序，但是我们当然可以轻松地更改驱动程序:

```java
XStream xstream = new XStream(new StaxDriver()); 
```

### 4.2。正在生成 XML

让我们首先为–`Customer`定义一个简单的 POJO:

```java
public class Customer {

    private String firstName;
    private String lastName;
    private Date dob;

    // standard constructor, setters, and getters
}
```

现在让我们生成对象的 XML 表示:

```java
Customer customer = new Customer("John", "Doe", new Date());
String dataXml = xstream.toXML(customer);
```

使用默认设置，将产生以下输出:

```java
<com.baeldung.pojo.Customer>
    <firstName>John</firstName>
    <lastName>Doe</lastName>
    <dob>1986-02-14 03:46:16.381 UTC</dob>
</com.baeldung.pojo.Customer> 
```

从这个输出中，我们可以清楚地看到包含标签默认使用全限定类名`Customer`的`.`

有很多原因可以让我们决定默认行为不适合我们的需求。例如，我们可能不愿意公开应用程序的包结构。此外，生成的 XML 要长得多。

## 5。别名

**别名**是我们希望用于元素的名称，而不是使用默认名称。

例如，我们可以通过为`Customer` 类注册一个别名来用`customer`替换`com.baeldung.pojo.Customer`。我们也可以为一个类的属性添加别名。通过使用别名，我们可以使我们的 XML 输出可读性更强，并且不那么特定于 Java。

### 5.1。阶级别名

别名可以通过编程或使用注释来注册。

现在让我们用`@XStreamAlias`来注释我们的`Customer`类:

```java
@XStreamAlias("customer")
```

现在我们需要配置我们的实例来使用这个注释:

```java
xstream.processAnnotations(Customer.class);
```

或者，如果我们希望以编程方式配置别名，我们可以使用下面的代码:

```java
xstream.alias("customer", Customer.class);
```

无论是使用别名还是编程配置，`Customer`对象的输出将更加清晰:

```java
<customer>
    <firstName>John</firstName>
    <lastName>Doe</lastName>
    <dob>1986-02-14 03:46:16.381 UTC</dob>
</customer> 
```

### 5.2。字段别名

我们还可以使用用于别名类的相同注释为字段添加别名。例如，如果我们希望在 XML 表示中用`fn`替换字段`firstName`，我们可以使用下面的注释:

```java
@XStreamAlias("fn")
private String firstName;
```

或者，我们可以通过编程实现相同的目标:

```java
xstream.aliasField("fn", Customer.class, "firstName");
```

`aliasField`方法接受三个参数:我们希望使用的别名、定义属性的类以及我们希望别名的属性名。

无论使用哪种方法，输出都是一样的:

```java
<customer>
    <fn>John</fn>
    <lastName>Doe</lastName>
    <dob>1986-02-14 03:46:16.381 UTC</dob>
</customer>
```

### 5.3。默认别名

课程有几个预先注册的别名，以下是其中的几个:

```java
alias("float", Float.class);
alias("date", Date.class);
alias("gregorian-calendar", Calendar.class);
alias("url", URL.class);
alias("list", List.class);
alias("locale", Locale.class);
alias("currency", Currency.class);
```

## 6。收藏

现在我们将在`Customer`类中添加一个`ContactDetails`列表。

```java
private List<ContactDetails> contactDetailsList;
```

对于集合处理的默认设置，以下是输出:

```java
<customer>
    <firstName>John</firstName>
    <lastName>Doe</lastName>
    <dob>1986-02-14 04:14:05.874 UTC</dob>
    <contactDetailsList>
        <ContactDetails>
            <mobile>6673543265</mobile>
            <landline>0124-2460311</landline>
        </ContactDetails>
        <ContactDetails>
            <mobile>4676543565</mobile>
            <landline>0120-223312</landline>
        </ContactDetails>
    </contactDetailsList>
</customer>
```

假设我们需要省略`contactDetailsList` 父标签`,`，我们只希望每个`ContactDetails`元素是`customer`元素的子元素。让我们再次修改我们的例子:

```java
xstream.addImplicitCollection(Customer.class, "contactDetailsList");
```

现在，当生成 XML 时，根标签被省略，产生下面的 XML:

```java
<customer>
    <firstName>John</firstName>
    <lastName>Doe</lastName>
    <dob>1986-02-14 04:14:20.541 UTC</dob>
    <ContactDetails>
        <mobile>6673543265</mobile>
        <landline>0124-2460311</landline>
    </ContactDetails>
    <ContactDetails>
        <mobile>4676543565</mobile>
        <landline>0120-223312</landline>
    </ContactDetails>
</customer>
```

使用注释也可以达到同样的效果:

```java
@XStreamImplicit
private List<ContactDetails> contactDetailsList;
```

## 7。转换器

XStream 使用一个`Converter`实例的映射，每个实例都有自己的转换策略。这些函数将提供的数据转换成特定的 XML 格式，然后再转换回来。

除了使用默认转换器，我们还可以修改默认值或注册自定义转换器。

### 7.1。修改现有转换器

假设我们对使用默认设置生成`dob` 标签的方式不满意。我们可以修改 XStream ( `DateConverter`)提供的`Date`的自定义转换器:

```java
xstream.registerConverter(new DateConverter("dd-MM-yyyy", null));
```

以上将产生“`dd-MM-yyyy`”格式的输出:

```java
<customer>
    <firstName>John</firstName>
    <lastName>Doe</lastName>
    <dob>14-02-1986</dob>
</customer>
```

### 7.2。定制转换器

我们还可以创建一个自定义转换器来实现与上一节相同的输出:

```java
public class MyDateConverter implements Converter {

    private SimpleDateFormat formatter = new SimpleDateFormat("dd-MM-yyyy");

    @Override
    public boolean canConvert(Class clazz) {
        return Date.class.isAssignableFrom(clazz);
    }

    @Override
    public void marshal(
      Object value, HierarchicalStreamWriter writer, MarshallingContext arg2) {
        Date date = (Date)value;
        writer.setValue(formatter.format(date));
    }

    // other methods
}
```

最后，我们注册我们的`MyDateConverter` 类如下:

```java
xstream.registerConverter(new MyDateConverter());
```

我们还可以创建实现`SingleValueConverter` 接口的转换器，该接口被设计成将对象转换成字符串。

```java
public class MySingleValueConverter implements SingleValueConverter {

    @Override
    public boolean canConvert(Class clazz) {
        return Customer.class.isAssignableFrom(clazz);
    }

    @Override
    public String toString(Object obj) {
        SimpleDateFormat formatter = new SimpleDateFormat("dd-MM-yyyy");
        Date date = ((Customer) obj).getDob();
        return ((Customer) obj).getFirstName() + "," 
          + ((Customer) obj).getLastName() + ","
          + formatter.format(date);
    }

    // other methods
}
```

最后，我们注册`MySingleValueConverter`:

```java
xstream.registerConverter(new MySingleValueConverter()); 
```

使用`MySingleValueConverter`，一个`Customer`的 XML 输出如下:

```java
<customer>John,Doe,14-02-1986</customer>
```

### 7.3。转换器优先级

注册`Converter`对象时，也可以设置其优先级。

从 [XStream javadocs](https://web.archive.org/web/20220120140643/https://x-stream.github.io/javadoc/com/thoughtworks/xstream/XStream.html) 开始:

> 转换器可以用明确的优先级注册。默认情况下，它们注册到 XStream。优先级 _ 正常。相同优先级的转换器将按注册的相反顺序使用。默认转换器，即如果没有其他已注册的转换器适用时将使用的转换器，可以用优先级 XStream 注册。优先级非常低。默认情况下，XStream 使用 ReflectionConverter 作为回退转换器。

API 提供了几个命名的优先级值:

```java
private static final int PRIORITY_NORMAL = 0;
private static final int PRIORITY_LOW = -10;
private static final int PRIORITY_VERY_LOW = -20; 
```

## 8。 **省略字段**

我们可以使用注释或编程配置从生成的 XML 中省略字段。为了使用注释省略一个字段，我们简单地将`@XStreamOmitField`注释应用到有问题的字段:

```java
@XStreamOmitField 
private String firstName;
```

为了以编程方式省略该字段，我们使用以下方法:

```java
xstream.omitField(Customer.class, "firstName");
```

无论我们选择哪种方法，输出都是一样的:

```java
<customer> 
    <lastName>Doe</lastName> 
    <dob>14-02-1986</dob> 
</customer>
```

## 9。属性字段

有时我们可能希望将字段序列化为元素的属性，而不是元素本身。假设我们添加了一个`contactType`字段:

```java
private String contactType;
```

如果我们想将`contactType`设置为 XML 属性，我们可以使用`@XStreamAsAttribute`注释:

```java
@XStreamAsAttribute
private String contactType; 
```

或者，我们可以通过编程实现相同的目标:

```java
xstream.useAttributeFor(ContactDetails.class, "contactType");
```

以上两种方法的输出是相同的:

```java
<ContactDetails contactType="Office">
    <mobile>6673543265</mobile>
    <landline>0124-2460311</landline>
</ContactDetails>
```

## 10。并发性

XStream 的处理模型提出了一些挑战。一旦实例被配置，它就是线程安全的。

值得注意的是，处理注释会在编组/解组之前修改配置。因此——如果我们需要使用注释动态地配置实例，通常为每个线程使用一个单独的`XStream` 实例是一个好主意。

## 11。结论

在本文中，我们介绍了使用 XStream 将对象转换为 XML 的基础知识。我们还了解了可以用来确保 XML 输出满足我们需求的定制。最后，我们看了注释的线程安全问题。

在本系列的下一篇文章中，我们将学习如何将 XML 转换回 Java 对象。

本文的完整源代码可以从链接的 [GitHub 库](https://web.archive.org/web/20220120140643/https://github.com/eugenp/tutorials/tree/master/xstream)下载。