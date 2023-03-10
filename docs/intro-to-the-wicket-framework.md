# Wicket 框架简介

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/intro-to-the-wicket-framework>

## 1。概述

Wicket 是一个面向 Java 服务器端 web 组件的框架，旨在通过引入桌面 UI 开发中的模式来简化 web 界面的构建。

有了 Wicket，只使用 Java 代码和 XHTML 兼容的 HTML 页面就可以构建 web 应用程序。不需要 Javascript，也不需要 XML 配置文件。

它在请求-响应周期上提供了一个层，避免了在底层工作，并允许开发人员专注于业务逻辑。

在本文中，我们将通过构建`HelloWorld W` icket 应用程序来介绍基础知识，然后是一个使用两个相互通信的内置组件的完整示例。

## 2。设置

要运行 Wicket 项目，让我们添加以下依赖项:

```java
<dependency>
    <groupId>org.apache.wicket</groupId>
    <artifactId>wicket-core</artifactId>
    <version>7.4.0</version>
</dependency>
```

您可能想在 [Maven 中央存储库](https://web.archive.org/web/20220706220028/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.apache.wicket%22%20AND%20a%3A%22wicket%22)中查看 Wicket 的最新版本，在您阅读时，它可能与这里使用的版本不一致。

现在我们准备构建我们的第一个 Wicket 应用程序。

## 3。`HelloWorld` 检票口

让我们从子类化 Wicket 的`WebApplication`类开始，它至少需要覆盖`Class<? extends Page> getHomePage()`方法。

Wicket 将使用这个类作为应用程序的主要入口点。在方法内部，简单地返回一个名为`HelloWorld:`的类的`class`对象

```java
public class HelloWorldApplication extends WebApplication {
    @Override
    public Class<? extends Page> getHomePage() {
        return HelloWorld.class;
    }
}
```

Wicket 更倾向于约定而不是配置。向应用程序添加新网页需要创建两个文件:一个 Java 文件和一个在同一目录下具有相同名称(但不同的扩展)的 HTML 文件。仅当要更改默认行为时，才需要额外的配置。

在源代码的包目录中，首先添加`HelloWorld.java`:

```java
public class HelloWorld extends WebPage {
    public HelloWorld() {
        add(new Label("hello", "Hello World!"));
    }
}
```

然后`HelloWorld.html`:

```java
<html>
    <body>
        <span wicket:id="hello"></span>
    </body>
</html>
```

最后一步，在`web.xml:`中添加过滤器定义

```java
<filter>
    <filter-name>wicket.examples</filter-name>
    <filter-class>
      org.apache.wicket.protocol.http.WicketFilter
    </filter-class>
    <init-param>
        <param-name>applicationClassName</param-name>
        <param-value>
          com.baeldung.wicket.examples.HelloWorldApplication
        </param-value>
    </init-param>
</filter>
```

就是这样。我们刚刚编写了第一个 Wicket web 应用程序。

通过构建一个`war`文件来运行项目(从命令行中构建`mvn package`，并将其部署在一个 servlet 容器上，比如 Jetty 或 Tomcat。

让我们在浏览器中访问[http://localhost:8080/hello world/](https://web.archive.org/web/20220706220028/http://localhost:8080/HelloWorld/)。将出现一个空白页面，显示消息`Hello World!` 。

## 4。Wicket 组件

Wicket 中的组件是由 Java 类、HTML 标记和模型组成的三元组。模型是组件用来访问数据的门面。

这种结构很好地分离了关注点，并且通过将组件从以数据为中心的操作中分离出来，增加了代码重用。

**下面的例子展示了如何将 Ajax 行为添加到组件中。它由一个包含两个元素的页面组成:一个下拉菜单和一个标签。当下拉选择改变时，标签(并且只有标签)将被更新。**

HTML 文件`CafeSelector.html`的主体将是最小的，只有两个元素，一个下拉菜单和一个标签:

```java
<select wicket:id="cafes"></select>
<p>
    Address: <span wicket:id="address">address</span>
</p>
```

在 Java 端，让我们创建标签:

```java
Label addressLabel = new Label("address", 
  new PropertyModel<String>(this.address, "address"));
addressLabel.setOutputMarkupId(true);
```

与 HTML 文件中分配的`wicket:id`匹配的`Label`构造函数中的第一个参数。第二个参数是组件的模型，它是组件中底层数据的包装器。

`setOutputMarkupId`方法使得组件可以通过 Ajax 修改。现在让我们创建下拉列表，并向其中添加 Ajax 行为:

```java
DropDownChoice<String> cafeDropdown 
  = new DropDownChoice<>(
    "cafes", 
    new PropertyModel<String>(this, "selectedCafe"), 
    cafeNames);
cafeDropdown.add(new AjaxFormComponentUpdatingBehavior("onchange") {
    @Override
    protected void onUpdate(AjaxRequestTarget target) {
        String name = (String) cafeDropdown.getDefaultModel().getObject();
        address.setAddress(cafeNamesAndAddresses.get(name).getAddress());
        target.add(addressLabel);
    }
});
```

创建过程类似于标签，构造函数接受 wicket id、模型和咖啡馆名称列表。

然后向`AjaxFormComponentUpdatingBehavior`添加了`onUpdate`回调方法，一旦发出 ajax 请求，该方法就会更新标签的模型。最后，将标签组件设置为刷新目标。

最后，将标签组件设置为刷新目标。

如你所见，一切都是 Java，不需要一行 Javascript 代码。为了改变标签显示的内容，我们简单地修改了一个 POJO。修改 Java 对象转化为网页变化的机制发生在幕后，与开发人员无关。

Wicket 提供了大量现成的支持 AJAX 的组件。带实例的组件目录可在[此处](https://web.archive.org/web/20220706220028/https://wicket.apache.org/learn/examples/index.html)获得。

## 5。结论

在这篇介绍性文章中，我们已经介绍了 Wicket 的基础知识 Java 中基于组件的 web 框架。

Wicket 提供了一个抽象层，旨在完全去除管道代码。

我们包含了两个简单的例子，可以在 GitHub 上找到[，让您体验一下使用这个框架的开发是什么样子的。](https://web.archive.org/web/20220706220028/https://github.com/eugenp/tutorials/tree/master/wicket)