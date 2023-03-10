# Java EE 7 中的转换器、监听器和验证器

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-ee7-converter-listener-validator>

## 1。概述

Java 企业版(JEE) 7 提供了一些有用的功能，例如验证用户输入，将值转换为适当的 Java 数据类型。

在本教程中，我们将关注转换器、监听器和验证器提供的特性。

## 2。转换器

转换器允许我们将字符串输入值转换成 Java 数据类型。预定义的转换器位于`javax.faces.convert`包中，它们兼容任何 Java 数据类型，甚至像`Date.`这样的标准类

为了定义一个`Integer`转换器，首先我们在被管理的 bean 中创建我们的属性，用作我们的`JSF`表单的后端:

```java
private Integer age;

// getters and setters
```

然后，我们使用`f:converter` 标签在表单中创建组件:

```java
<h:outputLabel value="Age:"/>
<h:inputText id="Age" value="#{convListVal.age}">
    <f:converter converterId="javax.faces.Integer" />
</h:inputText>
<h:message for="Age" />
```

以类似的方式，我们创建其他数字转换器，如`Double`转换器:

```java
private Double average;
```

然后，我们在视图中创建适当的 JSF 组件。请注意，我们使用的是变量`average,`,然后使用 getter 和 setter 通过名称约定将其映射到字段:

```java
<h:outputLabel value="Average:"/>
<h:inputText id="Average" value="#{convListVal.average}">
    <f:converter converterId="javax.faces.Double" />
</h:inputText>
<h:message for="Average" />
```

**如果我们想给用户反馈，我们需要包含一个`h:message` 标签，控件使用它作为错误消息的占位符。**

一个有用的转换器是`DateTime`转换器，因为它允许我们验证日期、时间和这些值的格式。

首先，和前面的转换器一样，我们用 getters 和 setters 来声明我们的字段:

```java
private Date myDate;
// getters and setters
```

然后我们在视图中创建组件。这里我们需要使用该模式输入日期，如果没有使用该模式，我们将得到一个错误，并给出一个正确的输入模式示例:

```java
<h:outputLabel value="Date:"/>
<h:inputText id="MyDate" value="#{convListVal.myDate}">
    <f:convertDateTime pattern="dd/MM/yyyy" />
</h:inputText>
<h:message for="MyDate" />
<h:outputText value="#{convListVal.myDate}">
    <f:convertDateTime dateStyle="full" locale="en"/>
</h:outputText>
```

在我们的例子中，我们可以转换输入日期并发送 post 数据，在我们的`h:outputText.`中格式化为完整的日期

## 3。听众

侦听器允许我们监控组件的变化；我们正在监视文本字段的值何时改变。

像以前一样，我们在托管 bean 中定义属性:

```java
private String name;
```

然后我们在视图中定义我们的监听器:

```java
<h:outputLabel value="Name:"/>
<h:inputText id="name" size="30" value="#{convListVal.name}">
    <f:valueChangeListener type="com.baeldung.convListVal.MyListener" />
</h:inputText>
```

我们通过添加一个`f:valueChangeListener` 来设置我们的`h:inputText` 标签，并且，在监听器标签中，我们需要指定一个类，当监听器被触发时，它将用于执行任务。

```java
public class MyListener implements ValueChangeListener {
    private static final Logger LOG = Logger.getLogger(MyListener.class.getName());	

    @Override
    public void processValueChange(ValueChangeEvent event)
      throws AbortProcessingException {
        if (event.getNewValue() != null) {
            LOG.log(Level.INFO, "\tNew Value:{0}", event.getNewValue());
        }
    }
}
```

监听器类必须实现`ValueChangeListener` 接口并覆盖`processValueChange()`方法来执行监听器任务，以写入日志消息。

## 4。验证器

我们使用一个验证器来验证 JSF 组件数据，并提供一组标准类来验证用户输入。

这里，我们定义了一个标准的验证器，使我们能够检查用户在文本字段中输入的长度。

首先，我们在受管 bean 中创建我们的字段:

```java
private String surname;
```

然后，我们在视图中创建组件:

```java
<h:outputLabel value="surname" for="surname"/>
<h:panelGroup>
    <h:inputText id="surname" value="#{convListVal.surname}">
        <f:validateLength minimum="5" maximum="10"/>
    </h:inputText>
    <h:message for="surname" errorStyle="color:red"  />
</h:panelGroup>
```

在`h:inputText`标签中，我们放置了验证器，来验证输入的长度。请记住，在 JSF 有各种预定义的标准验证器，我们可以用与这里类似的方式使用它们。

## 5。测试

为了测试这个 JSF 应用程序，我们将使用 Arquillian 对无人机、石墨烯和 Selenium Web 驱动程序进行功能测试。

首先，我们使用`ShrinkWrap:`部署我们的应用程序

```java
@Deployment(testable = false)
public static WebArchive createDeployment() {
    return (ShrinkWrap.create(
      WebArchive.class, "jee7.war").
      addClasses(ConvListVal.class, MyListener.class)).
      addAsWebResource(new File(WEBAPP_SRC, "ConvListVal.xhtml")).
      addAsWebInfResource(EmptyAsset.INSTANCE, "beans.xml");
}
```

然后，我们测试每个组件的错误消息，以验证我们的应用程序是否正常工作:

```java
@Test
@RunAsClient
public void givenAge_whenAgeInvalid_thenErrorMessage() throws Exception {
    browser.get(deploymentUrl.toExternalForm() + "ConvListVal.jsf");
    ageInput.sendKeys("stringage");
    guardHttp(sendButton).click();
    assertTrue("Show Age error message",
      browser.findElements(By.id("myForm:ageError")).size() > 0);
}
```

对每个组件进行类似的测试。

## 6。总结

在本教程中，我们创建了由 JEE7 提供的转换器、监听器和验证器的实现。

你可以在 Github 的文章[中找到代码。](https://web.archive.org/web/20221128105358/https://github.com/eugenp/tutorials/tree/master/web-modules/jee-7)