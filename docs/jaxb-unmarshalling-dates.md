# 使用 JAXB 解组日期

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/jaxb-unmarshalling-dates>

## 1.介绍

在本教程中，**我们将看到如何使用 JAXB 解组不同格式的日期对象。**

首先，我们将介绍默认的模式日期格式。然后，我们将探索如何使用不同的格式。我们还将看到如何处理这些技术带来的常见挑战。

## 2.模式到 Java 的绑定

首先，**我们需要理解 XML 模式和 Java 数据类型之间的关系**。特别是，我们对 XML 模式和 Java 日期对象之间的映射感兴趣。

根据 **[Schema 到 Java 的映射](https://web.archive.org/web/20220626082800/https://docs.oracle.com/javase/tutorial/jaxb/intro/bind.html)** ，我们需要考虑三种模式数据类型: **`xsd:date`、`xsd:time`和`xsd:dateTime`** 。我们可以看到，它们都映射到了 **`javax.xml.datatype.XMLGregorianCalendar`** 。

我们还需要理解这些 XML 模式类型的默认格式。`xsd:date`和`xsd:time`数据类型有“`YYYY-MM-DD”`和“`hh:mm:ss”`格式。**`xsd:dateTime`格式为“`YYYY-MM-DDThh:mm:ss”`** ”，其中“`T”`为分隔符，表示时间段的开始。

## 3.使用默认的模式日期格式

我们将构建一个解组日期对象的示例。让我们关注一下`xsd:dateTime`数据类型，因为它是其他类型的超集。

让我们用一个简单的 XML 文件来描述一本书:

```java
<book>
    <title>Book1</title>
    <published>1979-10-21T03:31:12</published>
</book>
```

我们希望将文件映射到相应的 Java `Book`对象:

```java
@XmlRootElement(name = "book")
public class Book {

    @XmlElement(name = "title", required = true)
    private String title;

    @XmlElement(name = "published", required = true)
    private XMLGregorianCalendar published;

    @Override
    public String toString() {
        return "[title: " + title + "; published: " + published.toString() + "]";
    }

}
```

最后，我们需要创建一个客户机应用程序，将 XML 数据转换成 JAXB 派生的 Java 对象:

```java
public static Book unmarshalDates(InputStream inputFile) 
  throws JAXBException {
    JAXBContext jaxbContext = JAXBContext.newInstance(Book.class);
    Unmarshaller jaxbUnmarshaller = jaxbContext.createUnmarshaller();
    return (Book) jaxbUnmarshaller.unmarshal(inputFile);
}
```

在上面的代码中，我们定义了一个`JAXBContext`，它是 JAXB API 的入口点。然后，我们在输入流上使用了 JAXB `Unmarshaller`来读取我们的对象:

如果我们运行上面的代码并打印结果，我们将得到下面的`Book`对象:

```java
[title: Book1; published: 1979-11-28T02:31:32]
```

我们应该注意到，尽管`xsd:dateTime`的默认映射是`XMLGregorianCalendar`，但是根据 [JAXB 用户指南](https://web.archive.org/web/20220626082800/https://javaee.github.io/jaxb-v2/doc/user-guide/ch03.html#customization-of-schema-compilation-using-different-datatypes)，我们也可以使用更常见的 Java 类型:`java.util.Date` 和`java.util.Calendar` `,`。

## 4.使用自定义日期格式

上面的例子之所以有效，是因为我们使用了默认的模式日期格式`“YYYY-MM-DDThh:mm:ss”.`

但是如果我们想使用另一种格式，比如去掉分隔符`“T”`的`“YYYY-MM-DD hh:mm:ss”,` 呢？如果我们在 XML 文件中用空格字符替换分隔符，默认的解组将会失败。

### 4.1.构建自定义`XmlAdapter`

**为了使用不同的日期格式，我们需要定义一个`XmlAdapter`。**

让我们看看如何用我们的自定义`XmlAdapter:`将`xsd:dateTime`类型映射到`java.util.Date`对象

```java
public class DateAdapter extends XmlAdapter<String, Date> {

    private static final String CUSTOM_FORMAT_STRING = "yyyy-MM-dd HH:mm:ss";

    @Override
    public String marshal(Date v) {
        return new SimpleDateFormat(CUSTOM_FORMAT_STRING).format(v);
    }

    @Override
    public Date unmarshal(String v) throws ParseException {
        return new SimpleDateFormat(CUSTOM_FORMAT_STRING).parse(v);
    }

}
```

在这个适配器中，**我们使用了** **[`SimpleDateFormat`](/web/20220626082800/https://www.baeldung.com/java-simple-date-format) 来格式化我们的日期。**我们需要小心，因为**的`SimpleDateFormat`不是[线程安全的](/web/20220626082800/https://www.baeldung.com/java-thread-safety)。**为了避免多线程遇到共享`SimpleDateFormat`对象的问题，我们在每次需要的时候都会创建一个新的。

### 4.2.`XmlAdapter`的内部结构

我们可以看到， **`XmlAdapter`有两个类型参数**，在这里是`String`和`Date`。第一个是 XML 内部使用的类型，称为值类型。在这种情况下，JAXB 知道如何将 XML 值转换成`String`。第二个称为绑定类型，与 Java 对象中的值相关。

适配器的目标是在值类型和绑定类型之间进行转换，这是 JAXB 默认无法做到的。

为了构建自定义的`Xml` `Adapter`，我们必须覆盖两个方法:`XmlAdapter.marshal()`和`XmlAdapter.unmarshal()`。

在解组过程中，JAXB 绑定框架首先将 XML 表示解组到一个`String`，然后调用`DateAdapter.unmarshal()`将值类型适配到一个`Date`。在编组期间，JAXB 绑定框架调用`DateAdapter.marshal()`将`Date`改编为`String`，然后将其编组为 XML 表示。

### 4.3.通过 JAXB 注释集成

`DateAdapter`像 JAXB 的插件一样工作，我们将使用`@XmlJavaTypeAdapter`注释把它附加到我们的日期字段。**`@XmlJavaTypeAdapte`r 注释指定使用`XmlAdapter`进行自定义解组**:

```java
@XmlRootElement(name = "book")
public class BookDateAdapter {

    // same as before

    @XmlElement(name = "published", required = true)
    @XmlJavaTypeAdapter(DateAdapter.class)
    private Date published;

    // same as before

}
```

我们还使用了[标准 JAXB 注释](/web/20220626082800/https://www.baeldung.com/jaxb) : `@XmlRootElement`和`@XmlElement`注释。

最后，让我们运行新代码:

```java
[title: Book1; published: Wed Nov 28 02:31:32 EET 1979]
```

## 5.Java 8 中的解组日期

Java 8 引入了新的 [`Date/Time` API](/web/20220626082800/https://www.baeldung.com/java-8-date-time-intro) 。这里，我们将重点关注最常用的`LocalDateTime`类。

### 5.1.构建基于`LocalDateTime`的`XmlAdapter`

默认情况下， **JAXB 不能自动将`xsd:dateTime`值绑定到`LocalDateTime`** 对象，不管日期格式如何。为了在 XML 模式日期值和`LocalDateTime`对象之间进行转换，我们需要定义另一个类似于上一个的`XmlAdapter` :

```java
public class LocalDateTimeAdapter extends XmlAdapter<String, LocalDateTime> {

    private DateTimeFormatter dateFormat = DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss");

    @Override
    public String marshal(LocalDateTime dateTime) {
        return dateTime.format(dateFormat);
    }

    @Override
    public LocalDateTime unmarshal(String dateTime) {
        return LocalDateTime.parse(dateTime, dateFormat);
    }

}
```

在本例中，**我们使用了 [`DateTimeFormatter`](/web/20220626082800/https://www.baeldung.com/java-datetimeformatter) 而不是`SimpleDateFormat`。前者是在 Java 8 中引入的，它与新的`Date/Time` API 兼容。**

注意，转换操作可以共享一个`DateTimeFormatter` 对象，因为`DateTimeFormatter`的**是线程安全的。**

### 5.2.集成新适配器

现在，让我们在我们的`Book`类中用新的适配器替换旧的适配器，并且用`LocalDateTime`替换`Date`:

```java
@XmlRootElement(name = "book")
public class BookLocalDateTimeAdapter {

    // same as before

    @XmlElement(name = "published", required = true)
    @XmlJavaTypeAdapter(LocalDateTimeAdapter.class)
    private LocalDateTime published;

    // same as before

}
```

如果我们运行上面的代码，我们将得到输出:

```java
[title: Book1; published: 1979-11-28T02:31:32]
```

请注意，`LocalDateTime.toString()`在日期和时间之间添加了分隔符`“T”`。

## 6.结论

在本教程中，我们探索了使用 JAXB 对日期进行**解组。**

首先，我们查看了 XML 模式到 Java 数据类型的映射，并使用默认的 XML 模式日期格式创建了一个示例。

接下来，我们学习了如何使用基于自定义`XmlAdapter`的自定义日期格式，以及如何处理`SimpleDateFormat`的线程安全。

最后，我们利用了高级的、线程安全的 Java 8 日期/时间 API，并用自定义格式解组日期。

和往常一样，教程中使用的源代码可以从 GitHub 上的[处获得。](https://web.archive.org/web/20220626082800/https://github.com/eugenp/tutorials/tree/master/jaxb)