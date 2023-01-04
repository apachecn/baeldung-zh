# Struts 2 简介

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/struts-2-intro>

## 1。简介

Apache Struts 2 是一个基于 MVC 的框架，用于开发企业 Java web 应用程序。这是对原始 Struts 框架的完全重写。它有一个开源 API 实现和丰富的特性集。

在本教程中，我们将向初学者介绍 Struts2 框架的不同核心组件。此外，我们将展示如何使用它们。

## 2。Struts 2 框架概述

Struts 2 的一些特性是:

*   基于 POJO(普通旧 Java 对象)的动作
*   对 REST、AJAX、Hibernate、Spring 等的插件支持
*   约定胜于配置
*   支持各种视图层技术
*   易于分析和调试

### 2.1。支柱 2 的不同部件

Struts2 是基于 MVC 的框架，因此以下三个组件将出现在所有 Struts2 应用程序中:

1.  **Action class—**它是一个 POJO 类(POJO 意味着它不是任何类型层次结构的一部分，可以作为独立的类使用)；我们将在这里实现我们的业务逻辑
2.  **控制器—**在 Struts2 中，使用 HTTP 过滤器作为控制器；它们主要执行拦截和验证请求/响应之类的任务
3.  **视图—**用于展示处理后的数据；它通常是一个 JSP 文件

## 3。设计我们的应用程序

让我们继续开发我们的 web 应用程序。在这个应用程序中，用户选择一个特定的`Car`品牌，就会收到一条定制的消息。

### 3.1。Maven 依赖关系

让我们将以下条目添加到`pom.xml`:

```
<dependency>
    <groupId>org.apache.struts</groupId>
    <artifactId>struts2-core</artifactId>
    <version>2.5.10</version>
</dependency>
<dependency>
    <groupId>org.apache.struts</groupId>
    <artifactId>struts2-junit-plugin</artifactId>
    <version>2.5.10</version>
</dependency>
<dependency>
    <groupId>org.apache.struts</groupId>
    <artifactId>struts2-convention-plugin</artifactId>
    <version>2.5.10</version>
</dependency>
```

依赖关系的最新版本可以在[这里](https://web.archive.org/web/20220815045911/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22org.apache.struts%22)找到。

### 3.2。业务逻辑

让我们创建一个动作类`CarAction`，它为一个特定的输入值返回一条消息。`CarAction`有两个字段——`carName` (用于存储用户的输入)和`carMessage` (用于存储要显示的自定义消息):

```
public class CarAction {

    private String carName;
    private String carMessage;
    private CarMessageService carMessageService = new CarMessageService();

    public String execute() {
        this.setCarMessage(this.carMessageService.getMessage(carName));
        return "success";
    }

    // getters and setters
}
```

`CarAction` 类使用 `CarMessageService`，它为`Car`品牌提供定制消息:

```
public class CarMessageService {

    public String getMessage(String carName) {
        if (carName.equalsIgnoreCase("ferrari")){
            return "Ferrari Fan!";
        }
        else if (carName.equalsIgnoreCase("bmw")){
            return "BMW Fan!";
        }
        else {
            return "please choose ferrari Or bmw";
        }
    }
}
```

### 3.3。接受用户输入

让我们添加一个`JSP`，它是我们应用程序的入口点。这是`input.jsp`文件的一个内容:

```
<body>
    <form method="get" action="/struts2/tutorial/car.action">
        <p>Welcome to Baeldung Struts 2 app</p>
        <p>Which car do you like !!</p>
        <p>Please choose ferrari or bmw</p>
        <select name="carName">
            <option value="Ferrari" label="ferrari" />
            <option value="BMW" label="bmw" />
         </select>
        <input type="submit" value="Enter!" />
    </form>
</body>
```

<`form` >标签指定了动作(在我们的例子中，它是一个 HTTP URI，GET 请求必须发送到它)。

### 3.4。控制器部分

`StrutsPrepareAndExecuteFilter` 是控制器，它将拦截所有传入的请求。我们需要在`web.xml:`中注册以下过滤器

```
<filter>
    <filter-name>struts2</filter-name>
    <filter-class>org.apache.struts2.dispatcher.filter.StrutsPrepareAndExecuteFilter</filter-class>
</filter>

<filter-mapping>
    <filter-name>struts2</filter-name>
    <url-pattern>/*</url-pattern>
</filter-mapping>
```

`StrutsPrepareAndExecuteFilter` 将过滤每一个传入的请求，因为我们正在指定 URL 匹配通配符 **`<url-pattern>/*</url-pattern>`**

### 3.5。配置应用程序

让我们给动作类`Car`添加以下注释:

```
@Namespace("/tutorial")
@Action("/car")
@Result(name = "success", location = "/result.jsp")
```

让我们来理解这个注释的逻辑。 **@** `**Namespace**`用于不同动作类的请求 URI 的逻辑分离；我们需要在请求中包含这个值。

此外，`**@Action**`告诉我们将到达我们的`Action`类的请求 URI 的实际端点。action 类查询 `CarMessageService`并初始化另一个成员变量`carMessage`的值。在`execute()`方法返回一个值之后，在我们的例子中是**“成功”**，它匹配那个值来调用`result.jsp`

最后，`**@Result**`有两个参数。第一个，`name,` 指定了我们的`Action`类将返回的值；这个值是从`Action`类的`execute()` 方法返回的。**这是将被执行的默认方法名称**。

第二部分，`location,`告诉我们在`execute()`方法返回一个值后哪个文件将被引用。这里，我们指定当`execute()` 返回一个值为“`success`的字符串时，我们必须将请求转发给`result.jsp`。

通过提供 XML 配置文件可以实现相同的配置:

```
<struts>
    <package name="tutorial" extends="struts-default" namespace="/tutorial">
        <action name="car" class="com.baeldung.struts.CarAction" method="execute">
            <result name="success">/result.jsp</result>
        </action>
    </package>
</struts>
```

### 3.6。视图

这是`result.jsp` 的内容，将用于向用户呈现消息:

```
<%@ page contentType="text/html; charset=UTF-8" %>
<%@ taglib prefix="s" uri="/struts-tags" %>
<body>
    <p> Hello Baeldung User </p>
    <p>You are a <s:property value="carMessage"/></p>
</body>
```

这里有两件重要的事情需要注意:

*   在 `<@taglib prefix=”s” uri=”/struts-tags %>`中，我们正在导入`struts-tags`库
*   在 `<s:property value=”carMessage”/>`中，我们使用`struts-tags`库来打印属性`carMessage`的值

## 4。运行应用程序

这个 web 应用程序可以在任何 web 容器中运行，例如在 Apache Tomcat 中。以下是实现这一目标的必要步骤:

1.  部署 web 应用程序后，打开浏览器并访问以下 URL: `http://www.localhost.com:8080/MyStrutsApp/input.jsp`
2.  从两个选项中选择一个并提交请求
3.  您将被转到`result.jsp`页面，根据选择的输入选项显示定制的消息

## 5。结论

在本教程中，我们一步一步地介绍了如何创建我们的第一个 Struts2 web 应用程序。我们讨论了 Struts2 领域中与 MVC 相关的不同方面，并展示了如何将它们放在一起进行开发。

和往常一样，这个教程可以作为一个 Maven 项目在 Github 上找到。