# Apache Tapestry 简介

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/apache-tapestry>

## 1.概观

如今，从社交网络到银行，从医疗保健到政府服务，所有的活动都可以在网上进行。因此，他们非常依赖 web 应用程序。

web 应用程序使用户能够消费/享受公司提供的在线服务。同时，它充当后端软件的接口。

在这篇介绍性教程中，我们将探索 Apache Tapestry web 框架，并使用它提供的基本特性创建一个简单的 web 应用程序。

## 2.阿帕奇挂毯

Apache Tapestry 是一个基于组件的框架，用于构建可伸缩的 web 应用程序。

它遵循**约定优先于配置**的范例，并为配置使用注释和命名约定。

所有组件都是简单的 POJOs。同时，它们是从零开始开发的，不依赖于其他库。

除了 Ajax 支持，Tapestry 还具有强大的异常报告功能。它还提供了一个丰富的内置通用组件库。

在其他伟大的特性中，一个突出的特性是代码的热重载。因此，使用这个特性，我们可以立即看到开发环境中的变化。

## 3.设置

Apache Tapestry 需要一组简单的工具来创建 web 应用程序:

*   Java 1.6 或更高版本
*   构建工具(Maven 或 Gradle)
*   IDE (Eclipse 或 IntelliJ)
*   应用服务器(Tomcat 或 Jetty)

在本教程中，我们将结合使用 Java 8、Maven、Eclipse 和 Jetty Server。

为了建立[最新的](https://web.archive.org/web/20220703153749/https://tapestry.apache.org/download.html) Apache Tapestry 项目，我们将使用 [Maven 原型](/web/20220703153749/https://www.baeldung.com/maven-archetype#creating-archetype)，并遵循官方文档提供的[指令](https://web.archive.org/web/20220703153749/https://tapestry.apache.org/getting-started.html):

```java
$ mvn archetype:generate -DarchetypeCatalog=http://tapestry.apache.org
```

或者，如果我们有一个现有的项目，我们可以简单地将 [tapestry-core](https://web.archive.org/web/20220703153749/https://search.maven.org/search?q=g:org.apache.tapestry%20a:tapestry-core) Maven 依赖项添加到`pom.xml`:

```java
<dependency>
    <groupId>org.apache.tapestry</groupId>
    <artifactId>tapestry-core</artifactId>
    <version>5.4.5</version>
</dependency>
```

一旦我们准备好设置，我们可以通过下面的 Maven 命令启动应用程序`apache-tapestry`:

```java
$ mvn jetty:run
```

默认情况下，可在`localhost:8080/apache-tapestry`访问该应用程序:

[![](img/b81078b099c3975cfa3c4323e953ca8f.png)](/web/20220703153749/https://www.baeldung.com/wp-content/uploads/2019/11/homepage-1.png)

## 4.项目结构

让我们探索由 Apache Tapestry 创建的项目布局:

[![](img/b96c108d474967b797b5aa24d84fff4f.png)](/web/20220703153749/https://www.baeldung.com/wp-content/uploads/2019/11/tree_structure.png)

我们可以看到一个类似 Maven 的项目结构，以及一些基于约定的包。

**Java 类放在`src/main/java`中，分为`components`、`pages`和`services.`、**

同样，`src/main/resources`保存我们的模板(类似于 HTML 文件)——这些模板有`.tml`扩展名。

**对于放置在`components`和`pages` 目录下的每个 Java 类，都应该创建一个同名的模板文件。**

`src/main/webapp` 目录包含图像、样式表和 JavaScript 文件等资源。同样，测试文件放在`src/test`中。

最后，`src/site`将包含文档文件。

为了更好地理解，让我们来看看在 Eclipse IDE 中打开的项目结构:

[![](img/224796993c954689f477415ff1320872.png)](/web/20220703153749/https://www.baeldung.com/wp-content/uploads/2019/11/project_structure.png)

## 5.释文

让我们讨论一下 Apache Tapestry 为日常使用提供的一些方便的[注释](https://web.archive.org/web/20220703153749/https://tapestry.apache.org/annotations.html)。接下来，我们将在实现中使用这些注释。

### 5.1.`**@Inject**`

在`org.apache.tapestry5.ioc.annotations` 包中有`[@Inject](https://web.archive.org/web/20220703153749/https://tapestry.apache.org/current/apidocs/org/apache/tapestry5/ioc/annotations/Inject.html)`注释，它提供了一种在 Java 类中注入依赖关系的简单方法。

这个注释对于注入资产、块、资源和服务非常方便。

### 5.2.`@InjectPage`

在`org.apache.tapestry5.annotations`包中， [`@InjectPage`](https://web.archive.org/web/20220703153749/https://tapestry.apache.org/current/apidocs/org/apache/tapestry5/annotations/InjectPage.html) 注释允许我们将一个页面注入到另一个组件中。此外，注入的页面始终是只读属性。

### 5.3.`@InjectComponent`

同样， [`@InjectComponent`](https://web.archive.org/web/20220703153749/https://tapestry.apache.org/current/apidocs/org/apache/tapestry5/annotations/InjectComponent.html) 注释允许我们注入一个在模板中定义的组件。

### 5.4.`@Log`

在`org.apache.tapestry5.annotations`包中有 [`@Log`](https://web.archive.org/web/20220703153749/https://tapestry.apache.org/current/apidocs/org/apache/tapestry5/annotations/Log.html) 注释，可以方便地在任何方法上启用调试级别日志记录。它记录方法入口和出口，以及参数值。

### 5.5.`@Property`

在`org.apache.tapestry5.annotations`包中， [`@Property`](https://web.archive.org/web/20220703153749/https://tapestry.apache.org/current/apidocs/org/apache/tapestry5/annotations/Property.html) 注释将一个字段标记为属性。同时，它会自动为属性创建 getters 和 setters。

### 5.6.`@Parameter`

类似地，`[@Parameter](https://web.archive.org/web/20220703153749/https://tapestry.apache.org/current/apidocs/org/apache/tapestry5/annotations/Parameter.html)`注释表示一个字段是一个组件参数。

## 6.页

所以，我们已经准备好探索这个框架的基本特性了。让我们在应用程序中创建新的`Home`页面。

首先，我们将在`src/main/java`的`pages`目录中定义一个 Java 类`Home`:

```java
public class Home {
}
```

### 6.1.模板

然后，我们将在`src/main/resources`下的`pages` 目录中创建一个相应的`Home.tml` [模板](https://web.archive.org/web/20220703153749/https://tapestry.apache.org/component-templates.html)。

扩展名为`.tml` (Tapestry 标记语言)的文件类似于 Apache Tapestry 提供的带有 XML 标记的 HTML/XHTML 文件。

例如，让我们看看`Home.tml`模板:

```java
<html xmlns:t="http://tapestry.apache.org/schema/tapestry_5_4.xsd">
    <head>
        <title>apache-tapestry Home</title>
    </head>
    <body>
        <h1>Home</h1>
    </body>   
</html>
```

瞧啊。只需重启 Jetty 服务器，我们就可以在`localhost:8080/apache-tapestry/home`访问`Home`页面:

[![](img/df17a2b32ca82dff4f7cbd3c430e448b.png)](/web/20220703153749/https://www.baeldung.com/wp-content/uploads/2019/11/home1.png)

### 6.2.财产

让我们探索一下如何在`Home`页面上呈现属性。

为此，我们将在`Home` 类中添加一个属性和一个 getter 方法:

```java
@Property
private String appName = "apache-tapestry";

public Date getCurrentTime() {
    return new Date();
}
```

要在`Home`页面上呈现`appName`属性，我们可以简单地使用`${appName}`。

类似地，我们可以编写`${currentTime}`来从页面访问`getCurrentTime`方法。

### 6.3.本地化

Apache Tapestry 提供了集成的[本地化](https://web.archive.org/web/20220703153749/https://tapestry.apache.org/localization.html)支持。按照惯例，页面名称属性文件保存了要在页面上呈现的所有本地消息的列表。

例如，我们将在`pages`目录中为带有本地消息的`Home`页面创建一个`home.properties`文件:

```java
introMsg=Welcome to the Apache Tapestry Tutorial
```

消息属性不同于 Java 属性。

出于同样的原因，带有`message`前缀的键名用于呈现消息属性——例如，`${message:introMsg}.`

### 6.4.布局组件

让我们通过创建`Layout.java`类来定义一个基本的布局组件。我们将把文件保存在`src/main/java`的`components`目录中:

```java
public class Layout {
    @Property
    @Parameter(required = true, defaultPrefix = BindingConstants.LITERAL)
    private String title;
}
```

这里，`title`属性被标记为 required，绑定的默认前缀被设置为 literal `String`。

然后，我们将在`src/main/resources`的`components`目录下写一个相应的模板文件`Layout.tml`:

```java
<html xmlns:t="http://tapestry.apache.org/schema/tapestry_5_4.xsd">
    <head>
        <title>${title}</title>
    </head>
    <body>
        <div class="container">
            <t:body />
            <hr/>
            <footer>
                <p>© Your Company</p>
            </footer>
        </div>
    </body>
</html>
```

现在，让我们使用`home`页面上的`layout`:

```java
<html t:type="layout" title="apache-tapestry Home" 
    xmlns:t="http://tapestry.apache.org/schema/tapestry_5_4.xsd">
    <h1>Home! ${appName}</h1>
    <h2>${message:introMsg}</h2>
    <h3>${currentTime}</h3>
</html>
```

注意，[名称空间](https://web.archive.org/web/20220703153749/https://tapestry.apache.org/schema/tapestry_5_4.xsd)用于标识 Apache Tapestry 提供的元素(`t:type`和`t:body`)。同时，命名空间还提供了组件和属性。

这里，`t:type`将在`home`页面上设置`layout`。并且，`t:body`元素将插入页面的内容。

让我们来看看带有布局的`Home`页面:

[![](img/46c92946fbbf82460d4d0f8a3ea12084.png)](/web/20220703153749/https://www.baeldung.com/wp-content/uploads/2019/11/homepage2-1.png)

## 7.形式

让我们创建一个带有表单的`Login`页面，允许用户登录。

正如已经探讨过的，我们将首先创建一个 Java 类`Login`:

```java
public class Login {
    // ...
    @InjectComponent
    private Form login;

    @Property
    private String email;

    @Property
    private String password;
}
```

这里，我们定义了两个属性— `email`和`password`。此外，我们还为登录注入了一个`Form`组件。

然后，让我们创建一个相应的模板`login.tml`:

```java
<html t:type="layout" title="apache-tapestry com.example"
      xmlns:t="http://tapestry.apache.org/schema/tapestry_5_3.xsd"
      xmlns:p="tapestry:parameter">
    <t:form t:id="login">
        <h2>Please sign in</h2>
        <t:textfield t:id="email" placeholder="Email address"/>
        <t:passwordfield t:id="password" placeholder="Password"/>
        <t:submit class="btn btn-large btn-primary" value="Sign in"/>
    </t:form>
</html>
```

现在，我们可以在`localhost:8080/apache-tapestry/login`访问`login`页面:

[![](img/53cd8a27b849e1f641d97330c6783f6e.png)](/web/20220703153749/https://www.baeldung.com/wp-content/uploads/2019/11/login-1.png)

## 8.确认

Apache Tapestry 为[表单验证](https://web.archive.org/web/20220703153749/https://tapestry.apache.org/forms-and-validation.html)提供了一些内置方法。它还提供了处理表单提交成功或失败的方法。

内置方法遵循事件和组件名称的约定。例如，方法`onValidationFromLogin` 将验证`Login`组件。

同样，像`onSuccessFromLogin`和`onFailureFromLogin`这样的方法分别用于成功和失败事件。

因此，让我们将这些内置方法添加到`Login` 类中:

```java
public class Login {
    // ...

    void onValidateFromLogin() {
        if (email == null)
            System.out.println("Email is null);

        if (password == null)
            System.out.println("Password is null);
    }

    Object onSuccessFromLogin() {
        System.out.println("Welcome! Login Successful");
        return Home.class;
    }

    void onFailureFromLogin() {
        System.out.println("Please try again with correct credentials");
    }
}
```

## 9.警报

没有适当的警告，表单验证是不完整的。更不用说，框架还内置了对警告消息的支持。

为此，我们将首先在`Login` 类中注入 [`AlertManager`](https://web.archive.org/web/20220703153749/https://tapestry.apache.org/current/apidocs/org/apache/tapestry5/alerts/AlertManager.html) 的实例来管理警报`.`，然后用警报消息替换现有方法中的`println`语句:

```java
public class Login {
    // ...
    @Inject
    private AlertManager alertManager;

    void onValidateFromLogin() {
        if(email == null || password == null) {
            alertManager.error("Email/Password is null");
            login.recordError("Validation failed"); //submission failure on the form
        }
    }

    Object onSuccessFromLogin() {
        alertManager.success("Welcome! Login Successful");
        return Home.class;
    }

    void onFailureFromLogin() {
        alertManager.error("Please try again with correct credentials");
    }
}
```

让我们看看登录失败时的警报:

[![](img/9ebf36b9743a3dfc0dc3b1f18d7b570e.png)](/web/20220703153749/https://www.baeldung.com/wp-content/uploads/2019/11/loginfail-1.png)

## 10.埃阿斯

到目前为止，我们已经探索了一个简单的带有表单的`home`页面的创建。同时，我们已经看到了对警告消息的验证和支持。

接下来，让我们探索 Apache Tapestry 对 Ajax 的内置支持。

首先，我们将在`Home` 类中注入 [`AjaxResponseRenderer`](https://web.archive.org/web/20220703153749/https://tapestry.apache.org/current/apidocs/org/apache/tapestry5/services/ajax/AjaxResponseRenderer.html) 和 [`Block`](https://web.archive.org/web/20220703153749/https://tapestry.apache.org/current/apidocs/org/apache/tapestry5/Block.html) 组件的实例。然后，我们将创建一个方法`onCallAjax`来处理 Ajax 调用:

```java
public class Home {
    // ....

    @Inject
    private AjaxResponseRenderer ajaxResponseRenderer;

    @Inject
    private Block ajaxBlock;

    @Log
    void onCallAjax() {
        ajaxResponseRenderer.addRender("ajaxZone", ajaxBlock);
    }
}
```

此外，我们需要对我们的`Home.tml`做一些修改。

首先，我们将添加 [`eventLink`](https://web.archive.org/web/20220703153749/https://tapestry.apache.org/current/apidocs/org/apache/tapestry5/corelib/components/EventLink.html) 来调用`onCallAjax`方法。然后，我们将添加一个 id 为`ajaxZone`的 [`zone`](https://web.archive.org/web/20220703153749/https://tapestry.apache.org/current/apidocs/org/apache/tapestry5/corelib/components/Zone.html) 元素来呈现 Ajax 响应。

最后，我们需要一个 block 组件，它将被注入到`Home `类中，并呈现为 Ajax 响应:

```java
<p><t:eventlink event="callAjax" zone="ajaxZone" class="btn btn-default">Call Ajax</t:eventlink></p>
<t:zone t:id="ajaxZone"></t:zone>
<t:block t:id="ajaxBlock">
    <hr/>
    <h2>Rendered through Ajax</h2>
    <p>The current time is: <strong>${currentTime}</strong></p>
</t:block>
```

让我们来看看更新后的`home`页面:

[![](img/765101d17cdd731f13953ad6d3e812d4.png)](/web/20220703153749/https://www.baeldung.com/wp-content/uploads/2019/11/home-1.png)

然后，我们可以单击 Call Ajax 按钮并查看`ajaxResponseRenderer`的运行情况:

[![](img/e42724210cc83857ee05fb5e74a7c061.png)](/web/20220703153749/https://www.baeldung.com/wp-content/uploads/2019/11/homeAjax.png)

## 11.记录

要启用内置日志功能，需要注入 [`Logger`](https://web.archive.org/web/20220703153749/http://www.slf4j.org/api/org/slf4j/Logger.html) 的实例。然后，我们可以使用它来记录任何级别的日志，如跟踪、调试和信息。

因此，让我们在`Home `类中进行必要的修改:

```java
public class Home {
    // ...

    @Inject
    private Logger logger;

    void onCallAjax() {
        logger.info("Ajax call");
        ajaxResponseRenderer.addRender("ajaxZone", ajaxBlock);
    }
}
```

现在，当我们单击 Call Ajax 按钮时，`logger`将在 INFO 级别进行记录:

```java
[INFO] pages.Home Ajax call 
```

## 12.结论

在本文中，我们探索了 Apache Tapestry web 框架。

首先，我们创建了一个 quickstart web 应用程序，并使用 Apache Tapestry 的基本特性添加了一个`Home`页面，比如`components`、`pages`和`templates`。

然后，我们研究了 Apache Tapestry 提供的一些方便的注释，用于配置属性和组件/页面注入。

最后，我们探索了框架提供的内置 Ajax 和日志支持。

像往常一样，所有的代码实现都可以在 GitHub 上获得[。](https://web.archive.org/web/20220703153749/https://github.com/eugenp/tutorials/tree/master/apache-tapestry)