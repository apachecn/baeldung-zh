# Smooks 简介

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/smooks>

## 1。概述

在本教程中，我们将介绍 [Smooks 框架](https://web.archive.org/web/20221208143832/http://www.smooks.org/)。

我们将描述它是什么，列出它的主要特性，并最终学习如何使用它的一些更高级的功能。

首先，让我们简单解释一下这个框架的目的是什么。

## 2。Smooks

Smooks 是一个数据处理应用程序框架，用于处理 XML 或 CSV 等结构化数据。

它提供了 API 和配置模型，允许我们定义预定义格式之间的转换(例如 XML 到 CSV、XML 到 JSON 等等)。

我们还可以使用许多工具来设置我们的映射——包括 FreeMarker 或 Groovy 脚本。

除了转换，Smooks 还提供了其他特性，比如消息验证或数据分割。

### 2.1。主要特征

让我们来看看 Smooks 的主要用例:

*   消息转换–将数据从各种源格式转换为各种输出格式
*   消息充实——用额外的数据填充消息，这些数据来自外部数据源，如数据库
*   数据分割–处理大文件(GB)并将它们分割成较小的文件
*   Java 绑定——从消息中构造和填充 Java 对象
*   消息验证——像 regex 一样执行验证，甚至创建自己的验证规则

## 3。初始配置

让我们从需要添加到我们的`pom.xml`中的 Maven 依赖项开始:

```
<dependency>
    <groupId>org.milyn</groupId>
    <artifactId>milyn-smooks-all</artifactId>
    <version>1.7.0</version>
</dependency>
```

最新版本可以在 [Maven Central](https://web.archive.org/web/20221208143832/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.milyn%22%20AND%20a%3A%22milyn-smooks-all%22) 上找到。

## 4。Java 绑定

现在让我们从关注将消息绑定到 Java 类开始。这里我们将进行一个简单的 XML 到 Java 的转换。

### 4.1。基本概念

我们从一个简单的例子开始。考虑下面的 XML:

```
<order creation-date="2018-01-14">
    <order-number>771</order-number>
    <order-status>IN_PROGRESS</order-status>
</order>
```

为了用 Smooks 完成这项任务，我们必须做两件事:准备 POJOs 和 Smooks 配置。

让我们看看我们的模型是什么样的:

```
public class Order {

    private Date creationDate;
    private Long number;
    private Status status;
    // ...
} 
```

```
public enum Status {
    NEW, IN_PROGRESS, FINISHED
}
```

现在，让我们继续讨论 Smooks 映射。

基本上，映射是一个包含转换逻辑的 XML 文件。在本文中，我们将使用三种不同类型的规则:

*   `bean –` 定义具体结构化部分到 Java 类的映射
*   `value`–定义 bean 的特定属性的映射。可以包含更高级的逻辑，如解码器，用于将值映射到某些数据类型(如日期或十进制格式)
*   w `iring –` 允许我们将一个 bean 连接到其他 bean(例如`Supplier` bean 将连接到`Order` bean)

让我们看看我们将在这里的例子中使用的映射:

```
<?xml version="1.0"?>
<smooks-resource-list 

  xmlns:jb="http://www.milyn.org/xsd/smooks/javabean-1.2.xsd">

    <jb:bean beanId="order" 
      class="com.baeldung.smooks.model.Order" createOnElement="order">
        <jb:value property="number" data="order/order-number" />
        <jb:value property="status" data="order/order-status" />
        <jb:value property="creationDate" 
          data="order/@creation-date" decoder="Date">
            <jb:decodeParam name="format">yyyy-MM-dd</jb:decodeParam>
        </jb:value>
    </jb:bean>
</smooks-resource-list>
```

现在，配置准备好了，让我们试着测试我们的 POJO 是否构造正确。

首先，我们需要构造一个 Smooks 对象，并将输入 XML 作为一个流传递:

```
public Order converOrderXMLToOrderObject(String path) 
  throws IOException, SAXException {

    Smooks smooks = new Smooks(
      this.class.getResourceAsStream("/smooks-mapping.xml"));
    try {
        JavaResult javaResult = new JavaResult();
        smooks.filterSource(new StreamSource(this.class
          .getResourceAsStream(path)), javaResult);
        return (Order) javaResult.getBean("order");
    } finally {
        smooks.close();
    }
}
```

最后，判断配置是否正确:

```
@Test
public void givenOrderXML_whenConvert_thenPOJOsConstructedCorrectly() throws Exception {
    XMLToJavaConverter xmlToJavaOrderConverter = new XMLToJavaConverter();
    Order order = xmlToJavaOrderConverter
      .converOrderXMLToOrderObject("/order.xml");

    assertThat(order.getNumber(), is(771L));
    assertThat(order.getStatus(), is(Status.IN_PROGRESS));
    assertThat(
      order.getCreationDate(), 
      is(new SimpleDateFormat("yyyy-MM-dd").parse("2018-01-14"));
}
```

### 4.2。高级绑定——引用其他 Beans 和列表

让我们用`supplier` 和`order-items`标签来扩展我们之前的例子:

```
<order creation-date="2018-01-14">
    <order-number>771</order-number>
    <order-status>IN_PROGRESS</order-status>
    <supplier>
        <name>Company X</name>
        <phone>1234567</phone>
    </supplier>
    <order-items>
        <item>
            <quanitiy>1</quanitiy>
            <code>PX1234</code>
            <price>9.99</price>
        </item>
        <item>
            <quanitiy>1</quanitiy>
            <code>RX990</code>
            <price>120.32</price>
        </item>
    </order-items>
</order>
```

现在让我们更新我们的模型:

```
public class Order {
    // ..
    private Supplier supplier;
    private List<Item> items;
    // ...
}
```

```
public class Item {

    private String code;
    private Double price;
    private Integer quantity;
    // ...
} 
```

```
public class Supplier {

    private String name;
    private String phoneNumber;
    // ...
}
```

我们还必须用`supplier`和`item` bean 定义来扩展配置映射。

注意，我们还定义了单独的`items` bean，它将保存`ArrayList`中的所有`item`元素。

最后，我们将使用 Smooks `wiring` 属性将它们捆绑在一起。

看看在这种情况下映射会是什么样子:

```
<?xml version="1.0"?>
<smooks-resource-list 

  xmlns:jb="http://www.milyn.org/xsd/smooks/javabean-1.2.xsd">

    <jb:bean beanId="order" 
      class="com.baeldung.smooks.model.Order" createOnElement="order">
        <jb:value property="number" data="order/order-number" />
        <jb:value property="status" data="order/order-status" />
        <jb:value property="creationDate" 
          data="order/@creation-date" decoder="Date">
            <jb:decodeParam name="format">yyyy-MM-dd</jb:decodeParam>
        </jb:value>
        <jb:wiring property="supplier" beanIdRef="supplier" />
        <jb:wiring property="items" beanIdRef="items" />
    </jb:bean>

    <jb:bean beanId="supplier" 
      class="com.baeldung.smooks.model.Supplier" createOnElement="supplier">
        <jb:value property="name" data="name" />
        <jb:value property="phoneNumber" data="phone" />
    </jb:bean>

    <jb:bean beanId="items" 
      class="java.util.ArrayList" createOnElement="order">
        <jb:wiring beanIdRef="item" />
    </jb:bean>
    <jb:bean beanId="item" 
      class="com.baeldung.smooks.model.Item" createOnElement="item">
        <jb:value property="code" data="item/code" />
        <jb:value property="price" decoder="Double" data="item/price" />
        <jb:value property="quantity" decoder="Integer" data="item/quantity" />
    </jb:bean>

</smooks-resource-list>
```

最后，我们将在之前的测试中添加一些断言:

```
assertThat(
  order.getSupplier(), 
  is(new Supplier("Company X", "1234567")));
assertThat(order.getItems(), containsInAnyOrder(
  new Item("PX1234", 9.99,1),
  new Item("RX990", 120.32,1)));
```

## 5。消息验证

Smooks 带有基于规则的验证机制。让我们来看看它们是如何使用的。

规则的定义存储在配置文件中，嵌套在`ruleBases`标签中，标签可以包含许多`ruleBase`元素。

每个`ruleBase`元素必须具有以下属性:

*   `name –` 唯一名称，仅供参考
*   `src –` 规则源文件的路径
*   `provider` –全限定类名，实现`RuleProvider`接口

Smooks 自带了两个提供者:`RegexProvider`和`MVELProvider`。

第一个用于以类似正则表达式的方式验证单个字段。

第二个用于在文档的全局范围内执行更复杂的验证。让我们看看他们的行动。

### 5.1。`RegexProvider`

让我们使用`RegexProvider` 来验证两件事:客户姓名的格式和电话号码。`RegexProvider` 作为源需要一个 Java 属性文件，该文件**应该包含键值形式的正则表达式验证**。

为了满足我们的要求，我们将使用以下设置:

```
supplierName=[A-Za-z0-9]*
supplierPhone=^[0-9\\-\\+]{9,15}$
```

### 5.2。`MVELProvider`

我们将使用`MVELProvider` 来验证每个`order-item`的总价是否小于 200。作为源，我们将准备一个包含两列的 CSV 文件:规则名称和 MVEL 表达式。

为了检查价格是否正确，我们需要以下条目:

```
"max_total","orderItem.quantity * orderItem.price < 200.00"
```

### 5.3。验证配置

一旦我们为`ruleBases`准备好了源文件，我们将继续实现具体的验证。

验证是 Smooks 配置中的另一个标记，它包含以下属性:

*   `executeOn` –已验证元素的路径
*   `name`–参考`ruleBase`
*   `onFail`–指定验证失败时将采取的操作

让我们将验证规则应用到我们的 Smooks 配置文件，并检查它看起来是什么样子(**注意，如果我们想使用`MVELProvider`，我们被迫使用 Java 绑定，所以这就是为什么我们已经导入了以前的 Smooks 配置):**

```
<?xml version="1.0"?>
<smooks-resource-list 

  xmlns:rules="http://www.milyn.org/xsd/smooks/rules-1.0.xsd"
  xmlns:validation="http://www.milyn.org/xsd/smooks/validation-1.0.xsd">

    <import file="smooks-mapping.xml" />

    <rules:ruleBases>
        <rules:ruleBase 
          name="supplierValidation" 
          src="supplier.properties" 
          provider="org.milyn.rules.regex.RegexProvider"/>
        <rules:ruleBase 
          name="itemsValidation" 
          src="item-rules.csv" 
          provider="org.milyn.rules.mvel.MVELProvider"/>
    </rules:ruleBases>

    <validation:rule 
      executeOn="supplier/name" 
      name="supplierValidation.supplierName" onFail="ERROR"/>
    <validation:rule 
      executeOn="supplier/phone" 
      name="supplierValidation.supplierPhone" onFail="ERROR"/>
    <validation:rule 
      executeOn="order-items/item" 
      name="itemsValidation.max_total" onFail="ERROR"/>

</smooks-resource-list>
```

现在，配置准备好了，让我们试着测试对供应商电话号码的验证是否会失败。

同样，我们必须构造`Smooks`对象并将输入 XML 作为流传递:

```
public ValidationResult validate(String path) 
  throws IOException, SAXException {
    Smooks smooks = new Smooks(OrderValidator.class
      .getResourceAsStream("/smooks/smooks-validation.xml"));
    try {
        StringResult xmlResult = new StringResult();
        JavaResult javaResult = new JavaResult();
        ValidationResult validationResult = new ValidationResult();
        smooks.filterSource(new StreamSource(OrderValidator.class
          .getResourceAsStream(path)), xmlResult, javaResult, validationResult);
        return validationResult;
    } finally {
        smooks.close();
    }
} 
```

最后，如果发生验证错误，则断言:

```
@Test
public void givenIncorrectOrderXML_whenValidate_thenExpectValidationErrors() throws Exception {
    OrderValidator orderValidator = new OrderValidator();
    ValidationResult validationResult = orderValidator
      .validate("/smooks/order.xml");

    assertThat(validationResult.getErrors(), hasSize(1));
    assertThat(
      validationResult.getErrors().get(0).getFailRuleResult().getRuleName(), 
      is("supplierPhone"));
}
```

## 6。消息转换

接下来我们要做的是将消息从一种格式转换成另一种格式。

在 Smooks 中，这种技术也被称为模板化，它支持:

*   FreeMarker(首选)
*   可扩展样式表语言（Extensible Stylesheet Language 的缩写）
*   `String`模板

在我们的示例中，我们将使用 FreeMarker 引擎将 XML 消息转换成与 EDIFACT 非常相似的格式，甚至为基于 XML 订单的电子邮件准备一个模板。

让我们看看如何为 EDIFACT 准备模板:

```
UNA:+.? '
UNH+${order.number}+${order.status}+${order.creationDate?date}'
CTA+${supplier.name}+${supplier.phoneNumber}'
<#list items as item>
LIN+${item.quantity}+${item.code}+${item.price}'
</#list>
```

对于电子邮件:

```
Hi,
Order number #${order.number} created on ${order.creationDate?date} is currently in ${order.status} status.
Consider contacting the supplier "${supplier.name}" with phone number: "${supplier.phoneNumber}".
Order items:
<#list items as item>
${item.quantity} X ${item.code} (total price ${item.price * item.quantity})
</#list>
```

这次的 Smooks 配置非常基础(只要记得导入之前的配置以便导入 Java 绑定设置即可):

```
<?xml version="1.0"?>
<smooks-resource-list 

  xmlns:ftl="http://www.milyn.org/xsd/smooks/freemarker-1.1.xsd">

    <import file="smooks-validation.xml" />

    <ftl:freemarker applyOnElement="#document">
        <ftl:template>/path/to/template.ftl</ftl:template>
    </ftl:freemarker>

</smooks-resource-list>
```

这一次我们只需要向 Smooks 引擎传递一个`StringResult`:

```
Smooks smooks = new Smooks(config);
StringResult stringResult = new StringResult();
smooks.filterSource(new StreamSource(OrderConverter.class
  .getResourceAsStream(path)), stringResult);
return stringResult.toString();
```

当然，我们可以测试它:

```
@Test
public void givenOrderXML_whenApplyEDITemplate_thenConvertedToEDIFACT()
  throws Exception {
    OrderConverter orderConverter = new OrderConverter();
    String edifact = orderConverter.convertOrderXMLtoEDIFACT(
      "/smooks/order.xml");

   assertThat(edifact,is(EDIFACT_MESSAGE));
}
```

## 7 .**。结论**

在本教程中，我们重点介绍了如何将消息转换成不同的格式，或者使用 Smooks 将它们转换成 Java 对象。我们还看到了如何基于正则表达式或业务逻辑规则执行验证。

和往常一样，这里使用的所有代码都可以在 GitHub 上找到[。](https://web.archive.org/web/20221208143832/https://github.com/eugenp/tutorials/tree/master/libraries-data-io)