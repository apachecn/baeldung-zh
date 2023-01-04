# JSF EL 2 简介

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/intro-to-jsf-expression-language>

## 1。简介

表达式语言(EL)是一种脚本语言，已经在许多 Java 框架中被采用，例如 Spring 和 [SpEL](/web/20220815040311/https://www.baeldung.com/spring-expression-language) 以及 JBoss 和 JBoss EL。

在本文中，我们将关注 JSF 对这种脚本语言的实现——统一 EL。

EL 目前的版本是 3.0，这是一个重大升级，允许处理引擎以独立模式使用，例如在 Java SE 平台上。以前的版本依赖于 Jakarta EE 兼容的应用服务器或 web 容器。本文讨论 2.2 版。

## 2。即时和延期评估

JSF EL 的主要功能是连接 JSF 视图(通常是 XHTML 标记)和基于 java 的后端。后端可以是用户创建的受管 beans，或者像 HTTP 会话这样的容器管理的对象。

我们将关注 EL 2.2。JSF 的 EL 有两种形式，即时语法 EL 和延迟语法 EL。

### 2.1。立即语法 EL

也称为 JSP EL，这是 java web 应用程序开发的 JSP 时代遗留下来的一种脚本格式。

JSP EL 表达式以美元符号(`$`)开始，然后是左花括号(`{`)，然后是实际表达式，最后以右花括号(`}`)结束:

```
${ELBean.value > 0}
```

该语法:

1.  在页面的生命周期中只评估一次(在开始时)。这意味着。由上面示例中的表达式读取必须在加载页面之前设置。
2.  提供对 bean 值的只读访问。
3.  因此，需要遵守 JavaBean 命名约定。

对于大多数用途来说，这种形式的 EL 不是很通用。

### 2.2。延期执行 EL

延期执行 EL 是专为 JSF 设计的 EL。它与 JSP EL 在语法上的主要区别是，它用“`#”`而不是“`$`来标记。

```
#{ELBean.value > 0}
```

延期 EL:

1.  与 JSF 的生命周期同步。这意味着延迟 EL 中的 EL 表达式在呈现 JSF 页面的不同点(开始和结束)被评估。
2.  提供对 bean 值的读写访问。这允许用户使用 EL 在 JSF 后备 bean(或任何其他地方)中设置一个值。
3.  允许程序员调用对象上的任意方法，并根据 EL 的版本，向这些方法传递参数。

统一 EL 是统一延迟 EL 和 JSP EL 的规范，允许在同一个页面中使用两种语法。

## 3。统一 EL

统一 EL 允许两种通用的表达式，值表达式和方法表达式。

快速注意——以下部分将展示一些例子，这些例子都可以在应用程序中找到(请参见最后的 Github 链接),方法是导航到:

```
http://localhost:8080/jsf/el_intro.jsf
```

### 3.1。值表达式

值表达式允许我们读取或设置托管 bean 属性，这取决于它的位置。

以下表达式将托管 bean 属性读取到页面上:

```
Hello, #{ELBean.firstName}
```

但是，下面的表达式允许我们在用户对象上设置一个值:

```
<h:inputText id="firstName" value="#{ELBean.firstName}" required="true"/>
```

该变量必须遵循 JavaBean 命名约定，才有资格进行这种处理。对于要提交的 bean 的值，只需要保存封装形式。

### 3.2。方法表达式

Unified EL 提供了方法表达式，可以在 JSF 页面中执行公共的非静态方法。这些方法可能有也可能没有返回值。

这里有一个简单的例子:

```
<h:commandButton value="Save" action="#{ELBean.save}"/>
```

被引用的`save()`方法是在名为`ELBean.` 的支持 bean 上定义的

从 EL 2.2 开始，您还可以向使用 EL 访问的方法传递参数。这可以让我们这样重写我们的例子:

```
<h:inputText id="firstName" binding="#{firstName}" required="true"/>
<h:commandButton value="Save"
  action="#{ELBean.saveFirstName(firstName.value.toString().concat('(passed)'))}"/>
```

我们在这里所做的是为`inputText`组件创建一个页面范围的绑定表达式，并将`value`属性直接传递给方法表达式。

注意，变量被传递给方法，没有任何特殊符号、花括号或转义字符。

### 3.3。隐式 EL 对象

JSF EL 引擎提供了对几个容器管理对象的访问。其中一些是:

*   `#{Application}`:也可用作`#{servletContext}`，这是表示 web 应用实例的对象
*   `#{applicationScope}`:可在 web 应用范围内访问的变量图
*   `#{Cookie}`:HTTP Cookie 变量的映射
*   `#{facesContext}`:当前`FacesContext`的实例
*   `#{flash}`:JSF 闪光灯的作用域对象
*   `#{header}`:当前请求中 HTTP 头的映射
*   `#{initParam}`:web 应用的上下文初始化变量的映射
*   `#{param}`:HTTP 请求查询参数的映射
*   `#{request}`:物体`HTTPServletRequest`
*   `#{requestScope}`:请求范围的变量映射
*   `#{sessionScope}`:会话范围的变量映射
*   `#{session}`:物体`HTTPSession`
*   `#{viewScope}`:视图(页面)范围的变量映射

下面这个简单的例子通过访问`headers` 隐式对象列出了所有的请求头和值:

```
<c:forEach items="#{header}" var="header">
   <tr>
       <td>#{header.key}</td>
       <td>#{header.value}</td>
   </tr>
</c:forEach>
```

## 4。你能在 EL 做什么

由于 EL 的多功能性，它可以出现在 Java 代码、XHTML 标记、Javascript 中，甚至出现在像`faces-config.xml` 文件这样的 JSF 配置文件中。让我们检查一些具体的用例。

### 4.1。在页面标记中使用 EL

EL 可以出现在标准 HTML 标签中:

```
<meta name="description" content="#{ELBean.pageDescription}"/>
```

### 4.2。在 JavaScript 中使用 EL

在 Javascript 或

```
<script type="text/javascript"> var theVar = #{ELBean.firstName};</script>
```

在这里，backing bean 变量将被设置为 javascript 变量。

### 4.3。使用运算符评估 EL 中的布尔逻辑

EL 支持相当高级的比较运算符:

*   `eq`相等运算符，相当于“`==.”`
*   `lt`小于运算符，相当于“<”
*   `le`小于等于运算符，相当于“< =”
*   `gt`大于运算符，相当于“>”
*   `ge`大于或等于，相当于`>=.`

### 4.4。在后备 Bean 中评估 EL

在后台 bean 代码中，可以使用 JSF 应用程序计算 EL 表达式。这打开了一个可能性的世界，将 JSF 页面与后台 bean 连接起来。您可以轻松地从后台 bean 中检索隐式 EL 对象，或者检索实际的 HTML 页面组件或它们的值:

```
FacesContext ctx = FacesContext.getCurrentInstance(); 
Application app = ctx.getApplication(); 
String firstName = app.evaluateExpressionGet(ctx, "#{firstName.value}", String.class); 
HtmlInputText firstNameTextBox = app.evaluateExpressionGet(ctx, "#{firstName}", HtmlInputText.class);
```

这使得开发人员在与 JSF 页面交互时有很大的灵活性。

## 5。在 EL 你不能做什么

EL < 3.0 确实有一定的局限性。下面几节讨论其中的一些。

### 5.1。无过载

EL 不支持使用重载。所以在一个 backing bean 中用以下方法:

```
public void save(User theUser);
public void save(String username);
public void save(Integer uid);
```

JSF·埃尔将不能恰当地评价下面的表达式

```
<h:commandButton value="Save" action="#{ELBean.save(firstName.value)}"/>
```

JSF `ELResolver`将自省`bean`的类定义，并选择由`java.lang.Class#getMethods`返回的第一个方法(返回类中可用方法的方法)。方法返回的顺序没有保证，这将不可避免地导致未定义的行为。

### 5.2。没有枚举或常量值

JSF EL < 3.0，不支持在脚本中使用常量值或枚举。因此，拥有以下任何一种

```
public static final String USER_ERROR_MESS = "No, you can’t do that";
enum Days { Sat, Sun, Mon, Tue, Wed, Thu, Fri };
```

意味着您将无法执行以下操作

```
<h:outputText id="message" value="#{ELBean.USER_ERROR_MESS}"/>
<h:commandButton id="saveButton" value="save" rendered="bean.offDay==Days.Sun"/>
```

### 5.3。无内置零安全

JSF EL < 3.0 版不提供隐式空安全访问，有些人可能会觉得现代脚本引擎有些奇怪。

因此，如果下面表达式中的`person`为空，整个表达式将失败，并出现难看的 NPE

```
Hello Mr, #{ELBean.person.surname}"
```

## 6。结论

我们已经研究了 JSF EL 的一些基本原理，优势和局限性。

这在很大程度上是一种通用的脚本语言，还有一些改进的空间；它也是将 JSF 视图绑定到 JSF 模型和控制器的粘合剂。

本文附带的源代码可以从 [GitHub](https://web.archive.org/web/20220815040311/https://github.com/eugenp/tutorials/tree/master/jsf) 获得。