# Primefaces 简介

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/jsf-primefaces>

## 1。简介

Primefaces 是一个针对 [Java 服务器 Faces](/web/20220628125404/https://www.baeldung.com/spring-jsf)[【JSF】](/web/20220628125404/https://www.baeldung.com/spring-jsf)应用的**开源 UI 组件套件。**

在本教程中，我们将介绍 Primefaces，并演示如何配置它和使用它的一些主要功能。

## 2。概述

### 2.1。Java 服务器面临

Java Server Faces 是一个面向组件的框架，用于为 Java web 应用程序构建用户界面。JSF 规范是通过 Java 社区过程形式化的，是一种标准化的显示技术。

更多关于 JSF 春天环境的信息可以在这里找到。

### 2.2。素数脸

建立在 JSF 之上， **Primefaces 通过提供可以添加到任何项目中的预建 UI 组件**来支持快速应用程序开发。

除了 Primefaces，还有 [Primefaces 扩展](https://web.archive.org/web/20220628125404/https://primefaces-extensions.github.io/)项目。这个基于社区的开源项目提供了标准组件之外的附加组件。

## 3。应用程序设置

为了演示一些 Primefaces 组件，让我们使用 Maven 创建一个简单的 web 应用程序。

### 3.1。Maven 配置

Primefaces 有一个轻量级的配置，只有一个 jar，所以首先，让我们将依赖项添加到我们的`pom.xml`:

```java
<dependency>
    <groupId>org.primefaces</groupId>
    <artifactId>primefaces</artifactId>
    <version>6.2</version>
</dependency>
```

最新版本可以在这里找到[。](https://web.archive.org/web/20220628125404/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22org.primefaces%22)

### 3.2。控制器管理的 Bean

接下来，让我们为组件创建 bean 类:

```java
@ManagedBean(name = "helloPFBean")
public class HelloPFBean {
}
```

我们需要提供一个`@ManagedBean `注释来将控制器绑定到视图组件。

### 3.3。查看

最后，让我们在。`xhtml`文件:

```java
<html xmlns:p="http://primefaces.org/ui"> 
```

## 4。示例组件

要呈现页面，请启动服务器并导航到:

```java
http://localhost:8080/jsf/pf_intro.xhtml
```

### 4.1。`PanelGrid`

让我们用`PanelGrid`作为标准 JSF `panelGrid` `:`的**扩展**

```java
<p:panelGrid columns="2">
    <h:outputText value="#{helloPFBean.firstName}"/>
    <h:outputText value="#{helloPFBean.lastName}" />
</p:panelGrid> 
```

在这里，我们定义了一个包含两列的`panelGrid`，并设置来自 JSF facelets 的`outputText `来显示来自受管 bean 的数据。

每个`outputText `中声明的值对应于我们的`@ManagedBean`中声明的`firstName `和`lastName `属性:

```java
private String firstName;
private String lastName; 
```

让我们看看第一个简单的组件:

```java
<img class=" wp-image-32802 alignnone" style="font-family: Georgia, 'Times New Roman', 'Bitstream Charter', Times, serif; white-space: normal;" src="https://www.baeldung.com/wp-content/uploads/2018/04/panelGridPF-300x68.png" alt="" width="256" height="58" />
```

### 4.2。`SelectOneRadio`

我们可以**使用`selectOneRadio `组件来提供单选按钮功能**:

```java
<h:panelGrid columns="2">
    <p:outputLabel for="jsfCompSuite" value="Component Suite" />
    <p:selectOneRadio id="jsfCompSuite" value="#{helloPFBean.componentSuite}">
        <f:selectItem itemLabel="ICEfaces" itemValue="ICEfaces" />
        <f:selectItem itemLabel="RichFaces" itemValue="Richfaces" />
    </p:selectOneRadio>
</h:panelGrid> 
```

我们需要在后台 bean 中声明 value 变量来保存单选按钮的值:

```java
private String componentSuite; 
```

这个设置将产生一个 2 选项单选按钮，它与底层的`String`属性`componentSuite`相关联:

[![selectOneRadioPF](img/bfdf5f8cc0b501a5e432d42e9121f22f.png)](/web/20220628125404/https://www.baeldung.com/wp-content/uploads/2018/04/selectOneRadioPF.png)

### 4.3。数据表

接下来，让**使用`dataTable`组件在表格布局**中显示数据:

```java
<p:dataTable var="technology" value="#{helloPFBean.technologies}">
    <p:column headerText="Name">
        <h:outputText value="#{technology.name}" />
    </p:column>

    <p:column headerText="Version">
        <h:outputText value="#{technology.currentVersion}" />
    </p:column>
</p:dataTable>
```

类似地，我们需要提供一个 Bean 属性来保存表中的数据:

```java
private List<Technology> technologies; 
```

这里，我们列出了各种技术及其版本号:

[![datatablePF](img/f8b37d40151ac33dd0cec47107510ce7.png)](/web/20220628125404/https://www.baeldung.com/wp-content/uploads/2018/04/datatablePF-1024x119.png)

### 4.4。阿贾克斯用`InputText`

我们还可以使用`p:ajax `为我们的组件提供 Ajax 特性。

例如，让我们使用此组件来应用模糊事件:

```java
<h:panelGrid columns="3">
    <h:outputText value="Blur event " />
    <p:inputText id="inputTextId" value="#{helloPFBean.inputText}}">
        <p:ajax event="blur" update="outputTextId"
	  listener="#{helloPFBean.onBlurEvent}" />
    </p:inputText>
    <h:outputText id="outputTextId" 
      value="#{helloPFBean.outputText}" />
</h:panelGrid> 
```

相应地，我们需要在 bean 中提供属性:

```java
private String inputText;
private String outputText; 
```

此外，我们还需要在 bean 中为 AJAX 模糊事件提供一个监听器方法:

```java
public void onBlurEvent() {
    outputText = inputText.toUpperCase();
}
```

这里，我们简单地将文本转换成大写字母来演示该机制:

[![blurPF](img/00efcfb9773836dceb173acebf65f795.png)](/web/20220628125404/https://www.baeldung.com/wp-content/uploads/2018/04/blurPF.png)

### 4.5。按钮

除此之外，我们还可以使用`p:commandButton `作为标准`h:commandButton `组件的**扩展。**

例如:

```java
<p:commandButton value="Open Dialog" 
  icon="ui-icon-note" 
  onclick="PF('exDialog').show();">
</p:commandButton> 
```

因此，有了这个配置，我们就有了用来打开对话框的按钮(使用`onclick` 属性):

[![commandButton](img/9195d1fb5c0093de0fd8997c6cd3d57d.png)](/web/20220628125404/https://www.baeldung.com/wp-content/uploads/2018/04/commandButton-300x77.png)

### 4.6。对话框

此外，**提供对话框的功能，我们可以使用`p:dialog`组件。**

让我们也使用上一个示例中的按钮来单击打开对话框:

```java
<p:dialog header="Example dialog" widgetVar="exDialog" minHeight="40">
    <h:outputText value="Hello Baeldung!" />
</p:dialog>
```

在这种情况下，我们有一个带有基本配置的对话框，可以使用上一节描述的`commandButton`来触发:

[![dialog](img/663a6bca5336e8c6ec82651d3ec8ff67.png)](/web/20220628125404/https://www.baeldung.com/wp-content/uploads/2018/04/dialog-300x137.png)

## 5.Primefaces 手机

Primefaces Mobile (PFM) **提供了一个 UI 工具包来为移动设备创建 Primefaces 应用程序**。

出于这个原因，PFM 支持为移动设备调整响应式设计。

### 5.1。配置

首先，我们需要在我们的`faces-config.xml`中启用移动导航支持:

```java
<navigation-handler>
    org.primefaces.mobile.application.MobileNavigationHandler
</navigation-handler>
```

### 5.2。名称空间

然后，为了使用 PFM 组件，我们需要在我们的`.xhtml`文件中包含 PFM 名称空间:

```java
xmlns:pm="http://primefaces.org/mobile"
```

除了标准的 Primefaces jar，我们的配置中不需要任何额外的库。

### 5.3.`RenderKit`

最后，我们需要提供用于在移动环境中呈现组件的 **`RenderKit,`。**

如果应用程序中只有一个 PFM 页面，我们可以在页面中定义一个`RenderKit`:

```java
<f:view renderKitId="PRIMEFACES_MOBILE" />
```

对于完整的 PFM 应用程序，我们可以在`faces-config.xml`内的应用程序范围中定义我们的`RenderKit`:

```java
<default-render-kit-id>PRIMEFACES_MOBILE</default-render-kit-id> 
```

### 5.4。示例页

现在，让我们以 page 佩奇为例:

```java
<pm:page id="enter">
    <pm:header>
        <p:outputLabel value="Introduction to PFM"></p:outputLabel>
    </pm:header>
    <pm:content>
        <h:form id="enterForm">
            <pm:field>
	        <p:outputLabel 
                  value="Enter Magic Word">
                </p:outputLabel>
	        <p:inputText id="magicWord" 
                  value="#{helloPFMBean.magicWord}">
                </p:inputText>
	    </pm:field>
            <p:commandButton 
              value="Go!" action="#{helloPFMBean.go}">
            </p:commandButton>
	</h:form>
     </pm:content>
</pm:page>
```

可以看到，我们使用 PFM 的`page, header`和`content `组件构建了一个带有标题的简单表单:

[![pfmIntroBaeldung](img/68ea8c5622fe561ee5a19b02ba316537.png)](/web/20220628125404/https://www.baeldung.com/wp-content/uploads/2018/04/pfmIntroBaeldung-1024x233.png)

此外，我们使用我们的后备 bean 进行用户输入检查和导航:

```java
public String go() {
    if(this.magicWord != null 
      && this.magicWord.toUpperCase().equals("BAELDUNG")) {
	return "pm:success";
     }

    return "pm:failure";
}
```

如果单词正确，我们将进入下一页:

```java
<pm:page id="success">
    <pm:content>
        <p:outputLabel value="Correct!">
        </p:outputLabel>			
	<p:button value="Back" 
          outcome="pm:enter?transition=flow">
        </p:button>
    </pm:content>
</pm:page>
```

这种配置导致了以下布局:

[![correctPagePFM](img/951d8cb41d3b261302d074a7a7bfa25c.png)](/web/20220628125404/https://www.baeldung.com/wp-content/uploads/2018/04/correctPagePFM-1024x138.png)

如果单词不正确，我们将进入下一页:

```java
<pm:page id="failure">
    <pm:content>
        <p:outputLabel value="That is not the magic word">
        </p:outputLabel>
	<p:button value="Back" outcome="pm:enter?transition=flow">
        </p:button>
    </pm:content>
</pm:page>
```

此配置将产生以下布局:

[![incorrectWordPFM](img/a9972f33f4c5d0fbdad1989619ee0e5a.png)](/web/20220628125404/https://www.baeldung.com/wp-content/uploads/2018/04/incorrectWordPFM-1024x137.png)

**请注意， [PFM 在 6.2 版中被弃用](https://web.archive.org/web/20220628125404/https://www.primefaces.org/primefaces-6-2-roadmap/)，在 6.3 版中将被移除，以支持响应标准套件。**

## 6.结论

在本教程中，我们解释了使用 Primefaces JSF 组件套件的好处，并演示了如何在基于 Maven 的项目中配置和使用 Primefaces。

此外，我们还推出了 Primefaces Mobile，这是一款专门针对移动设备的 UI 套件。

和往常一样，GitHub 上的[提供了本教程的代码示例。](https://web.archive.org/web/20220628125404/https://github.com/eugenp/tutorials/tree/master/jsf)